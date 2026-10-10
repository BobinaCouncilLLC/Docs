# Environment Variable Setup

Part of the [Bobina Council Docs](../../README.md) · [Reference](./README.md)

This guide covers every environment variable that **Bobina.moe** (`BobinaCouncil/Bobina.moe`, Vercel project `bobinacouncil`) and **BobinaCompanion** (`BobinaCouncil/BobinaCompanion`, Vercel project `bobinacompanion`) read. The list comes from the code on `main` (`process.env` reads), checked against the variable names set in Vercel. It contains no values. Never paste a real secret into a doc, an issue, a PR or a chat.

**Columns**
- **App:** Moe = Bobina.moe, Comp = BobinaCompanion.
- **Req:** R = required (the feature fails closed or the app won't work without it), O = optional (has a default or switches a feature off).
- **Env:** where to set it in Vercel. P = Production, Pr = Preview, D = Development. Secrets are **P only** unless a preview really needs them.
- **Sens:** Yes = mark as **Sensitive** in Vercel. Sensitive values can't be read back, only replaced.

---

## 1. Quick start

1. Open the project in Vercel: **Settings → Environment Variables** ([docs](https://vercel.com/docs/environment-variables)).
2. Work through the groups below. For each secret, generate a new value (section 2). Never reuse one value for two variables.
3. Add it, tick only the environments listed, and turn on **Sensitive** where marked ([sensitive env vars](https://vercel.com/docs/environment-variables/sensitive-environment-variables)).
4. Redeploy. Env changes only take effect on the next deployment.
5. Local development: `vercel env pull .env.local` only pulls non-sensitive Development values. Put local-only secrets in `.env.local` yourself (it's gitignored). Companion ships an `.env.example` for the AI section.

## 2. Generating keys

| Kind | Command | Used for |
|---|---|---|
| Generic secret / HMAC key | `openssl rand -base64 48` | `BOBINA_HASH_SECRET`, `IP_HASH_KEY` (32+ chars required), `CRON_SECRET`, `COMPANION_API_SECRET`, `MCP_TOKEN_SECRET`, `NEXTAUTH_SECRET`, `TELEGRAM_WEBHOOK_SECRET`*, `TELEGRAM_BOT_SECRET`, `COMPANION_SECRET` |
| 32-byte AES key | `openssl rand -base64 32` | `MERCH_PII_ENCRYPTION_KEY` (must decode to exactly 32 bytes), `GROK_WEBHOOK_ENCRYPTION_KEY` (32 bytes, base64 or 64 hex chars) |
| P-256 (ES256) key pair | see below | `COMPANION_USER_TOKEN_PRIVATE_KEY` (Moe) / `COMPANION_USER_TOKEN_PUBLIC_KEY` (Comp), `OAUTH_JWT_PRIVATE_KEY` / `OAUTH_JWT_PUBLIC_KEY` (Moe) |

\* Telegram only allows `A-Z a-z 0-9 _ -` in the webhook secret (1–256 chars). Use `openssl rand -hex 32` for `TELEGRAM_WEBHOOK_SECRET`.

Every secret gets its own value, is marked Sensitive and goes into Production only unless the table says otherwise.

### P-256 key pair (Companion user tokens)

Bobina.moe signs per-member Companion tokens with **ES256**. The private key is read with Node's `createPrivateKey` (PKCS#8 or SEC1 PEM, or base64 DER) and must be P-256. Companion verifies with `createPublicKey` (SPKI PEM).

```bash
openssl ecparam -name prime256v1 -genkey -noout -out priv.pem
openssl pkcs8 -topk8 -nocrypt -in priv.pem -out companion_user_token_private.pem
openssl ec -in priv.pem -pubout -out companion_user_token_public.pem
```

- Paste the **whole** file, including the `-----BEGIN ...-----` and `-----END ...-----` lines, into the Vercel value box. Multi-line values are fine.
- `companion_user_token_private.pem` → **Moe** `COMPANION_USER_TOKEN_PRIVATE_KEY` (P, Sensitive).
- `companion_user_token_public.pem` → **Comp** `COMPANION_USER_TOKEN_PUBLIC_KEY` (P, Sensitive is optional because it's a public key, but recommended).
- Delete the three `.pem` files afterwards (`shred -u *.pem` or move them to your password manager).

Use the same commands, and a **separate** pair, for `OAUTH_JWT_PRIVATE_KEY` / `OAUTH_JWT_PUBLIC_KEY` (Login with Bobina.moe ID tokens).

---

## 3. Variables by group

### Auth / session

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `NEXTAUTH_SECRET` | Moe | R | P (+Pr if previews sign in) | Yes | Signs/encrypts NextAuth session JWTs | `openssl rand -base64 48` |
| `NEXTAUTH_URL` | Moe | R | P | No | Canonical site URL for auth callbacks | `https://bobina.moe` |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | Moe | O | P | Secret: Yes | Google sign-in / account linking | [Google Cloud Console → Credentials](https://console.cloud.google.com/apis/credentials), OAuth client (Web), redirect `https://bobina.moe/api/auth/callback/google` |
| `DISCORD_CLIENT_ID` / `DISCORD_CLIENT_SECRET` | Moe | O | P | Secret: Yes | Discord sign-in / linking | Discord portal → your app → **OAuth2** (see §4) |
| `TWITTER_CLIENT_ID` / `TWITTER_CLIENT_SECRET` | Moe | O | P | Secret: Yes | X (OAuth 2.0) sign-in / linking | X portal → app → **Keys and tokens → OAuth 2.0 Client ID and Secret** |
| `FACEBOOK_CLIENT_ID` / `FACEBOOK_CLIENT_SECRET` | Moe | O | P | Secret: Yes | Facebook linking | [Meta for Developers → My Apps](https://developers.facebook.com/apps/) → App settings → Basic |
| `FACEBOOK_APP_ID` / `FACEBOOK_APP_SECRET` | Moe | O | P | Secret: Yes | Legacy names still read in one place; set the same as above only if that path is used | same |
| `INSTAGRAM_CLIENT_ID` / `INSTAGRAM_CLIENT_SECRET` (+ legacy `INSTAGRAM_APP_ID` / `_SECRET`) | Moe | O | P | Secret: Yes | Instagram linking | Meta app → Instagram product |
| `TIKTOK_CLIENT_KEY` / `TIKTOK_CLIENT_SECRET` | Moe | O | P | Secret: Yes | TikTok linking | [TikTok for Developers](https://developers.tiktok.com/apps/) |
| `SIWE_ALLOWED_DOMAINS` | Moe | O | P | No | Extra domains accepted in Sign-In with Ethereum messages | comma-separated hostnames |
| `NEXT_PUBLIC_RECAPTCHA_SITE_KEY` / `RECAPTCHA_SECRET_KEY` | Moe | O | P | Secret: Yes | reCAPTCHA on forms | [reCAPTCHA admin](https://www.google.com/recaptcha/admin) |
| `OAUTH_JWT_PRIVATE_KEY` / `OAUTH_JWT_PUBLIC_KEY` | Moe | R for Login with Bobina | P | Yes | Signs OAuth/OIDC ID tokens for third-party apps | P-256 pair (§2) |
| `GROK_OAUTH_REDIRECT_URIS`, `GROK_REDIRECT_URIS` | Moe | O | P | No | Allowed redirect URIs for the Grok Bot OAuth client | comma-separated URLs |
| `MCP_TOKEN_SECRET` | Moe + Comp | R | P | Yes | HMAC for signed MCP identity tokens; **same value in both apps** | `openssl rand -base64 48` (16+ chars) |
| `MCP_REQUIRE_SIGNED_TOKEN`, `MCP_ALLOW_LEGACY_HEADER_AUTH` | Comp | O | P | No | Hardening switches for MCP auth | `true` / `false` |

### Admin

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `COUNCIL_ADMIN_USERNAMES` | Moe | R | P | Yes | Env admins: comma-separated `bc_…` ids (or usernames) that hold every tab and scope. After Moe #875 they are the **only** ones who can grant Staff Permissions, change scopes, or assign Council Admin/Mod. Keep at least one id here. | your `bc_` id from your profile |
| `COUNCIL_MOD_USERNAMES` | Moe | — | — | — | **Being removed** by Moe #875. Delete it from Vercel after #875 deploys. | — |
| `ADMIN_TELEGRAM_USER_IDS` | Moe | O | P | No | Telegram user ids allowed to run admin bot actions on Moe | numeric ids, comma-separated |
| `TELEGRAM_ADMIN_USER_IDS` | Comp | O | P | No | Same idea for the Companion bot | numeric ids |
| `DISCORD_ADMIN_ROLE_IDS` | Comp | O | P | No | Discord role ids treated as admin by the bot | Discord → Developer Mode → right-click role → Copy ID |

### AI Gateway and models

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `AI_GATEWAY_API_KEY` | Moe + Comp | O on Vercel / R locally | D (local) | Yes | Auth for the Vercel AI Gateway. On Vercel the project's **OIDC token** (`VERCEL_OIDC_TOKEN`, injected automatically) is used, so only set this for local dev or off-Vercel. | [AI Gateway → API keys](https://vercel.com/docs/ai-gateway/authentication) |
| `BOBINA_AI_MODEL` | Comp | **R** | P, Pr | No | Bobina's own model (final reply, stance, vision, /opinion, observation). **No default. If unset, those calls fail closed (`model_unavailable`, refunded).** | Gateway id, e.g. `spacexai/grok-4.3` ([model list](https://vercel.com/ai-gateway/models)) |
| `BOBINA_FAST_MODEL` | Comp | **R** | P, Pr | No | Planner, minions, recall, profile, sentiment. **No default and no fallback to `BOBINA_AI_MODEL`.** | Gateway id |
| `BOBINA_EMBEDDING_MODEL` | Comp | O | P | No | Embeddings (default `openai/text-embedding-3-small`; stored vectors are 1536-dim, so changing it means re-embedding) | Gateway id |
| `BOBINA_IMAGE_MODEL` | Comp | O | P | No | Image generation (default `google/gemini-3-pro-image`) | Gateway id |
| `BOBINA_STT_MODEL` | Moe | O | P | No | Speech-to-text model for voice input | Gateway id |
| `XAI_API_KEY`, `XAI_BASE_URL`, `BOBINA_NEWS_SEARCH_MODEL` | Comp | O | P | Key: Yes | Direct xAI calls for /opinion live search | [xAI console](https://console.x.ai/) |
| `ELEVENLABS_API_KEY` (+ `ELEVENLABS_MODEL_ID`, `ELEVENLABS_VOICE_ID`, `ELEVENLABS_BASE_URL`) | Comp | R for voice | P | Key: Yes | Text-to-speech | [ElevenLabs → API keys](https://elevenlabs.io/app/settings/api-keys) |
| `EMOTION_LAYER_V2`, `MEMORY_CARDS_V1`, `GROK_MCP_V1`, `PASSIVE_ONLY` | Comp | O | P | No | Feature flags | `1` / `true` to enable |

### Redis / Upstash

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `KV_REST_API_URL` / `KV_REST_API_TOKEN` | Moe + Comp | **R** | P (+Pr if previews need data) | Token: Yes | Main Redis (users, credits, rate limits, sessions) | Upstash console → database → **REST API** (§4) — use the **read-write** token |
| `UPSTASH_REDIS_REST_URL` / `UPSTASH_REDIS_REST_TOKEN` | Moe + Comp | O | P | Token: Yes | Older alias for the same database, read by a couple of helpers | same values as `KV_*` |
| `REDIS_DRIVER`, `REDIS_URL` | Comp | O | — | Yes | Self-host only: switch to a TCP Redis | `redis://…` |

### Supabase

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `SUPABASE_URL` | Moe + Comp | **R** | P, Pr | No | Project API URL | Supabase → Project Settings → **API** |
| `NEXT_PUBLIC_SUPABASE_URL` | Moe + Comp | R | P, Pr | No | Same URL, exposed to the browser | same |
| `SUPABASE_SERVICE_ROLE_KEY` | Moe + Comp | **R** | P | Yes | Server-side DB access (bypasses RLS; never ship to the browser) | Project Settings → API → `service_role` (or the new **secret** key) |
| `SUPABASE_ANON_KEY` | Comp | O | P | No | Public anon key, used in one client path | Project Settings → API → `anon` / publishable key |
| `POSTGRES_URL`, `POSTGRES_URL_NON_POOLING` | Comp | O | P | Yes | Direct Postgres for scripts/migrations | Supabase → **Connect** |

### Blob / storage

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `BLOB_READ_WRITE_TOKEN` | Moe + Comp | R | P | Yes | Public asset/voice Blob store (read implicitly by `@vercel/blob`) | Vercel → **Storage → Blob → Connect to project** adds it automatically ([docs](https://vercel.com/docs/vercel-blob)) |
| `BLOB_STORE_ACCESS` | Moe | O | P | No | Set `private` only if the store was created as private | `private` |
| `GROK_MEDIA_BLOB_READ_WRITE_TOKEN` | Comp | R for Grok Bot media | P | Yes | A **separate private** Blob store for short-lived Grok Bot turn media | Create a second Blob store (private) and copy its read-write token into this name |
| `STORAGE_DRIVER` + `S3_ENDPOINT`, `S3_REGION`, `S3_BUCKET`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`, `S3_PUBLIC_BASE_URL`, `S3_FORCE_PATH_STYLE`, `GROK_MEDIA_S3_BUCKET` | Moe + Comp | O | — | Keys: Yes | Self-host/VPS only: `STORAGE_DRIVER=s3` swaps Blob for any S3-compatible bucket | your S3 provider |

### Discord

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `DISCORD_BOT_TOKEN` | Comp (+Moe) | R | P | Yes | Bot login / REST calls | Portal → **Bot → Reset Token** |
| `DISCORD_BOT_ID` | Comp | R | P | No | Application (bot) id | Portal → **General Information → Application ID** |
| `DISCORD_PUBLIC_KEY` | Comp + Moe | R | P | No | Verifies interaction signatures (`/api/discord`) | Portal → **General Information → Public Key** |
| `DISCORD_GUILD_ID` | Comp + Moe | R | P | No | The Bobina Council server | Developer Mode → right-click server → Copy Server ID |
| `DISCORD_AUTH_CHANNEL`, `DISCORD_NOTIFICATION_CHANNEL_ID`, `DISCORD_OBSERVATION_CHANNEL_ID`, `DISCORD_VERIFIED_CHANNEL_ID` | Comp | O | P | No | Channel ids for verification, notices and observations | right-click channel → Copy Channel ID |
| `DISCORD_ROLE_COUNCIL_ADMIN`, `DISCORD_ROLE_COUNCIL_MOD`, `DISCORD_ROLE_COUNCIL_ELDER`, `DISCORD_ROLE_COUNCIL_MEMBER`, `DISCORD_ROLE_BOBINA_MAKER`, `DISCORD_ROLE_BEARSEXUAL`, `DISCORD_ROLE_BOBINACHAD`, `DISCORD_ROLE_BUSYBEE`, `DISCORD_ROLE_HONEY` | Comp | O | P | No | Role ids the bot syncs | right-click role → Copy Role ID |
| `DISCORD_ARTICLES_WEBHOOK_URL`, `DISCORD_CHANGELOG_WEBHOOK_URL`, `DISCORD_HOT_BOBINAS_WEBHOOK_URL`, `DISCORD_MEMBERS_WEBHOOK_URL`, `DISCORD_MERCH_WEBHOOK_URL`, `DISCORD_PROPOSALS_WEBHOOK_URL`, `DISCORD_VOTES_WEBHOOK_URL` | Moe | O | P | Yes | Channel webhooks for site announcements (the URL is the secret) | Channel → **Edit Channel → Integrations → Webhooks → New Webhook → Copy URL** |

### Telegram

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `TELEGRAM_BOT_TOKEN` | Comp + Moe | R | P | Yes | Bot API token | @BotFather `/newbot` or `/token` (§4) |
| `TELEGRAM_WEBHOOK_SECRET` | Comp (+Moe) | R | P | Yes | Must match the `X-Telegram-Bot-Api-Secret-Token` header Telegram sends to `/api/telegram` | `openssl rand -hex 32`, then `setWebhook` (§4) |
| `TELEGRAM_BOT_USERNAME` | Moe | R for TG login | P | No | Bot username for the login widget | from BotFather (no `@`) |
| `TELEGRAM_BOT_SECRET` | Moe | O | P | Yes | Shared secret for Moe's Telegram bot endpoints | `openssl rand -base64 48` |
| `TELEGRAM_BOT_CHANNEL_ID`, `TELEGRAM_GROUP_ID` | Moe / Comp | O | P | No | Channel / group the bots post to | forward a message to @userinfobot or read `chat.id` from an update |

### X / Twitter

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `TWITTER_API_KEY` / `TWITTER_API_SECRET` | Moe + Comp | O | P | Yes | OAuth 1.0a consumer key/secret (posting as the Bobina account) | X portal → app → **Keys and tokens → Consumer Keys** |
| `TWITTER_ACCESS_TOKEN` | Moe + Comp | O | P | Yes | OAuth 1.0a user access token | **Keys and tokens → Access Token and Secret** (set app permissions to Read and Write **first**) |
| `TWITTER_ACCESS_TOKEN_SECRET` | Moe | O | P | Yes | Access token secret (Moe's name) | same |
| `TWITTER_ACCESS_SECRET` | Comp | O | P | Yes | Access token secret (**Companion's name differs from Moe's**) | same value as above |
| `TWITTER_BEARER_TOKEN` | Comp | O | P | Yes | App-only bearer for read lookups | **Keys and tokens → Bearer Token** |
| `X_LOOKUPS_PER_MINUTE`, `X_ONLY` | Comp | O | P | No | Lookup rate cap / X-only mode | number / flag |

### Coinbase / onramp and chain data

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `COINBASE_API_KEY_NAME` / `COINBASE_API_KEY_SECRET` | Moe | R for onramp | P | Yes | CDP Secret API key used to mint onramp session tokens | [CDP Portal → API Keys](https://portal.cdp.coinbase.com/) → Create Secret API key; copy the key name/id and the private key |
| `BASE_RPC_URL`, `ETHEREUM_RPC_URL`, `INK_RPC_URL`, `NEXT_PUBLIC_SOLANA_RPC_URL` | Moe | O | P | RPC with key: Yes | RPC endpoints | your RPC provider |
| `BASESCAN_API_KEY`, `ETHERSCAN_API_KEY`, `MORALIS_API_KEY`, `OPENSEA_API_KEY`, `DUNE_API_KEY` | Moe | O | P | Yes | Chain/NFT data | [Etherscan](https://etherscan.io/myapikey) (one key covers Basescan via API v2), [Moralis](https://admin.moralis.com/), [OpenSea](https://docs.opensea.io/reference/api-keys), [Dune](https://dune.com/settings/api) |
| `COINMARKETCAP_API_KEY` | Comp | O | P | Yes | Token prices | [CoinMarketCap API](https://pro.coinmarketcap.com/account) |

### Merch / PII

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `MERCH_PII_ENCRYPTION_KEY` | Moe | **R** for merch | P | Yes | AES-256-GCM key for shipping addresses. Must base64-decode to **exactly 32 bytes** or merch fails closed. | `openssl rand -base64 32`. **Never rotate without re-encrypting** (§5) |
| `LEGAL_NOTICE_ADDRESS` | Moe | O | P | No | Address shown on legal pages | text |

### Hashing, crons and webhooks

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `IP_HASH_KEY` | Moe | **R** | P | Yes | HMAC key for every stored IP hash, IP bans included. 32+ chars or sign-in IP checks throw. | `openssl rand -base64 48`. **Do not rotate** |
| `BOBINA_HASH_SECRET` | Moe + Comp (separate values are fine) | R | P | Yes | Moe: terms-acceptance IP hashes, proposal/submission hashes, `/api/companion/lookup` auth. Comp: terms IP hashes and `/terms` prompt rate-limit markers. | `openssl rand -base64 48` |
| `CRON_SECRET` | Moe + Comp | **R** | P | Yes | Vercel sends `Authorization: Bearer $CRON_SECRET` to cron routes; routes reject anything else | `openssl rand -base64 48` ([docs](https://vercel.com/docs/cron-jobs/manage-cron-jobs#securing-cron-jobs)) |
| `COMPANION_SECRET` | Comp | O | P | Yes | Legacy admin-token auth and the token-scan IP hash | `openssl rand -base64 48` |
| `GROK_WEBHOOK_ENCRYPTION_KEY` | Comp | O | P | Yes | Encrypts members' saved Grok Bot webhook URLs. If unset, derived from `COMPANION_API_SECRET`. | `openssl rand -base64 32` |
| `GROK_WEBHOOK_HOSTS`, `GROK_INBOUND_MAX_MINUTES` | Comp | O | P | No | Allowed webhook hosts / inbound window | hostnames / number |

### Companion ↔ Moe link

| Name | App | Req | Env | Sens | What it does | How to get it |
|---|---|---|---|---|---|---|
| `COMPANION_API_SECRET` | Moe + Comp | **R** | P | Yes | Server-to-server secret between the apps; **identical in both** | `openssl rand -base64 48` |
| `COMPANION_API_URL` | Moe | R | P | No | Companion base URL as seen from Moe | `https://<companion domain>` |
| `NEXT_PUBLIC_COMPANION_URL` | Moe | O | P | No | Companion URL for browser links | same |
| `COMPANION_USER_TOKEN_PRIVATE_KEY` | Moe | **R** | P | Yes | Signs per-member ES256 tokens Companion trusts | P-256 pair (§2) |
| `COMPANION_USER_TOKEN_PUBLIC_KEY` | Comp | **R** | P | Recommended | Verifies those tokens | matching public PEM |
| `BOBINA_COUNCIL_URL`, `BOBINA_MOE_URL`, `BOBINA_MOE_API_URL` | Comp (+Moe) | R | P | No | Moe base URLs as seen from Companion | `https://bobina.moe` |
| `ACHIEVEMENT_ENDPOINT`, `COMPANION_API_BASE_URL` | Comp | O | P | No | Overrides for Moe endpoints | URL |

### Site URLs and misc

| Name | App | Req | Env | Sens | What it does |
|---|---|---|---|---|---|
| `NEXT_PUBLIC_SITE_URL` | Moe | R | P, Pr, D | No | Canonical public URL used in links and metadata |
| `NEXT_PUBLIC_APP_URL`, `NEXT_PUBLIC_URL`, `NEXT_PUBLIC_BASE_URL` | Moe / Comp | O | P | No | Legacy URL aliases |
| `NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION`, `NEXT_PUBLIC_GROK_TEMPLATE_URL` | Moe | O | P | No | Search Console tag / Grok template link |
| `VERCEL_URL`, `VERCEL_ENV`, `VERCEL_PROJECT_PRODUCTION_URL`, `NEXT_PUBLIC_VERCEL_URL`, `VERCEL_OIDC_TOKEN`, `NODE_ENV` | both | auto | — | — | Set by Vercel/Next.js. Don't add them yourself ([system env vars](https://vercel.com/docs/environment-variables/system-environment-variables)) |

Test-only variables (`TEST_*`, `TSX_BIN`, `OUT_DIR`, `ROOT`, `DBG`, `KVDBG`, `BOBINA_CHAT_DEBUG`, …) are read by scripts and the test harness, never by the deployed app. Don't set them in Vercel.

---

## 4. Portal walkthroughs

### Discord Developer Portal
[discord.com/developers/applications](https://discord.com/developers/applications) → select (or **New Application**) Bobina.
- **General Information:** copy **Application ID** → `DISCORD_BOT_ID` (Moe `DISCORD_CLIENT_ID` uses the same app id) and **Public Key** → `DISCORD_PUBLIC_KEY`. Set **Interactions Endpoint URL** to `https://<companion domain>/api/discord`. Discord pings it and rejects the save if the public key doesn't verify, so deploy the key first.
- **Bot:** **Reset Token** → `DISCORD_BOT_TOKEN` (shown once). Turn on the privileged intents the bot uses (Server Members, Message Content).
- **OAuth2:** copy **Client Secret** → Moe `DISCORD_CLIENT_SECRET`. Add redirect `https://bobina.moe/api/auth/callback/discord`.
- Guild/channel/role ids: Discord **User Settings → Advanced → Developer Mode**, then right-click → **Copy ID**.

### Telegram @BotFather
Open [@BotFather](https://t.me/BotFather) ([docs](https://core.telegram.org/bots/features#botfather)): `/newbot` (or `/token` for an existing bot) → `TELEGRAM_BOT_TOKEN`. `/setdomain` → `bobina.moe` for the login widget.

Register the webhook with the secret header (run locally, don't paste the output anywhere):

```bash
curl -s "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/setWebhook" \
  -d "url=https://<companion domain>/api/telegram" \
  -d "secret_token=$TELEGRAM_WEBHOOK_SECRET" \
  -d 'allowed_updates=["message","callback_query"]'
curl -s "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/getWebhookInfo"
```

Telegram then sends `X-Telegram-Bot-Api-Secret-Token` on every update ([setWebhook](https://core.telegram.org/bots/api#setwebhook)). Re-run `setWebhook` whenever you change the secret.

### X Developer Portal
[developer.x.com/en/portal/dashboard](https://developer.x.com/en/portal/dashboard) → Project → App.
1. **User authentication settings:** App permissions **Read and write**, type **Web App**, callback `https://bobina.moe/api/auth/callback/twitter`.
2. **Keys and tokens:** API Key and Secret → `TWITTER_API_KEY` / `TWITTER_API_SECRET`. Bearer Token → `TWITTER_BEARER_TOKEN`. Access Token and Secret (generate **after** step 1, or they stay read-only) → `TWITTER_ACCESS_TOKEN` and `TWITTER_ACCESS_TOKEN_SECRET` (Moe) / `TWITTER_ACCESS_SECRET` (Comp). OAuth 2.0 Client ID and Secret → `TWITTER_CLIENT_ID` / `TWITTER_CLIENT_SECRET` (Moe sign-in).

### Upstash
[console.upstash.com](https://console.upstash.com/) → Redis → your database → **REST API** section. Copy `UPSTASH_REDIS_REST_URL` and the **read-write** `UPSTASH_REDIS_REST_TOKEN` into `KV_REST_API_URL` / `KV_REST_API_TOKEN`. The **read-only** token can't write and will break credits, rate limits and sessions, so don't use it for these. If the database was added through the Vercel Marketplace, Vercel injects prefixed copies (e.g. `UPSTASH_FOR_REDIS_*`). The code doesn't read those names, so map them to `KV_*`.

### Supabase
[supabase.com/dashboard](https://supabase.com/dashboard) → project → **Project Settings → API Keys / Data API**: Project URL → `SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_URL`. `service_role` / secret key → `SUPABASE_SERVICE_ROLE_KEY` (server only, Sensitive). The anon / publishable key is only needed for Companion's `SUPABASE_ANON_KEY`. Connection strings: the **Connect** button.

### Vercel AI Gateway
On Vercel nothing is needed: the Gateway authenticates with the project's OIDC token automatically ([authentication](https://vercel.com/docs/ai-gateway/authentication)). For local dev or self-hosting, create a key under **AI Gateway → API Keys** in the team dashboard and set `AI_GATEWAY_API_KEY`, or run `vercel env pull` to get a short-lived OIDC token. Model ids are `creator/model` from the [model list](https://vercel.com/ai-gateway/models). `BOBINA_AI_MODEL` and `BOBINA_FAST_MODEL` have **no defaults**: unset means every turn fails closed with `model_unavailable`, and the member is refunded.

### Vercel Blob
Vercel → **Storage → Create → Blob** → **Connect Project** adds `BLOB_READ_WRITE_TOKEN` ([docs](https://vercel.com/docs/vercel-blob)). For Companion's Grok media, create a **second, private** store, connect it with a custom prefix or copy its token, and save it as `GROK_MEDIA_BLOB_READ_WRITE_TOKEN`.

### Coinbase Developer Platform (onramp)
[portal.cdp.coinbase.com](https://portal.cdp.coinbase.com/) → project → **API Keys → Create API key** (Secret API Key). Save the key name/id → `COINBASE_API_KEY_NAME` and the private key → `COINBASE_API_KEY_SECRET`. Enable **Onramp** for the project and add `bobina.moe` to the allowed domains ([Onramp docs](https://docs.cdp.coinbase.com/onramp/docs/welcome)).

---

## 5. Rotation: what each rotation breaks

| Variable | Effect of rotating | Safe procedure |
|---|---|---|
| `IP_HASH_KEY` | **Breaks every IP ban** and all IP matching: stored hashes can no longer be matched. | **Don't rotate.** If it leaks, rotate and accept that IP bans must be re-applied. |
| `BOBINA_HASH_SECRET` (Comp) | `/terms` prompt rate-limit markers reset (they expire within minutes anyway). Old terms IP hashes stop matching anything, but acceptances stay valid because they're keyed by account. | Replace and redeploy. |
| `BOBINA_HASH_SECRET` (Moe) | Same terms effect. Proposal/submission hashes made before can't be re-derived. Whatever calls `/api/companion/lookup` must get the new value at the same time. | Replace and redeploy together with the caller. |
| `MERCH_PII_ENCRYPTION_KEY` | Every stored shipping address becomes **unreadable**. | Re-encrypt first (decrypt with the old key, encrypt with the new key, e.g. adapting `scripts/merch-encrypt-legacy-addresses.ts`), then swap. |
| `COMPANION_USER_TOKEN_PRIVATE_KEY` / `_PUBLIC_KEY` | Every outstanding member token is rejected. Members get logged out of Companion tokens and must re-auth. | Generate a new pair, set **both** apps, deploy both together. |
| `OAUTH_JWT_PRIVATE_KEY` / `_PUBLIC_KEY` | ID tokens issued to third-party apps stop verifying until they refetch keys. | Rotate in a quiet window. |
| `COMPANION_API_SECRET` | Moe↔Companion calls fail until both match. Saved Grok webhooks become unreadable if `GROK_WEBHOOK_ENCRYPTION_KEY` isn't set, and members must re-save them. | Set both apps, deploy both. |
| `MCP_TOKEN_SECRET` | Signed MCP tokens in flight are rejected (they're short-lived). | Set both apps, deploy both. |
| `GROK_WEBHOOK_ENCRYPTION_KEY` | Saved Grok webhook URLs become unreadable, and members re-save them. | Replace and redeploy. |
| `NEXTAUTH_SECRET` | Everyone is signed out of bobina.moe. | Replace and redeploy. |
| `CRON_SECRET` | Nothing persistent. Vercel uses the new value on the next deployment. | Replace and redeploy. |
| `TELEGRAM_WEBHOOK_SECRET` | Telegram updates are rejected until you re-run `setWebhook`. | Set env, redeploy, re-run `setWebhook`. |
| Bot tokens / third-party keys | The old one stops working once revoked in the portal. | Reset in the portal, then update Vercel and redeploy. |

---

## 6. Known gaps (from the last audit)

- `TWITTER_ACCESS_SECRET` (Companion code) vs `TWITTER_ACCESS_TOKEN_SECRET` (set in Companion's Vercel): the names differ, so Companion's X posting reports "not configured". Add `TWITTER_ACCESS_SECRET` to Companion.
- `BOBINA_HASH_SECRET` falls back to the literal `"default_secret"` for proposal/submission hashes on Moe if unset. Always set it.
- Set in Vercel but not read by code: Moe `BOBINA_WEBHOOK_SECRET`, `NEXT_PUBLIC_DYNAMIC_ENVIRONMENT_ID`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_ANON_KEY`, `SUPABASE_JWT_SECRET`, `POSTGRES_*` (integration-injected). Comp `BOBINA_VISION_MODEL`, `DISCORD_MEMBERS_WEBHOOK_URL`, `TWITTER_CLIENT_ID`, `TWITTER_CLIENT_SECRET`, `UPSTASH_FOR_REDIS_*`. `COUNCIL_MOD_USERNAMES` goes away with Moe #875.
