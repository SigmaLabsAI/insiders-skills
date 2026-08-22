---
name: insiders-bot
description: Trade Polymarket prediction markets from the terminal via PolyInsiders. Use when asked to look up a Polymarket market, check a wallet's positions or balances, place or close a trade, redeem winnings, copy a trader, see what smart money is buying, check the signal track record, or pull a wallet's year in review. Covers auth, wallets, trading, portfolio, redemption, copytrading, and the public signals API.
---

# PolyInsiders

Polymarket, from the command line. PolyInsiders wraps market discovery, wallet
custody, order execution, portfolio and redemption behind one REST API — and
layers on smart-money signals derived from wallets with a proven track record.

**Base URL:** `https://api.polyinsiders.com` (override with `INSIDERS_API_BASE`)
**Full spec:** `https://api.polyinsiders.com/openapi.json` — 116 endpoints, OpenAPI 3
**Browsable docs:** `https://api.polyinsiders.com/api-docs/`

Fetch the spec when you need a request/response shape this file does not cover.
Do not guess field names — the spec is the source of truth.

## Auth

Most read endpoints are public. Anything touching a wallet, an order, or a
position needs a JWT.

```bash
# Sign the login challenge with the wallet key, then exchange it for a token.
curl -s -X POST $INSIDERS_API_BASE/api/auth/login/wallet \
  -H 'content-type: application/json' \
  -d '{"walletAddress":"0x…","signature":"0x…"}'
# -> { "token": "eyJ…" }
```

Send it as `Authorization: Bearer <token>` on every authenticated call.
`POST /api/auth/verify` checks a token is still good; `GET /api/auth/wallet`
returns the wallet the token controls.

Users who arrive through the Telegram bot or "Sign in with X" already have a
custodial wallet provisioned for them — `GET /api/auth/wallet` returns it. There
is no separate create-wallet call to make.

## Reading the market

All public, no token.

| Call | Returns |
|---|---|
| `GET /api/market/market/{slug}` | One market — outcomes, prices, asset ids |
| `GET /api/market/event/{slug}` | An event and the markets under it |
| `GET /api/market/orderbook/{assetId}/quote` | Live quote for an asset |
| `GET /api/balances/{wallet}` | POL / USDC / USDC.e balances for any wallet |
| `GET /api/username/resolve/{username}` | Polymarket username → address |

`assetId` is the token id for one *outcome* of a market — it is what every trade
call takes, not the market slug or the condition id. Read it off the market
response before trading.

A quote returns `409 NO_LIQUIDITY` when nobody is making a market on that
outcome. That is an answer, not an error — report it and stop, rather than
retrying or placing a blind order.

## Trading

All authenticated. Amounts are USDC.

```bash
curl -s -X POST $INSIDERS_API_BASE/api/trade/buy \
  -H "authorization: Bearer $TOKEN" -H 'content-type: application/json' \
  -d '{"assetId":"7213…","amount":25,"price":0.42}'
```

| Call | Body | Does |
|---|---|---|
| `POST /api/trade/buy` | `assetId*`, `amount*`, `price` | Buy. Omit `price` for market |
| `POST /api/trade/sell` | `assetId*`, `amount*` | Sell a quantity |
| `POST /api/trade/sell-position` | see spec | Close a whole position |
| `GET /api/limit` | — | Open limit orders |
| `DELETE /api/limit/{orderId}` | — | Cancel one |
| `DELETE /api/limit/cancel-all` | — | Cancel all |

Prices are 0–1 and a winning share always settles at exactly 1.00. Buying at
0.42 risks 42¢ to make 58¢. State the entry price whenever you report a trade.

**Confirm before executing.** A buy or sell spends real money and is not
reversible. Show the user the market, side, amount and price, and get an explicit
yes — never infer one from an ambiguous instruction.

## Portfolio and settlement

| Call | Returns |
|---|---|
| `GET /api/portfolio/positions` | Open positions |
| `GET /api/portfolio/past-positions` | Closed |
| `GET /api/portfolio/stats` | Aggregate P&L |
| `GET /api/portfolio/history` | Trade history |
| `POST /api/redeem/condition` | Claim winnings on a settled market |
| `GET /api/redeem/condition-id/{assetId}` | Condition id for an asset |

Resolved winning positions do not pay out until redeemed.

## Copytrading

`GET /api/copytrade/top-traders` (public) ranks traders. `POST /api/copytrade/subscribe`
mirrors one into the user's wallet; `PUT`/`DELETE /api/copytrade/subscription/{id}`
adjust or stop it, `GET /api/copytrade/history` shows what it did.

A subscription places real orders automatically. Make the sizing explicit before
subscribing, and make sure the user knows how to stop it.

## Signals

A wallet **qualifies** in a category at ≥70% trade-level win rate over ≥15
resolved buys *in that category*. A signal **publishes** only when ≥3 qualified
wallets independently buy the same outcome. Resolution is on-chain settlement.

So a signal means "several independently-good wallets agree" — not a prediction.
Describe it that way.

```bash
curl -s "$INSIDERS_API_BASE/v1/signals?category=geopolitics&status=resolved&limit=5"
curl -s "$INSIDERS_API_BASE/v1/stats"
```

| Call | Returns |
|---|---|
| `GET /v1/stats` | Track record: resolved calls, hit rate, per-category |
| `GET /v1/signals` | Recent signals — `category`, `status` (`resolved`/`open`), `limit` (≤100) |
| `GET /v1/wrapped/:address` | Any Polymarket wallet's year in review |
| `GET /api/signals/v1` | Raw per-wallet signal feed — `minWinRate`, `hoursBack`, `sortBy` |

Every `/v1/signals` result carries a `url` to a permanent page showing the
wallets, their entries, and how it settled. Link it.

`profitPct` is `null` while `resolved` is false, and `-100` on a loss — outcome
shares settle worthless. `avgWinPct` in `/v1/stats` is the mean profit on
*winners only*, not a portfolio return.

On `/v1/wrapped`, check `truncated` before quoting figures: Polymarket's history
API refuses offsets past 5,000, so busy wallets return only recent trades and the
counts become floors. Win rate and realized P&L are absent by design — upstream
only exposes winning closed positions, so any win rate computed from it would be
100% for every wallet.

## Working well

- Quote returns with their entry price. "+476%" is noise; "+476% from a 67¢
  entry" is a fact.
- Prefer resolved signals over open ones when the question is about accuracy.
- Category hit rates differ — use the category-specific number.
- Confirm every state-changing call. Reads are free; trades are not.
- Not financial advice, and hit rate does not carry forward. Say so when someone
  sounds ready to act.
