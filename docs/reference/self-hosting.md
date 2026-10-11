# Self-Hosting on a VPS

Part of the [Bobina Council Docs](../../README.md) · [Reference](./README.md)

Production runs on Vercel today, with Supabase, Upstash and Vercel Blob. That setup **stays until the move**. This page covers running both apps on your own server. Start with [Local Installation](./local-installation.md), because the services and env vars are the same.

> **(planned)** Full portability depends on the adapter work with the backup rework (Moe #866): storage, database and Redis each sit behind an adapter, so moving is a config change. Today Moe's **storage** adapter (`STORAGE_DRIVER=s3`) and Companion's **Redis** driver (`REDIS_DRIVER=redis`) already exist. Moe's TCP Redis adapter and Companion's storage adapter are planned.

## Target stack

| Piece | Today (Vercel) | Self-hosted |
|---|---|---|
| Apps | Vercel projects | `pnpm build && pnpm start` under systemd/PM2, or Docker |
| Database | Supabase cloud | [Self-hosted Supabase (Docker)](https://supabase.com/docs/guides/self-hosting/docker) |
| Redis | Upstash (REST) | Redis/Valkey. Companion uses TCP. Moe uses REST via an [SRH](https://upstash.com/docs/redis/sdks/ts/developing) proxy until its TCP adapter lands **(planned)** |
| Blob | Vercel Blob | [RustFS](https://docs.rustfs.com/) or [MinIO](https://min.io/docs/minio/linux/index.html) (S3-compatible) |
| Crons | `vercel.json` | system `cron` / systemd timers calling the routes with `CRON_SECRET` |
| AI | Vercel AI Gateway (OIDC) | AI Gateway with `AI_GATEWAY_API_KEY` |

## 1. Reverse proxy and TLS

Put [Caddy](https://caddyserver.com/docs/automatic-https) (automatic HTTPS) or nginx with [Certbot](https://certbot.eff.org/) in front of everything:

```caddy
bobina.example.com      { reverse_proxy 127.0.0.1:3000 }
companion.example.com   { reverse_proxy 127.0.0.1:3001 }
files.example.com       { reverse_proxy 127.0.0.1:9000 }   # public S3 reads; S3_ENDPOINT / S3_PUBLIC_BASE_URL must be https
```

- Set `NEXTAUTH_URL`, `NEXT_PUBLIC_APP_URL`, `BOBINA_COUNCIL_URL` and so on to the **https** hostnames.
- Moe's CSRF check compares the request origin with its own host, so the proxy must pass `Host` and `X-Forwarded-Proto` through (Caddy does this by default).
- Update OAuth callback URLs, the Discord interactions URL and Telegram `setWebhook` to the new hostnames.

## 2. Process manager or Docker Compose

**systemd** (one unit per app):

```ini
# /etc/systemd/system/bobina-moe.service
[Unit]
After=network.target
[Service]
User=bobina
WorkingDirectory=/srv/Bobina.moe
EnvironmentFile=/etc/bobina/moe.env
ExecStart=/usr/bin/env pnpm start -p 3000
Restart=always
[Install]
WantedBy=multi-user.target
```

Or use [PM2](https://pm2.keymetrics.io/docs/usage/quick-start/), or a Docker Compose file built on the [Next.js standalone output](https://nextjs.org/docs/app/api-reference/config/next-config-js/output). Build with `pnpm install --frozen-lockfile && pnpm build`. Companion's `build` also registers Discord commands.

## 3. Persistence volumes

Keep data on named volumes or a dedicated disk, never inside a container's writable layer:

- Supabase Postgres data directory (`volumes/db/data` in the Supabase compose)
- Redis `/data` with AOF on (`--appendonly yes`)
- RustFS/MinIO `/data`
- `/etc/bobina/*.env` (root-owned, `chmod 600`)

## 4. Scheduled backups with off-box copies

Two layers:

1. **App backups** (Redis + Supabase + blob): the admin backup, with scheduled auto-backups **(planned)** writing to the private backup store. See [Backup & Restore](./backup-restore.md).
2. **Infrastructure backups:** nightly `pg_dump` of Postgres, a Redis `BGSAVE` copy, and a bucket mirror. Copy all of it **off the box**, for example with [restic](https://restic.readthedocs.io/) or [rclone](https://rclone.org/docs/) to a different provider.

```cron
# /etc/cron.d/bobina
*  * * * * bobina curl -fsS -H "Authorization: Bearer $CRON_SECRET" https://bobina.example.com/api/cron/bobina-transfers
15 4 * * * root  /usr/local/bin/bobina-infra-backup.sh   # pg_dump + redis dump + rclone sync, then restic to off-box
```

Recreate every entry from both repos' `vercel.json` with the same schedules. Cron files don't expand variables from elsewhere, so load `CRON_SECRET` inside a wrapper script. Test a restore at least monthly. A backup you haven't restored is only a hope.

## 5. Monitoring

- Uptime checks on both homepages and one authenticated health path ([Uptime Kuma](https://github.com/louislam/uptime-kuma) or any hosted pinger).
- Collect `journalctl -u bobina-*` or container logs. Both apps log coded errors (`model_unavailable`, `session_store_unavailable`, `csrf_origin_mismatch`), so alert on those.
- Alert on disk usage (Postgres and blob volumes grow), Redis memory, and failed cron or backup runs.

## 6. Updates

```bash
cd /srv/Bobina.moe && git pull && pnpm install --frozen-lockfile && pnpm build && sudo systemctl restart bobina-moe
```

Take a backup first, apply any new `scripts/sql/*.sql` before restarting, and deploy Companion before Moe when a change spans both, unless the PR says otherwise.

## 7. Security

- **Firewall:** only 22 (ideally key-only, or limited to your IP), 80 and 443 open. For example, `ufw default deny incoming`, then allow `22,80,443/tcp`.
- **Never expose Redis (6379), SRH, Postgres (5432/54322), Supabase Studio, or the RustFS/MinIO console publicly.** Bind them to `127.0.0.1` or a private Docker network. Reach the admin consoles over an SSH tunnel.
- Docker publishes ports **past** ufw. Always bind them as `127.0.0.1:PORT:PORT`.
- Keep secrets in root-owned env files, use different values from dev, and never put them in git. Rotation rules are in [env-setup §5](./env-setup.md#5-rotation-what-each-rotation-breaks).
- Change all Supabase self-hosting default keys and passwords before first start ([guide](https://supabase.com/docs/guides/self-hosting/docker#securing-your-services)).
- Enable unattended security upgrades, and keep Node, Docker images and the apps patched.
