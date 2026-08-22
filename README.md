# insiders-skill

Trade [Polymarket](https://polymarket.com) from your terminal — an agent skill for
[PolyInsiders](https://api.polyinsiders.com).

Install it into any agent that reads `SKILL.md` and that agent can look up
markets, check balances, place and close trades, redeem winnings, copy traders,
and pull the smart-money signal feed — without you leaving the shell.

```
you  ▸ what's smart money buying in geopolitics?
you  ▸ show me the orderbook for that market
you  ▸ buy $25 of Yes at 0.42
you  ▸ what are my open positions?
```

## Install

**Claude Code**

```bash
git clone https://github.com/SigmaLabsAI/insiders-skills.git ~/.claude/skills/insiders-bot
```

**Anything else** — `SKILL.md` is self-contained. Load it as a system prompt
fragment, a tool description, or plain context.

## API

| | |
|---|---|
| Base URL | `https://api.polyinsiders.com` |
| Spec | [`/openapi.json`](https://api.polyinsiders.com/openapi.json) — 116 endpoints, OpenAPI 3 |
| Docs | [`/api-docs/`](https://api.polyinsiders.com/api-docs/) |

Market data, balances, signals and the track record are public — no key, no
signup. Wallets, orders, positions and redemption take a JWT from
`POST /api/auth/login/wallet`.

```bash
curl -s https://api.polyinsiders.com/v1/stats
curl -s "https://api.polyinsiders.com/v1/signals?category=politics&status=resolved&limit=5"
```

## What it covers

- **Markets** — look up a market or event, live orderbook quotes, resolve a
  Polymarket username to an address
- **Wallets** — balances for any address; the authenticated user's custodial
  wallet is provisioned automatically on first contact
- **Trading** — market and limit buys and sells, close a position, cancel orders
- **Portfolio** — open and closed positions, aggregate P&L, trade history
- **Settlement** — redeem winnings on resolved markets
- **Copytrading** — rank traders, subscribe, adjust, stop
- **Signals** — what qualified wallets are buying, the resolved track record, and
  any wallet's year in review

## Signals

A wallet qualifies in a category at ≥70% trade-level win rate over ≥15 resolved
buys in that category. A signal publishes only when ≥3 qualified wallets
independently buy the same outcome. Resolution is on-chain settlement.

Every signal has a permanent page showing the wallets behind it, their entry
prices, and how it settled.

## Safety

The skill instructs agents to confirm before anything that spends money — buys,
sells and copytrade subscriptions place real orders and are not reversible.

Not financial advice. Historical hit rate does not carry forward.

## License

MIT
