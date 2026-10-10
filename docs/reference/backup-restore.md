# Backup & Restore

Part of the [Bobina Council Docs](../../README.md) · [Reference](./README.md)

> ⚠️ **Backup/restore is being reworked in Moe #866; commands and flags marked (planned) may change.**
> Anything not marked (planned) describes what is on `main` today (`lib/backup.ts`, `lib/backup-restore.ts`, `app/api/admin/backup/*`).

Related: [Local Installation](./local-installation.md) · [Self-Hosting on a VPS](./self-hosting.md) · [Environment Variable Setup](./env-setup.md)

## 1. How the backup decides what to include

The backup is **discovered from the code and live stores, not from a hand-written list**, so a new table, key or file is included automatically.

- **Supabase:** every base table in `public` is found at runtime through the `backup_list_tables()` RPC, which also returns each table's primary key (used as the upsert target on restore). If discovery fails, the backup **fails loudly** instead of saving a partial copy.
- **Redis:** the whole keyspace is walked with `SCAN` (hashes with `HSCAN`), and each key's type and TTL is recorded in `redis/_index.json`.
- **Blob:** every object is listed, except earlier backups under `backups/`.
- **(planned)** Instead of a manifest, there's only a **skip list** plus stats for what was skipped: temp, dedupe and lock keys, private Grok media, and Companion private member files. The backup reports counts for everything included and skipped.

### Included and excluded

| | Today (`main`) | After #866 **(planned)** |
|---|---|---|
| Redis / Upstash keys | All keys, with type + TTL | All keys except the skip list (temp/dedupe/lock) |
| Supabase **data** | All `public` tables via `backup_list_tables()` | Same |
| Supabase **schema** (tables, indexes, policies, functions) | ❌ not included. Use `supabase db dump` | ✅ included |
| Public blob hierarchy | All objects (full archive includes the bytes) | Kept as-is, minus Companion private files |
| Private Grok media, Companion private member files | (not separately handled) | ❌ excluded |
| Where archives are stored | `backups/` in the active store. Private on S3 or a private Blob store. On a public Vercel Blob store they fall back to public with unguessable keys, and downloads still go through an authed proxy. | **Private backup store only, never public blob** |

## 2. Permissions

Only staff with the **Backup** tab plus the matching scope, checked on the server (coded 403, or 503 if the store is unreachable):

| Action | Requires |
|---|---|
| Create / list / download a backup, download the full archive | `backup` tab + `downloadBackup` scope |
| Restore, delete a backup | `backup` tab + `restoreBackup` scope |

Since #875, only env admins (`COUNCIL_ADMIN_USERNAMES`) can grant these scopes.

## 3. Creating a backup

**In the admin UI:** Admin → **Backup** → create a backup. The result is `backups/council-backup-<timestamp>.zip`, which contains Redis files, Supabase table files, `supabase/_schema.json` (table list + primary keys), `redis/_index.json` and `blob/manifest.json`, plus per-blob snapshots under `backups/<stem>-blobs/`.

**Full (offline) archive:** use **Download full** (`GET /api/admin/backup/download-full`). It streams one self-contained zip:

```
full-backup-<name>/
  data.zip      Redis + Supabase + manifests
  blobs/<path>  raw bytes of every blob at its original pathname
  RESTORE.md    restore instructions
```

Keep this file. It's what you restore from on a fresh VPS or local stack, because it doesn't rely on anything still sitting in the old store.

