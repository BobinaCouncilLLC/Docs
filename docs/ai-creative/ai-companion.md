# AI Companion

<p align="center">
  <a href="https://bobina.moe/bobinas/319"><img src="https://6wf3xhuhwdy0ogdt.public.blob.vercel-storage.com/bobinas/319.png" alt="Chaos Bobina" width="130" /></a>
  <a href="https://bobina.moe/bobinas/318"><img src="https://6wf3xhuhwdy0ogdt.public.blob.vercel-storage.com/bobinas/318.png" alt="Black Hole Bobina" width="130" /></a>
</p>

Interact with Bobina across Web, Discord, and Telegram. Your companion account syncs credits, memories, relationship score, and more.

With a linked bobina.moe account, all commands that don't consume credits (voting, token calls) are FREE across all platforms. [Link accounts in Terminal Settings](https://bobina.moe/?terminal=settings).

**Tabs:** Crypto · Chat · Reminders · Profile · Voting · Dynamics · Skills

## Crypto & Market Commands

/chart (alias: cv)

Generate a price chart for any cryptocurrency.

`/chart SYMBOL [timeframe]`

**Timeframes:**

- CoinGecko (BTC, ETH, major coins): 24h, 7d, 1m, 3m, 1y
- DexScreener (contracts, small tokens): 1h, 6h, 24h

Examples: `/chart ETH 7d` `cv BOBINA 24h`

