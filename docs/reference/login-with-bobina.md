# Login with Bobina.moe

<p align="center">
  <a href="https://bobina.moe/bobinas/319"><img src="https://6wf3xhuhwdy0ogdt.public.blob.vercel-storage.com/bobinas/319.png" alt="Chaos Bobina" width="130" /></a>
  <a href="https://bobina.moe/bobinas/318"><img src="https://6wf3xhuhwdy0ogdt.public.blob.vercel-storage.com/bobinas/318.png" alt="Black Hole Bobina" width="130" /></a>
</p>

Let your users sign in with their Bobina.moe account and share scoped, verified data with your site.

## Overview

"Login with Bobina.moe" is an OpenID Connect (OIDC) identity provider built on the standard OAuth 2.0 Authorization Code flow with PKCE. Partner sites can authenticate Bobina Council members and read a minimal, user-consented set of profile and on-chain data — without ever handling passwords or seeing data the user has not explicitly approved.

## What we expect from partners

- Request only the scopes your integration genuinely needs (principle of least privilege).
- Store the client secret server-side only. Never expose it in browser or mobile code.
- Use exact, HTTPS redirect URIs — no wildcards, no localhost in production.
- Clearly disclose to your users what Bobina data you collect and why.
- Never resell, share, or retain Bobina user data beyond your stated purpose.

OAuth client provisioning is granted at the Bobina Council's sole discretion. We may decline, suspend, or revoke any application — including for token partnerships and auto-verification — at any time. Approval is manual and credentials are issued by Council staff; there is no public self-service registration.

## Prerequisites

- An approved OAuth application (contact Council staff to request provisioning).
- A `client_id` and `client_secret` issued by the admin panel (the secret is shown only once).
- One or more pre-registered, exact-match HTTPS redirect URIs.
- A PKCE-capable client (S256 code challenge is mandatory).
- The ability to verify ID tokens against our published JWKS (ES256 / EC P-256).

Discovery document: `https://bobina.moe/.well-known/openid-configuration`

## Available scopes

Each scope is independently allow-listed per application by staff and consented to by the user. A claim is released only if both conditions are met.

| Scope | Grants access to | Claims returned |
| --- | --- | --- |
| `openid` (required) | Your Bobina Council ID (BC_ID) | `sub` |
| `profile` | Username, display name, profile picture | `username, name, picture` |
| `email` | Email address, if one is linked | `email, email_verified` |
| `wallet` | Verified on-chain wallet address | `wallet_address` |
| `holdings` | Verified BOBINA balance across all supported chains | `bobina_total, bobina_mainnet, bobina_base, bobina_ink` |

## Supported chains for holdings verification

The `holdings` scope verifies BOBINA balances on-chain. The `bobina_total` claim is the summed balance across all EVM chains below.

| Chain | Chain ID | BOBINA contract | Verification |
| --- | --- | --- | --- |
| Ethereum | `1` | `0xAF3a37E8…D5Cb52` | On-chain |
| Base | `8453` | `0x289E4958…614786` | On-chain |
| Ink | `57073` | `0x36B9115F…C4b1B9` | On-chain |
| Solana | `—` | `h85j7prr…r5hsz` | Bridged (NTT) — not summed |

BOBINA bridges natively between Ethereum and Ink via Wormhole NTT. Holdings are cached for up to one hour and re-verified on demand.

## Privacy by design

We deliberately expose the minimum needed for authentication and token-gating. We never share private messages, companion data, internal identifiers beyond your BC_ID, IP addresses, or any data outside the scopes a user has explicitly consented to. Users can review and revoke an application's access at any time.

## Setting up (e.g. Pickle Charts)

1. Request an OAuth application from Council staff, providing your app name and exact redirect URI(s).
2. Receive your `client_id` and `client_secret` (store the secret securely).
3. Redirect users to `/api/oauth/authorize` with your scopes, `state`, and PKCE `code_challenge`.
4. Exchange the returned code at `/api/oauth/token` for an ID token and access token.
5. Call `/api/oauth/userinfo` with the access token to read consented claims (e.g. wallet + holdings for auto-verification).

**Terms of Service:** the consent screen is hosted on bobina.moe and requires the user to have accepted the Terms of Service. A user who hasn't accepted yet sees the Terms dialog first. Server actions that require acceptance return `terms_required`. This includes the Bobina Companion's own OAuth consent.

For token-partnership auto-verification, request the `wallet` and `holdings` scopes — the `bobina_total` claim lets you gate features by BOBINA balance across Ethereum, Base, and Ink in a single call.

## OAuth vs MCP

"Login with Bobina.moe" answers *who a user is* (identity + consented profile/holdings). It does not let an app *do things* as the Companion. Programmatic access to Bobina's tools — chat, market opinions, charts, image generation, and more — is exposed separately through the [Model Context Protocol (MCP)](#mcp), where each tool has a published credit price. Use OAuth to sign users in; use MCP to call capabilities on their behalf.


---

<p align="center">
  <a href="https://bobina.moe/bobinas/318"><img src="https://6wf3xhuhwdy0ogdt.public.blob.vercel-storage.com/bobinas/318.png" alt="Black Hole Bobina" width="100" /></a>
</p>

<p align="center"><sub>Art from the <a href="https://bobina.moe/bobinas">Bobina gallery</a> · Back to the <a href="../../README.md">Docs index</a></sub></p>

