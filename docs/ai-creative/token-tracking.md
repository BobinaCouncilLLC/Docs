# Token Tracking

<p align="center">
  <a href="https://bobina.moe/bobinas/313"><img src="../../assets/bobinas/313-bamboo-bobina.webp" alt="Bamboo Bobina" width="130" /></a>
  <a href="https://bobina.moe/bobinas/312"><img src="../../assets/bobinas/312-fathers-day-bobina.webp" alt="Father’s Day Bobina" width="130" /></a>
</p>

Community-driven token call leaderboards with anti-manipulation safeguards.

## Overview

The Token Tracking system surfaces the best token calls from community members across Discord and Telegram. Calls are tracked by entry price, current performance, and community engagement.

## Leaderboard Views

### Top Tokens

Ranked by performance multiplier (current price / entry price). Shows the best performing calls.

### Rising

Trending tokens based on combined scan activity and trading volume. Manipulation-resistant scoring.

### Top Callers

Community members ranked by average multiplier, win rate, and total calls made.

## Rising Score Formula

The Rising leaderboard uses a combined score that rewards genuine interest backed by real trading activity:

combinedScore = sqrt(scanScore x volumeScore) + liquidityBonus

scanScore = weighted scans (recent activity counts more)

volumeScore = log10(1 + volume24h) x (1 + volumeRatio x 10)

liquidityBonus = bonus for tokens with > $10k liquidity

## Anti-Manipulation Safeguards

| Attack Vector | Defense |
| --- | --- |
| Scan farming with bots | One scan per user/IP per hour + diminishing returns without volume |
| Wash trading | Volume alone not enough; need real user scans to rank high |
| Sybil attacks | One scan per hour per identity limits bot effectiveness |
| Low liquidity rugs | No liquidity bonus below $10k, reducing fake token incentive |

## Community Voting

Signed-in members can upvote or downvote token calls to help surface quality content. Only verified Council IDs (bc_) can vote. Votes update instantly with optimistic UI - see your vote reflected immediately while the server syncs in the background.

Upvote good calls

Downvote suspicious activity

Reputation Score = Upvotes - Downvotes

## Warning Flags & Bad Actor Detection

The system automatically analyzes tokens and callers for suspicious patterns. Warning badges appear on token cards to help you identify potential risks before investing.

### Rug Pull Detection

Identifies tokens where liquidity was removed or ownership renounced suspiciously

### Honeypot Warning

Flags tokens with sell restrictions or suspicious smart contract code

### Repeat Rugger

Caller has history of promoting tokens that later rugged

### Low Liquidity

Token has insufficient liquidity for safe trading

Warning severity levels: Critical (immediate danger), High (significant risk), Medium (caution advised). Always DYOR (Do Your Own Research) before trading.

## Key Metrics

- **Multiplier:** Current price / entry price (e.g., 2.5x means 150% gain)
- **ATH Multiplier:** Highest multiplier reached since call
- **Win Rate:** Percentage of calls that reached > 1.0x
- **Scan Count:** Number of unique users who scanned the token
- **Volume 24h:** 24-hour trading volume in USD
- **Liquidity:** Pool liquidity in USD


---

<p align="center">
  <a href="https://bobina.moe/bobinas/312"><img src="../../assets/bobinas/312-fathers-day-bobina.webp" alt="Father’s Day Bobina" width="100" /></a>
</p>

<p align="center"><sub>Art from the <a href="https://bobina.moe/bobinas">Bobina gallery</a> · Back to the <a href="../../README.md">Docs index</a></sub></p>

