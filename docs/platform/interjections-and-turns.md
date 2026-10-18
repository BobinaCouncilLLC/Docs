# Interjections & Turns

Part of the [Bobina Council Docs](../../README.md) · [Platform](./README.md). Mirrors the **Interjections & Turns** section on [bobina.moe/docs](https://bobina.moe/docs#interjections) (Bobina.moe #892).

How her acknowledgment, mid-turn messages and Council Sessions work. Prices, the first-ack promo, the per-turn cap and the on/off switches are **live** from the admin Credits Config: see the [Credits System](./credits-system.md#interjections) page and [bobina.moe/credits](https://bobina.moe/credits). Nothing here is a fixed price.

## Interjections and the acknowledgment

- The acknowledgment is just her first interjection: a quick line that replaces the thinking placeholder. When the free-first-ack promo is on, the first one is free.
- On the web it works for text, voice and convo. On Telegram and Discord it replaces the "Thinking…" message, and her reply posts below it.
- Anything else you send while she is working is an interjection (up to the per-turn cap, `maxInterjectionsPerTurn`).
- On bobina.moe/docs, an unset price shows "Price not set yet", and a switched-off part is marked "Acknowledgment off", "Mid-turn interjections off", or "Currently off" when both are off.

## One turn at a time

- Bobina runs one turn per member at a time.
- With Grok mode on, a text message sent mid-turn goes to Grok Bot, and Bobina answers once at the end with a Grok summary. With Grok mode off, it is folded into her reply and charged as an interjection.
- On Telegram and Discord, voice and /talk sent mid-turn are refused, with no charge. On the web, mid-turn voice is allowed and plays after the current audio.
- MCP gets no acknowledgment and no interjections.

## Council Sessions and commands

- Telegram, Discord and MCP turns record a live Council Session that only you (the owner) can see.
- /terms works on Discord.