**(planned)** Scheduled auto-backups write to the private backup store on a schedule, with retention. For off-box copies, see [self-hosting §4](./self-hosting.md#4-scheduled-backups-with-off-box-copies). There are no CLI scripts for app backups on `main` yet. Any script or flag names that come with #866 are **(planned)**.

## 4. Verifying a backup

1. Open the zip and check that `supabase/_schema.json` lists every table you expect. Compare it to `select count(*) from information_schema.tables where table_schema='public' and table_type='BASE TABLE';`
2. Check that `redis/_index.json`'s key count is close to `DBSIZE` at the time of the backup (TTL keys drift).
3. Check that `blob/manifest.json`'s count matches the `blobs/` file count in the full archive.
4. **(planned)** Compare the included/skipped stats the backup prints.
5. Best check: restore it into a throwaway local stack (§6) and run the [smoke test](./local-installation.md#10-smoke-test).

## 5. Restoring (admin UI)

`POST /api/admin/backup/restore` accepts either:

- `{ path | backupUrl }`: a backup that's already in this store, or
- `{ uploadedArchivePath, kind: "full" }`: an uploaded **full** archive (Admin → Backup → **Restore from File**).

Before restoring, the server takes a **pre-restore safety backup** (`pre-restore-backup-…`) unless `skipPreBackup: true` is sent. Don't skip it. If that safety backup fails, it logs a warning and keeps going, so check the logs.

A full-archive restore:
1. restores Redis and Supabase from `data.zip`,
2. re-uploads every file under `blobs/` at the same pathname, then
3. rewrites old blob URLs in the restored data to the new store's origin.

Rows are **upserted** on primary key. Rows that exist now but aren't in the backup are **not deleted**. For a clean point-in-time restore, restore into an empty database.

### Restore order

Always go **schema → data → Redis → blob**:

1. **Schema:** today from `supabase db dump` (`psql -f schema.sql`), or from the backup's schema section **(planned)**. Then apply the repos' `scripts/sql/*.sql`.
2. **Supabase data:** upsert each table file, using the primary keys from `_schema.json`.
3. **Redis:** recreate each key by type (`SET`, `RPUSH`, `SADD`, `ZADD`, `HSET`), then `PEXPIRE` where ttl > 0.
4. **Blob:** upload at the same pathnames, then rewrite the old origin to the new one.

The in-app restore does steps 2–4 for you. Step 1 is manual until #866.

## 6. Restore into a fresh local or VPS stack

1. Bring up Supabase, Redis and RustFS/MinIO ([local](./local-installation.md) or [VPS](./self-hosting.md)).
2. Apply the schema (step 1 above) so `backup_list_tables()` exists.
3. Set the env vars, **including the same encryption and hash keys as the source** (§7).
4. Start Moe and sign in as an env admin (`COUNCIL_ADMIN_USERNAMES`). The restore needs the `restoreBackup` scope, which env admins always have.
5. Admin → Backup → **Restore from File**, then pick the full archive.
6. Run the smoke test, then re-point the Telegram webhook, Discord interactions URL and OAuth callbacks.

**(planned)** Because storage, database and Redis sit behind adapters, moving from Vercel/Supabase cloud/Upstash to RustFS/self-hosted Supabase/local Redis is a config change: take a backup on the old stack, change the env vars, and restore on the new one. Upstash is REST and local Redis is TCP. Companion already switches with `REDIS_DRIVER=redis` + `REDIS_URL`. Moe needs its planned TCP adapter, or the SRH REST proxy in the meantime.

## 7. Key and rotation caveats

Some data in the backup can only be read with the **same keys** that wrote it. Restore with the source's values, or the data is unreadable:

| Key | If it doesn't match |
|---|---|
| `IP_HASH_KEY` (Moe) | Every stored IP hash stops matching, so **IP bans stop working** |
| `MERCH_PII_ENCRYPTION_KEY` (Moe) | Encrypted merch shipping addresses can't be decrypted. Never rotate without re-encrypting first. |
| `GROK_WEBHOOK_ENCRYPTION_KEY` | Stored Grok webhook secrets can't be decrypted |
| `BOBINA_HASH_SECRET` | Low impact: terms-prompt rate-limit markers reset, and old acceptance IP hashes go stale (acceptances stay valid) |
| `NEXTAUTH_SECRET` | Everyone gets signed out (no data loss) |

Store these keys **separately from the backups**, for example in a password manager. Keeping both in the same place means one leak exposes everything.

## 8. Member export / import (planned)

This is a separate stacked PR on top of #866, so everything in this section is **(planned)**.

- Each member can export and import **their own** data: memories, profiles, Bobina settings, turns, council sessions and feedback.
- **Excluded:** messages, achievements, credits, tokens and the Moe profile. These are account and economy records, so a member can't move or re-import them.
- An import only applies to the signed-in member's own account.

## 9. Disaster-recovery checklist

- [ ] Recent **full archive** downloaded and stored off-box (two locations, at least one with a different provider)
- [ ] `IP_HASH_KEY`, `MERCH_PII_ENCRYPTION_KEY`, `GROK_WEBHOOK_ENCRYPTION_KEY`, `NEXTAUTH_SECRET`, `COMPANION_API_SECRET` and the user-token key pair stored safely, **apart from** the backups
- [ ] Schema dump (`supabase db dump`) kept with each backup until #866 includes the schema
- [ ] Restore tested into a throwaway stack within the last month
- [ ] Your `bc_` id is in `COUNCIL_ADMIN_USERNAMES` on the target, so you can run the restore
- [ ] After restore: smoke test, Telegram `setWebhook`, Discord interactions URL, OAuth callbacks, crons re-enabled
- [ ] Rotate any secret that may have leaked during the incident. Rotate the keys in §7 only after re-encrypting.
