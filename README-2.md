# Deriv Trading Platform — Dashboard

This is the **dashboard / frontend** half of the platform: a read-and-view
operational console that consumes the backend's `/api/v1/*` endpoints. It
contains no trading logic of its own — all execution, risk enforcement, and
state live in the companion `deriv-trading-bot-backend` package.

> **Status:** Pre-implementation. Dashboard build begins at Phase 9 of the
> overall project (see the backend package's `PHASES.md`), after the
> backend API it depends on exists.

## Purpose

An operational console for a single operator — fast, clear, and legible
during an incident, not a polished consumer product. Dark theme only, no
decorative UI or unnecessary animation.

## Technology Stack

| Layer | Choice |
|---|---|
| Framework | React + Vite |
| Styling | Tailwind CSS |
| Charts | TradingView Lightweight Charts |

## What It Displays

- **Header** — Active Account, Balance, Equity, Daily P&L, Trading Status
- **Live Market** — real-time candlestick chart, current price/symbol/
  timeframe, buy/sell markers, executed trades
- **Active Trades** — symbol, contract, stake, entry price, P&L, status,
  time opened
- **Trade History** — searchable, time/symbol/direction/stake/result/P&L
- **Statistics** — total trades, win rate, consecutive wins/losses, daily
  P&L, average execution time
- **Status Bar** — TradingView, Queue, Deriv, Database, Railway, Last
  Signal, Last Trade, Current Latency
- **Account Mode control** — Demo/Live toggle, gated by the backend's demo
  validation criteria, requiring the operator to type
  `ENABLE LIVE TRADING` to confirm switching to Live

## Data Source

This app has **no direct connection to Deriv, Postgres, Redis, or
Telegram.** Everything it shows comes from the backend's versioned API:

- `GET /api/v1/health` — health level and subsystem checks
- `GET /api/v1/status` — active account, trading enabled/disabled,
  maintenance mode, circuit breaker state
- `GET /api/v1/trades` — trade history and active trades

See the backend package's `docs/API.md` for the full contract.

## Environment Configuration

```
VITE_API_BASE_URL=https://your-backend-domain/api/v1
```

No secrets belong in this package — the dashboard never holds a Deriv API
token, database credential, or Telegram token. It only ever talks to the
backend's own API.

## Documentation

- [ADR 0006: Dashboard Architecture](./docs/adr/0006-dashboard-architecture.md)

## License

MIT — see [LICENSE](./LICENSE).

---

## ADR 0006: Dashboard Architecture

# ADR 0006: Dashboard Architecture

**Status:** Accepted

## Context

The dashboard is an operational console for a single operator, not a
consumer product — it needs to be fast, clear, and trustworthy under
degraded conditions, not visually elaborate.

## Decision

React + Vite + Tailwind CSS, dark theme only, no decorative UI or
unnecessary animation. TradingView Lightweight Charts for the live market
view. The dashboard reads from the same `/api/v1/*` endpoints used
elsewhere (Status, Trades, Health) rather than a separate privileged
internal API — it has no special access the operator's own API calls
don't have.

## Alternatives Considered

- Server-rendered dashboard (e.g. EJS/Pug) — rejected: live-updating
  charts and trade state benefit from a client-side reactive framework.
- A separate internal-only API for the dashboard — rejected: unnecessary
  duplication for a single-operator system; adds surface area without
  benefit.

## Rationale

Simplicity and operational clarity outweigh visual polish for a console
whose job is to make health, risk, and trade state legible at a glance,
especially during an incident.

## Consequences

Dashboard functionality is naturally limited to what the API exposes —
any new dashboard capability requires a corresponding API capability,
which keeps the two in sync by construction.
