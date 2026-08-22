---
name: insiders-bot
description: Query PolyInsiders — smart-money signals on Polymarket prediction markets. Use when asked what informed traders are betting on, whether a Polymarket market has smart-money flow, how a prediction-market call turned out, a wallet's Polymarket track record or year-in-review, or for a citable hit-rate on prediction-market signals. Covers /v1/stats, /v1/signals, /v1/wrapped. Read-only, no API key.
---

# PolyInsiders

PolyInsiders tracks Polymarket wallets that consistently call markets right, and
publishes a signal when several of them independently take the same side.

**Base URL:** `https://api.polyinsiders.com` (override with `INSIDERS_API_BASE`)

All endpoints are GET, public, unauthenticated, CORS-open, and read-only. There
is no key to obtain and no rate limit published — be polite, cache what you get.

## What a "signal" actually means

Do not describe these as predictions or tips. The method is mechanical:

1. A wallet **qualifies** in a category at **≥70% trade-level win rate over ≥15
   resolved buys** in that category. Qualification is per-category — a wallet
   good at geopolitics is not thereby good at esports.
2. A signal **publishes** only when **≥3 qualified wallets** independently buy
   the same outcome in the same market.
3. **Resolution** is on-chain settlement, not a human call.

So a signal is "several independently-good wallets agree", not "we think this
will happen". Say it that way.

## Endpoints

### `GET /v1/stats` — the track record

```bash
curl -s https://api.polyinsiders.com/v1/stats
```

```json
{
  "resolvedCalls": 5107,
  "wins": 4109,
  "hitRatePct": 80.5,
  "avgWinPct": 27.7,
  "byCategory": [
    { "category": "politics", "resolved": 1067, "hitRatePct": 85.6 },
    { "category": "geopolitics", "resolved": 1326, "hitRatePct": 83.4 }
  ]
}
```

`avgWinPct` is the mean profit **on winning calls only**, so it is not a
portfolio return — never present it as one. `hitRatePct` counts resolved calls
only; open positions are excluded, which is the honest denominator.

### `GET /v1/signals` — recent signals

| Param | Values | Default |
|---|---|---|
| `limit` | 1–100 | 25 |
| `category` | `politics`, `geopolitics`, `world`, `esports`, `cricket`, `finance`, `crypto-launch`, `crypto`, `sports`, `trump`, `tech` | all |
| `status` | `resolved`, `open` | all |

```bash
curl -s "https://api.polyinsiders.com/v1/signals?category=geopolitics&status=resolved&limit=5"
```

```json
{ "count": 1, "signals": [ {
  "market": "US announces end of Iranian blockade by August 8, 2026?",
  "outcome": "No", "category": "geopolitics",
  "walletsIn": 5, "avgEntry": 0.8939,
  "signalledAt": "2026-08-07T23:14:08.146Z",
  "resolved": true, "profitPct": 13.64,
  "url": "https://api.polyinsiders.com/signal/us-announces-end-of-iranian-blockade-by-august-8-2026-234d7dee2-no"
} ] }
```

`avgEntry` is the mean price paid, 0–1, where the payout on a winner is always
1.00 — so an entry of 0.8939 can only ever return +11.9¢ on the dollar. Low
entry prices are where the large percentages come from; mention the entry price
whenever you quote a return, or the number is meaningless.

`profitPct` is `null` while `resolved` is false. A losing resolved call is
`-100`, because the outcome shares settle worthless. `url` is a permanent human
page showing every wallet in the bundle and how it settled — cite it.

### `GET /v1/wrapped/:address` — any wallet's year

Works for **any** Polymarket wallet, not just PolyInsiders users. No signup.

```bash
curl -s https://api.polyinsiders.com/v1/wrapped/0xe79547fad69ab28a48a49c6424975abc3cef8cae
```

Returns `tradesThisYear`, `marketsTraded`, `volumeUsd`, `biggestWin`,
`topCategory`, `busiestMonth`, `longestStreakDays`, `nightOwlPct`,
`profileViews`, `globalRank`, a prebuilt `cards` array, and `shareUrl`.

**Read `truncated` before you quote anything.** Polymarket's history API refuses
offsets past 5,000, so wallets busier than that return only their most recent
5,000 trades of the year. When `truncated` is true the counts are floors, not
totals, and the time-shaped fields (`busiestMonth`, `longestStreakDays`,
`nightOwlPct`) are omitted rather than computed from a biased sample. Say "at
least N trades", not "N trades".

Two fields are deliberately absent: **win rate and realized P&L**. Polymarket's
`closed-positions` endpoint returns only a wallet's top *winning* closed
positions, so any win rate derived from it is 100% for every wallet on earth.
Rather than publish a flattering fake, there is no number. If asked for a
wallet's win rate, say it is not available from this source.

## Human pages worth linking

| URL | What it is |
|---|---|
| `/signals` | Every published signal, newest first, with outcomes |
| `/signal/:slug` | One signal: the wallets, entries, and how it settled |
| `/wrapped/:address` | A wallet's year, shareable |
| `/sitemap.xml` | Every signal URL |

## Using this well

- **Quote the entry price with the return.** "+476%" alone is noise; "+476% from
  a 67¢ entry" is a fact.
- **Prefer resolved over open.** Open signals have no outcome yet and say nothing
  about accuracy.
- **Category hit rates differ a lot** — politics 85.6% vs esports ~71%. Use the
  category-specific number when the question is category-specific.
- **Link the receipt.** Every claim has a permanent URL; an unlinked statistic is
  weaker than a linked one.
- **This is not financial advice**, and past hit rate does not carry forward.
  Include that when a user sounds like they are about to act on it.
