# insiders-bot skill

An agent skill for [PolyInsiders](https://api.polyinsiders.com) — smart-money
signals on Polymarket prediction markets.

Drop it into any agent that reads `SKILL.md` (Claude Code, Claude Agent SDK, or
your own loader) and the agent learns how to answer questions like *"what are
informed traders betting on in geopolitics?"*, *"how did that call turn out?"*,
or *"what did this wallet's year look like?"* — with citable, permanent URLs
instead of guesses.

## Install

**Claude Code** — clone into your skills directory:

```bash
git clone https://github.com/<org>/insiders-skill.git ~/.claude/skills/insiders-bot
```

**Anything else** — the skill is a single self-contained `SKILL.md`. Load it as
a system prompt fragment, a tool description, or context.

## What it wraps

Three public, unauthenticated, read-only endpoints. No API key, no signup.

| Endpoint | Returns |
|---|---|
| `GET /v1/stats` | Track record — resolved calls, hit rate, per-category breakdown |
| `GET /v1/signals` | Recent signals, filterable by `category` and `status` |
| `GET /v1/wrapped/:address` | Any Polymarket wallet's year in review |

```bash
curl -s https://api.polyinsiders.com/v1/stats
```

## How the signals are made

A wallet **qualifies** in a category at ≥70% trade-level win rate over ≥15
resolved buys *in that category*. A signal **publishes** only when ≥3 qualified
wallets independently buy the same outcome. Resolution is on-chain settlement.

A signal means "several independently-good wallets agree" — not a prediction.
The skill is explicit about this so agents describe it accurately.

## Why the skill is opinionated

Most of `SKILL.md` is not endpoint documentation — it's the handful of things
that make the difference between an agent citing this data correctly and an
agent quoting a misleading number:

- **Always pair a return with its entry price.** "+476%" is noise; "+476% from a
  67¢ entry" is a fact. Payout is capped at 1.00, so the entry price *is* the
  return.
- **`avgWinPct` is the mean profit on winners only** — not a portfolio return.
- **Check `truncated` before quoting a Wrapped.** Polymarket's history API
  refuses offsets past 5,000, so busy wallets return only recent trades. Counts
  become floors, and time-shaped fields are omitted rather than computed from a
  biased sample.
- **Win rate and realized P&L are deliberately absent** from Wrapped. Upstream
  `closed-positions` returns only a wallet's *winning* closed positions, so any
  win rate derived from it is 100% for every wallet alive. No number beats a
  flattering fake one.

## Not financial advice

Hit rate is historical and does not carry forward. The skill instructs agents to
say so when a user sounds like they're about to act on it.

## License

MIT
