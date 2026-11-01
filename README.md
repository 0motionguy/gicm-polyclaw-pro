# @gicm/polyclaw-pro

A signal and risk engine for prediction markets: seven strategies over Polymarket and Kalshi data, position tracking and risk limits. It does not place orders.

Polyclaw Pro scans open markets on Polymarket and Kalshi, runs seven heuristic strategies over price and volume history, aggregates the signals per market and sizes tracked positions against a risk manager. It is for developers building prediction-market research tools or agent assistants who want the analysis and bookkeeping layer. It reads public market data only; there is no wallet and no order placement.

Part of [gICM](https://github.com/0motionguy/gICM), a family of OpenClaw agent packages.

## What it does

- **Scanner:** `MarketScanner` pulls active markets from the public Polymarket and Kalshi APIs, filters by volume, liquidity and keywords, normalises them to one `Market` type and keeps up to 100 price points per market.
- **Seven strategies:** momentum, contrarian (prices beyond 15% or 85%), cross-platform arbitrage (spread of 3% or more), volume spike (3x average), mean reversion (15% deviation), event catalyst (within 3 days of resolution) and whale following (a volume-plus-price-move proxy for large-trader activity). Thresholds are configurable per strategy.
- **Signals:** `SignalEngine` runs the enabled strategies with per-strategy weights and aggregates a consensus, average confidence and signal count per market.
- **Risk:** `RiskManager` checks a confidence floor, open-position count, daily loss, per-position and total exposure, and liquidity, shrinks the size where it can, and sets stop-loss and take-profit (15% and 50% by default).
- **Positions:** `PositionManager` tracks open and closed positions in memory, fires stop and take-profit triggers on price updates, and reports win rate, Sharpe ratio, drawdown and per-strategy results.
- **Facade:** `PolyclawPro` ties it together: `scan()`, `evaluate()`, `execute()` (opens a tracked position), `closePosition()` and optional periodic scanning (every 10 minutes by default).

## Example

```ts
import { PolyclawPro } from "@gicm/polyclaw-pro";

const polyclaw = new PolyclawPro({
  platforms: ["polymarket"],
  risk: { maxPositionSize: 100, dailyLossLimit: 50 },
});

const aggregates = await polyclaw.scan();            // needs network access
const signal = aggregates[0]?.signals[0];
if (signal) console.log(polyclaw.evaluate(signal, 50)); // risk verdict and suggested size
```

## How it's built

TypeScript (ESM) bundled with tsup and tested with vitest; zod validates the config. Market data is fetched with plain `fetch` from the two public APIs, with no SDK and no native dependency. Defaults for risk and strategies sit in `src/types.ts`.

## Repository layout

- `src/`: `scanner`, `strategies`, `signals`, `risk`, `positions`, `polyclaw`, `types`
- `src/__tests__/`: vitest suite (63 test cases)
- `skill/SKILL.md`: the OpenClaw skill definition

## Run it locally

```bash
pnpm install
pnpm build   # tsup -> dist/
pnpm test    # vitest (add --run for a single pass)
```

## Status

Beta. Version 1.0.0, a single extracted release (February 2026) with a test suite. The strategies are simple heuristics and the repo ships no backtests or performance records, so treat the output as research input rather than advice.
