# MCP Capabilities

<p align="center">
  <a href="https://bobina.moe/bobinas/317"><img src="../../assets/bobinas/317-jacko-o-bobina.webp" alt="Jacko-o-bobina" width="130" /></a>
  <a href="https://bobina.moe/bobinas/316"><img src="../../assets/bobinas/316-grotto-bobina.webp" alt="Grotto Bobina" width="130" /></a>
</p>

How Bobina exposes her tools to AI clients through the Model Context Protocol.

## What is MCP?

The **Model Context Protocol (MCP)** is an open standard for connecting AI assistants to external tools and data. The Bobina Companion publishes a **capability registry** over MCP so compatible clients can discover and invoke Bobina's tools — the same capabilities available in chat (talk, market opinions, chart lookups, heatmaps, image generation, and more).

## How capabilities work

- **Namespaced:** every tool has a fully-qualified name (e.g. `bobina.chat.talk`) grouped by namespace so related tools stay organized.
- **Free or billable:** each tool is either free or costs a fixed number of credits per call. Prices are set by the Council and published in the registry — there are no hidden or variable fees.
- **Official or community:** tools are labeled so clients can distinguish first-party Council capabilities from community-contributed ones.
- **Fails closed:** a billable tool that has no published price is treated as disabled everywhere until the Council prices it. Nothing is ever charged at an unknown rate.

## Credits & billing

Billable MCP tools spend the same credits as the rest of the platform. Your [credit balance](#credits) is shared across web chat, Discord, Telegram, and MCP — a call made through any surface draws from one balance. Earned (free/daily) credits are spent before purchased credits, and balances update live: every read reflects the latest deduction, with no caching delay between spending on one surface and seeing the new total on another.

> **Info:** Current per-tool prices are always shown in the [Credits System](#credits) section and the in-app Credits panel. Because pricing is registry-driven, new tools appear there automatically once published.

## Interjections over MCP (spec)

- **No acknowledgement.** `bobina.talk` never shows or charges a turn-start acknowledgement. You get her one reply.
- **Mid-turn messages.** One turn runs per member at a time. A `bobina.talk` **text** call sent while her turn is running is an interjection: it is charged the `interjection` price (free daily slots first), up to N per turn (`maxInterjectionsPerTurn`), and folded into the running turn, or forwarded to your Grok Bot when Grok + Bobina is on. It does not start a separate Bobina reply. A **voice** call (`voice: true`) mid-turn is refused with `rate_limited` (429) and is not charged.
- **Refusals.** These are coded and never charged. Over the cap: `rate_limited`. Interjections switched off in the Credits Config: `interjections_disabled` (409), in her voice. Price or config unreadable: `billing_unresolved` (503). Text over 2,000 characters: `invalid_request` (400).
- **Credits.** If the forward or her turn fails after a charge, the charge is refunded once, each part (free slot or paid credits) back to where it came from. The `bobina.talk` tool description includes the same Interjections line as the credits pages.

## Privacy & access

MCP access is authenticated and tied to your Council ID, so tool calls draw from your own credits and respect your relationship and privacy settings. Capability access is granted at the Council's discretion, and the same data-minimization principles as [Login with Bobina.moe](#oauth) apply: tools only ever return what their described purpose requires.

## Grok Bot scopes

Grok Bot and grok.com connect with OAuth and can only be granted these scopes: `bobina.whoami`, `bobina.memory.recall`, `bobina.talk`, `bobina.grok.note`, `bobina.grok.reply`. See [Bobina for Grok Bot](./grok-bot.md) for what each one does and how to set it up.


---

<p align="center">
  <a href="https://bobina.moe/bobinas/316"><img src="../../assets/bobinas/316-grotto-bobina.webp" alt="Grotto Bobina" width="100" /></a>
</p>

<p align="center"><sub>Art from the <a href="https://bobina.moe/bobinas">Bobina gallery</a> · Back to the <a href="../../README.md">Docs index</a></sub></p>