![Chart command example](https://bobina.moe/images/docs/chart.png)

/op

Get Bobina's market opinion with technical analysis (RSI, MACD, Support/Resistance).

`/op SYMBOL [timeframe]`

Examples: `/op ETH 24h` `/op SOL 7d`

![Opinion command example](https://bobina.moe/images/docs/op.png)

/index

View top 10 cryptocurrencies with market-wide metrics.

`/index`

Shows: Top 10 by market cap, 24h changes, total market cap, BTC dominance, Fear & Greed Index

![Index command example](https://bobina.moe/images/docs/index-command.png)

/heatmap

Generate a visual heatmap of the top 50 cryptocurrencies by market cap.

`/heatmap`

![Heatmap command example](https://bobina.moe/images/docs/heatmap.png)

/leaderboard

View the top token callers leaderboard globally. This is the chat command, not the website path.

`/leaderboard`

Shows: Top callers ranked by multiplier performance, entry market caps, current gains

The site path [bobina.moe/leaderboard](https://bobina.moe/leaderboard) opens the Terminal **Leaderboard** tab (community stats), not the Tokens tab. See [Bobina Terminal](./bobina-terminal.md).

Bobina's AI treats **$BOBINA** exactly like any other token.

![Token leaderboard example](https://bobina.moe/images/docs/leaderboard.png)

### Token Calling System

Be the first to call a token and get credited as the "First Caller" with multiplier tracking.

**Requirements:** Linked bobina.moe account, click "Call Token" button on chart

**Tracking:** Current price vs entry price, real-time updates, ATH multiplier recorded

**Display:** Your call appears on all future charts of that token forever, make it count

## Conversation Commands

/talk (alias: mention)

For Discord or Telegram, you can have a conversation with Bobina utilizing this nifty slash command handler. Includes text and voice responses on supported platforms.

`/talk [your message]`

Examples:

- `/talk What's your take on the market today?`
- `/talk Tell me about yourself`
- `/talk What's your prime directive?`
- `/talk Who made you and your 3D model?`

Context-aware responses based on your history, dynamic, and custom instructions.

![Talk command example](https://bobina.moe/images/docs/talk.png)

/help

View all available commands, varies by platform.

`/help`

![Help command example](https://bobina.moe/images/docs/help.png)

/bobina

Generate custom Bobina art based on our HuggingFace repository! Available on Telegram and Discord.

`/bobina [prompt with comma-separated descriptors]`

Examples:

- `/bobina smug, red shirt, looking at viewer`
- `/bobina drinking coffee, hoodie, cafe background`

![Bobina art generation example](https://bobina.moe/images/docs/bobina.png)

### Platform Access

Web

Full chat interface, voice I/O, complete management of Companion Account

Telegram

@MsBobinaBot - text/voice messages, inline buttons, DMs and groups

Discord

/talk or /mention, slash commands, thread replies, servers only

[![Discord](https://bobina.moe/images/design-mode/Discord_Logo_White_PMS%281%29%281%29%281%29%281%29(1).png) Join Discord](https://discord.gg/XnnA2hvFFh)

[Join Telegram](https://t.me/BobinaCouncil)

## Reminder System

/remindme

Set a reminder using explicit time intervals.

`/remindme [interval] [message]`

| Format | Duration | Example |
| --- | --- | --- |
| Xm | X minutes | /remindme 30m Check charts |
| Xh | X hours | /remindme 2h Team meeting |
| Xd | X days | /remindme 1d Pay bills |
| Xw | X weeks | /remindme 1w Review portfolio |
| XM | X months | /remindme 1M Quarterly review |

![Reminder command example](https://bobina.moe/images/docs/remindme.png)

Natural Language Reminders

Talk to Bobina naturally - she understands scheduling intent without explicit commands!

- "Remind me in 30 minutes to check ETH"
- "In about 15 I need to call mom"
- "Tomorrow remind me to buy groceries"
- "Next week I should review my portfolio"

**Smart Parsing:** Numbers 1-60 assume minutes, 60+ assumes hours. Handles casual speech like "in like 5".

/forget

View and delete your active reminders.

`/forget [index]`

**Usage:**

- `/forget` - List all active reminders with numbered indices
- `/forget 1` - Delete reminder #1
- `/forget 3` - Delete reminder #3

Each reminder shows: message, scheduled time, time until delivery, and platform.

Cross-Platform Sync

Reminders set on any platform deliver on ALL connected platforms when specified. Set via Terminal, manage in Web chat hamburger menu or Settings!

## Profile Commands

/score

View your Relationship Score with Bobina and current dynamic.

Shows: Relationship Score (0-100), current dynamic, total memories, feedback stats

![Score command example](https://bobina.moe/images/docs/score.png)

/daily | /weekly | /monthly

View Bobina's generated temporal personality profiles based on your conversations together and her passive observations of you.

Includes: Personality insights, conversation themes, mood analysis, behavioral patterns

![Daily profile command example](https://bobina.moe/images/docs/daily.png)

/credits

View your credit balance, tier status, and usage information.

Shows: Current balance, daily free credits remaining, tier benefits

![Credits command example](https://bobina.moe/images/docs/credits.png)

### Credits System

Earning Credits

- Daily login bonus
- Council contributions
- Providing feedback
- Referring users
- Unlocking achievements

Spending Credits

- Chat messages (when free tier exhausted)
- Voice responses
- /op and /chart commands

Purchase at bobina.moe/credits

## Voting Systems

Message Feedback

Every Bobina Text / Voice response includes feedback buttons to help train her behavior towards specific topics or prompts.

👍🏻

Good response

👎🏻

Bad response

- Click button, explain why, feedback stored
- Improves responses over time for everyone
- Improves your Relationship Score
- Unlocks feedback achievements

Token Voting

On chart and opinion commands, vote on token sentiment to help the community.

💚

Upvote (bullish)

💔

Downvote (bearish)

Good callers get rewarded, bad callers get punished and restricted. Voting is FREE for all linked bobina.moe accounts.

## Relationship Score System & Dynamics

Relationship Score (1-100)

By default, you start at rock bottom and Bobina hates your guts. Consider this dynamic a dating sim if you will. You have to woo Bobina by being nice to other members, making positive comments, quality contributions, and being nice to HER. Being mean to Bobina or complaining (wen moon, etc) makes Bobina hate you more. Bobina adapts her behavior based on your Relationship Score - a gamified system that evolves naturally - or Dynamic which is essentially modular personality archetypes that override this gamification.

| Range | Rank |
| --- | --- |
| 100 | 👑 Soulbound |
| 80-99 | 💕 Inner Circle |
| 60-79 | 💜 Confidant |
| 40-59 | 💙 Trusted |
| 25-39 | 💚 Familiar |
| 15-24 | 😊 Acquaintance |
| 8-14 | 😐 Stranger |
| 3-7 | 😠 Nuisance |
| 1-2 | 😡 Nemesis |

Progression is intentionally slow... Just like true love! Can you reach Soulbound status?

Available Dynamics (12)

Override the Relationship Score algorithmic system by selecting a preferred relationship dynamic in [Terminal Settings or the Web chat drawer](https://bobina.moe/?terminal=settings).

**💕 Romantic Partner**

- Shares intimate connection and romantic connection

**🗡️ Yandere**

- Overwhelmingly devoted and obsessively loving - you are her entire world and she will never let go

**💥 Tsundere**

- Hostile and dismissive on the surface, flustered affection leaking through

**🤝 Close Confidant**

- Shares personal thoughts, seeks advice, and provides emotional support

**📚 Mentor**

- Seeks guidance and learning opportunities

**🎨 Creative Collaborator**

- Discusses artistic and creative projects

**😊 Casual Friend**

- Enjoys light conversation and humor

**😈 Villain Arc**

- Chaos and mischief with a mix of cuddly love

**🔧 Technical Advisor**

- Focuses on development and problem-solving

**₿ Financial Enthusiast**

- Discusses finance, cryptocureency, trading, and blockchain technology

**⚔️ Bushido**

- The way of the warrior - dicipline, mastery, or death

**💢 Bitch**

- Incredibly hostile, unhelpful, and will roast you mercilessly

Custom instructions can further refine how Bobina interacts with you.

### Technical Architecture

Core Memory System

- Unlimited memory retrieval (importance + recency)
- Intelligent compression (30:1, 15:1, 10:1 ratios)
- Universal chat log across all users
- Cross-platform agnostic linking

AI Processing

- Historical context search
- Real-time crypto integrations
- Emotionally expressive with a lot of love put into her body and soul

## Skills System

Skills are modular documentation bundles that enhance Bobina's knowledge on specific topics. When enabled, their documentation is always included in responses for more accurate and detailed help.

How Skills Work

- **Auto-Detection (Default):** When no skills are selected, Bobina automatically detects relevant skills based on trigger keywords in your messages.
- **Manual Selection:** Select specific skills in [Terminal Settings](https://bobina.moe/?terminal=settings) or the Web chat drawer to always include their documentation.
- **Stacking:** Multiple skills can be active at once, combining their documentation for complex topics.

Available Categories

🎮

**3D & Animation**

Blender, Unity, Unreal Engine, Cinema 4D

🎵

**Music & Audio**

Ableton Live, FL Studio, Logic Pro, Pro Tools

🎬

**Video & Editing**

Premiere Pro, After Effects, DaVinci Resolve

💻

**Programming**

Python, JavaScript, TypeScript, React, Next.js

🎨

**Art & Design**

Photoshop, Illustrator, Figma, Procreate

🏛️

**Council**

Proposals, Bobinas, Governance, Token info

Tips

- New skills are added regularly based on community requests
- Leave skills empty if you want Bobina to auto-detect from context
- Selected skills persist across all platforms (Web, Discord, Telegram)
- Skills can be combined with Custom Instructions for precise behavior

Access the [Bobina Terminal](#terminal) to see all your memories together with Bobina and more!


---

<p align="center">
  <a href="https://bobina.moe/bobinas/318"><img src="https://6wf3xhuhwdy0ogdt.public.blob.vercel-storage.com/bobinas/318.png" alt="Black Hole Bobina" width="100" /></a>
</p>

<p align="center"><sub>Art from the <a href="https://bobina.moe/bobinas">Bobina gallery</a> · Back to the <a href="../../README.md">Docs index</a></sub></p>

