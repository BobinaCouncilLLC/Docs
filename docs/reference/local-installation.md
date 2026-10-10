# Local Installation

Part of the [Bobina Council Docs](../../README.md) · [Reference](./README.md)

How to run **Bobina.moe** (Moe, the site, auth and admin) and **BobinaCompanion** (Companion, Bobina's AI core, Telegram/Discord bots and MCP) on your own machine. For a server, see [Self-Hosting on a VPS](./self-hosting.md). For data, see [Backup & Restore](./backup-restore.md). Every variable is explained in [Environment Variable Setup](./env-setup.md).

> Items marked **(planned)** are part of the storage/database/Redis adapter work that comes with the backup rework (Moe #866). They describe the agreed design and may change.

## 1. Prerequisites

| Tool | Version | Why |
|---|---|---|
| [Node.js](https://nodejs.org/) | **20.9 or newer** (22 LTS recommended) | Both apps run Next.js 16, which needs Node 20.9+. Neither `package.json` pins `engines`, so this is the floor Next sets. |
| [pnpm](https://pnpm.io/installation) | **9 or newer** | Both lockfiles are `lockfileVersion: '9.0'`. `corepack enable` is the easiest install. |
| [Docker](https://docs.docker.com/get-docker/) | current | Local Supabase, Redis and RustFS/MinIO |
| [Supabase CLI](https://supabase.com/docs/guides/local-development/cli/getting-started) | current | Local Postgres + Auth/REST stack |
| Git, `openssl` | any | Cloning, generating keys |

## 2. Clone and install

```bash
git clone https://github.com/BobinaCouncil/Bobina.moe.git
git clone https://github.com/BobinaCouncil/BobinaCompanion.git
(cd Bobina.moe && pnpm install)
(cd BobinaCompanion && pnpm install)
```

## 3. Local Supabase

```bash
mkdir -p ~/bobina-local/supabase && cd ~/bobina-local/supabase
supabase init
supabase start          # prints API URL, anon key and service_role key
```

`supabase start` gives you a local API (default `http://127.0.0.1:54321`) and Postgres (`127.0.0.1:54322`). Use the printed **API URL** for `SUPABASE_URL` / `NEXT_PUBLIC_SUPABASE_URL` and the **service_role** key for `SUPABASE_SERVICE_ROLE_KEY`. Docs: [Supabase local development](https://supabase.com/docs/guides/local-development).

### Schema

The repos do **not** contain a full schema. They only contain incremental SQL:

- Moe: `scripts/sql/*.sql` (`achievement_awards`, `credit_purchase_claims`, `grok_grants`, `terms_acceptances`)
- Companion: `scripts/sql/*.sql` (RLS hardening, semantic-search index/function), `scripts/migrations/*.sql`, and the one-off `scripts/*.sql` files

To get a working database you need the base schema first:

1. **From production (recommended):** dump the schema only, no data:
   ```bash
   supabase link --project-ref <your-project-ref>
   supabase db dump -f schema.sql            # schema only by default
   psql "postgresql://postgres:postgres@127.0.0.1:54322/postgres" -f schema.sql
   ```
   See [`supabase db dump`](https://supabase.com/docs/reference/cli/supabase-db-dump).
2. **From a backup (planned):** the reworked admin backup includes the schema (tables, indexes, policies, functions), so you can restore it into a fresh local stack. See [Backup & Restore](./backup-restore.md#restore-into-a-fresh-local-or-vps-stack).
3. Then apply each repo's `scripts/sql/*.sql` files in date/name order. They're written to be re-runnable, but read each one before running it.

Admin backups need the `backup_list_tables()` function to exist. It's part of the production schema, so a dump includes it.

## 4. Local Redis

The two apps talk to Redis differently. This is the most common local pitfall.

| App | Client | Local option |
|---|---|---|
| Companion | `@upstash/redis` (REST) **or** `ioredis` (TCP) | Set `REDIS_DRIVER=redis` and `REDIS_URL=redis://127.0.0.1:6379`. Anything else, or `redis` without a `redis://`/`rediss://` URL, fails closed. |
| Moe | `@upstash/redis` (REST) only, via `KV_REST_API_URL` / `KV_REST_API_TOKEN` | Plain Redis does **not** speak REST. Run Upstash's [Serverless Redis HTTP (SRH)](https://upstash.com/docs/redis/sdks/ts/developing) proxy in front of a local Redis, or point Moe at a free Upstash dev database. A TCP adapter for Moe is **(planned)**. |

```yaml
# ~/bobina-local/docker-compose.yml
services:
  redis:
    image: redis:7
    ports: ["127.0.0.1:6379:6379"]
    volumes: ["redis-data:/data"]
    command: ["redis-server", "--appendonly", "yes"]
  srh:            # REST proxy for Moe
    image: hiett/serverless-redis-http:latest
    ports: ["127.0.0.1:8079:80"]
    environment:
      SRH_MODE: env
      SRH_TOKEN: local-dev-token        # any string; use the same value in KV_REST_API_TOKEN
      SRH_CONNECTION_STRING: redis://redis:6379
    depends_on: [redis]
  rustfs:
    image: rustfs/rustfs:latest
    ports: ["127.0.0.1:9000:9000", "127.0.0.1:9001:9001"]
    environment:
      RUSTFS_ACCESS_KEY: localaccess
      RUSTFS_SECRET_KEY: localsecret-change-me
    volumes: ["rustfs-data:/data"]
volumes:
  redis-data:
  rustfs-data:
```

```bash
cd ~/bobina-local && docker compose up -d
```

For Moe use `KV_REST_API_URL=http://127.0.0.1:8079` and `KV_REST_API_TOKEN=local-dev-token`. Point both apps at the same Redis, the way production shares one Upstash database.

## 5. Object storage (RustFS or MinIO)

Moe already has a storage adapter (`lib/storage`). The default is Vercel Blob, and `STORAGE_DRIVER=s3` switches to any S3-compatible store ([RustFS](https://docs.rustfs.com/), [MinIO](https://min.io/docs/minio/linux/index.html), AWS S3, R2).

1. Open the RustFS console at `http://127.0.0.1:9001` and create a bucket such as `bobina`.
2. Allow anonymous **read** on the public prefixes (or the whole bucket for local dev), and keep `backups/` private.
3. Set these in Moe:

```env
STORAGE_DRIVER=s3
S3_BUCKET=bobina
S3_ENDPOINT=http://127.0.0.1:9000
S3_REGION=us-east-1
S3_ACCESS_KEY_ID=localaccess
S3_SECRET_ACCESS_KEY=localsecret-change-me
S3_FORCE_PATH_STYLE=true
# Leave S3_PUBLIC_BASE_URL unset locally: Moe derives it as S3_ENDPOINT + "/" + S3_BUCKET
# (path-style → http://127.0.0.1:9000/bobina/<key>). Set it only when a CDN or
# different public hostname should be persisted instead of the endpoint URL.
```

Companion also speaks S3 when `STORAGE_DRIVER=s3`: it uses the shared `S3_*` credentials plus **`GROK_MEDIA_S3_BUCKET`** for private Grok turn media (presigned reads; it does **not** read `S3_PUBLIC_BASE_URL` or `S3_BUCKET`). Default remains Vercel Blob via `GROK_MEDIA_BLOB_READ_WRITE_TOKEN`.

## 6. Environment variables

Copy and fill in `.env.local` in each repo. Never commit it. The full list, with generation commands, is in [Environment Variable Setup](./env-setup.md). This is the minimum for a working local stack:

**BobinaCompanion `.env.local`** (start from `.env.example`)

```env
AI_GATEWAY_API_KEY=            # required off-Vercel, https://vercel.com/docs/ai-gateway/authentication
BOBINA_AI_MODEL=spacexai/grok-4.3
BOBINA_FAST_MODEL=spacexai/grok-4.1-fast-reasoning
SUPABASE_URL=http://127.0.0.1:54321
NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
SUPABASE_SERVICE_ROLE_KEY=     # from `supabase start`
SUPABASE_ANON_KEY=             # from `supabase start`
REDIS_DRIVER=redis
REDIS_URL=redis://127.0.0.1:6379
COMPANION_API_SECRET=          # openssl rand -base64 48, same value in Moe
BOBINA_COUNCIL_URL=http://localhost:3000
BOBINA_MOE_URL=http://localhost:3000
NEXT_PUBLIC_BASE_URL=http://localhost:3001
CRON_SECRET=                   # openssl rand -base64 48
BOBINA_HASH_SECRET=            # openssl rand -base64 48
COMPANION_USER_TOKEN_PUBLIC_KEY=   # P-256 public PEM, see env-setup §2
```

**Bobina.moe `.env.local`**

```env
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=               # openssl rand -base64 48
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_COMPANION_URL=http://localhost:3001
COMPANION_API_URL=http://localhost:3001
COMPANION_API_SECRET=          # same as Companion
COMPANION_USER_TOKEN_PRIVATE_KEY=  # matching P-256 private PEM
SUPABASE_URL=http://127.0.0.1:54321
NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
SUPABASE_SERVICE_ROLE_KEY=
KV_REST_API_URL=http://127.0.0.1:8079
KV_REST_API_TOKEN=local-dev-token
CRON_SECRET=
IP_HASH_KEY=                   # 32+ chars
BOBINA_HASH_SECRET=
COUNCIL_ADMIN_USERNAMES=       # your bc_ id, makes you env admin locally
AI_GATEWAY_API_KEY=            # for voice/STT and other Moe AI calls
# plus STORAGE_DRIVER / S3_* from §5
```

Add OAuth provider credentials (Discord, X, Telegram login) only for the providers you want to test. Each provider needs `http://localhost:3000` callback URLs registered in its developer portal (see env-setup §4).

Use **different** secrets locally than in production, and never copy production `IP_HASH_KEY`, `MERCH_PII_ENCRYPTION_KEY` or `GROK_WEBHOOK_ENCRYPTION_KEY` to a dev machine unless you're restoring production data on purpose (see [Backup & Restore](./backup-restore.md#key-and-rotation-caveats)).

## 7. Run both apps and link them

```bash
(cd Bobina.moe && pnpm dev)                   # http://localhost:3000
(cd BobinaCompanion && pnpm dev --port 3001)  # http://localhost:3001
```

The link between them is `COMPANION_API_URL` / `NEXT_PUBLIC_COMPANION_URL` on Moe, `BOBINA_COUNCIL_URL` / `BOBINA_MOE_URL` on Companion, the shared `COMPANION_API_SECRET`, and the user-token key pair. If any of these don't match, Moe → Companion calls come back as 401/403.

`pnpm build` in Companion also runs `discord:register`. Locally, run `pnpm build:skills` once and use `pnpm dev`. Only run `pnpm discord:register` against a **dev** Discord application.

## 8. Discord and Telegram in development

Both platforms need a public HTTPS URL that reaches your local Companion. Use a tunnel:

- [cloudflared quick tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/trycloudflare/): `cloudflared tunnel --url http://localhost:3001`
- [ngrok](https://ngrok.com/docs/getting-started/): `ngrok http 3001`

Always use **separate dev bots**. Don't point the production bot's webhook at your laptop.

**Telegram:** create a dev bot with [@BotFather](https://core.telegram.org/bots#how-do-i-create-a-bot), then register the webhook ([setWebhook](https://core.telegram.org/bots/api#setwebhook)):

```bash
curl -s "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/setWebhook" \
  -d "url=https://<tunnel-host>/api/telegram" \
  -d "secret_token=$TELEGRAM_WEBHOOK_SECRET"     # openssl rand -hex 32
```

The webhook route is `app/api/telegram/route.tsx`. When you're done, call `deleteWebhook` or point it back.

**Discord:** in the [Developer Portal](https://discord.com/developers/applications), set your dev app's **Interactions Endpoint URL** to `https://<tunnel-host>/api/discord`, set `DISCORD_PUBLIC_KEY` / `DISCORD_BOT_TOKEN` / `DISCORD_GUILD_ID` for the dev app, and run `pnpm discord:register`.

## 9. Crons locally

Vercel runs the crons in each repo's `vercel.json` (for example, Moe has `/api/cron/bobina-transfers` every minute and Companion has `/api/cron/reminders` every minute). Locally, nothing runs them, so call them by hand when you need one:

```bash
curl -H "Authorization: Bearer $CRON_SECRET" http://localhost:3000/api/cron/refresh-users
curl -H "Authorization: Bearer $CRON_SECRET" http://localhost:3001/api/cron/reminders
```

For a loop, use `watch -n 60 curl ...`, or add a crontab entry pointing at localhost.

## 10. Smoke test

1. `http://localhost:3000` loads, and you can sign in with one provider.
2. Your `bc_` id is in `COUNCIL_ADMIN_USERNAMES`, and `/admin` shows every tab.
3. Chat with Bobina on web: you get a reply, and the council session streams live.
4. Redis: `redis-cli -p 6379 dbsize` grows after you sign in.
5. Storage: uploading a profile image creates an object in the RustFS bucket.
6. Telegram/Discord dev bot replies through the tunnel.
7. `pnpm test` and `pnpm typecheck` pass in both repos.

## 11. Troubleshooting

| Symptom | Cause / fix |
|---|---|
| Moe Redis errors such as `fetch failed` or `invalid URL` against `redis://` | Moe only speaks Upstash **REST**. Use an `http(s)://` REST URL (SRH or Upstash), not `redis://`. |
| Companion: `REDIS_DRIVER=redis needs REDIS_URL` | Set `REDIS_URL=redis://…` or unset `REDIS_DRIVER` to use REST. |
| `csrf_origin_mismatch` 403 on writes | Moe's proxy only accepts cookie-authenticated writes from its own origin. Open the site at the same host as `NEXTAUTH_URL` (`localhost` vs `127.0.0.1` count as different origins). |
| Signed in but logged out after a redirect, or the cookie is missing | Cookie domain/secure flag mismatch. Use `http://localhost:3000` throughout, and keep `NEXTAUTH_URL` exact. |
| Browser CORS errors calling Companion | Companion must allow Moe's local origin. Match `BOBINA_COUNCIL_URL` to the exact URL you're using. |
| Bobina says her brain is offline, `model_unavailable` | `BOBINA_AI_MODEL` / `BOBINA_FAST_MODEL` are unset or the Gateway key is missing. These fail closed by design and refund the turn. |
| S3 uploads work locally but fail in production | Production `S3_ENDPOINT` and `S3_PUBLIC_BASE_URL` must be **HTTPS**. Mixed content and presigned uploads break over plain HTTP. |
| Admin backup fails at discovery | `backup_list_tables()` is missing from your local schema. Restore the schema from a dump (§3). |
| Telegram webhook gets no updates | Run `getWebhookInfo`. The secret must be hex (`openssl rand -hex 32`), and the tunnel URL changes every time cloudflared/ngrok restarts. |
