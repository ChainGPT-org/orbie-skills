---
name: market-regime
description: Read the market regime (trend-up, trend-down, chop, high-vol) and crowding before sizing risk-on trades, setting agent schedules or answering "is now a good time": price vs 50/200-day averages and 30-day efficiency, 30-day realized volatility against its past year, funding and open-interest heat, then a stance (risk-on / neutral / risk-off) with a size multiplier for position sizing. Use before opening leveraged or directional positions, when an agent plans its next trades, or when the user asks about market conditions.
---

# Market regime

A regime is context for how much risk to take, never a signal to trade by itself. Thresholds marked *(default)* are Orbie's judgment.

## 1. Read it
- `market_regime` for BTC (the market's risk appetite) and for the coin you plan to trade. Quote `regime`, `stance`, `sizeMultiplier` and the four `reasons`, with the sources and time.
- **Trend** *(default)*: up when price and the 50-day average are above the 200-day average and the last 30 days moved up efficiently (Kaufman efficiency ≥ 0.3); down mirror-wise. A fast fall inside a long uptrend is a turn, not a trend. Otherwise chop.
- **Volatility** *(default)*: 30-day realized volatility as a percentile of the past year; ≥ 80th is high-vol, ≤ 20th low. Low volatility often precedes a large move; it is not safety.
- **Crowding** *(default)*: Hyperliquid funding (7-day realized average) above ~30% a year with open interest up > 10% in 7 days, or above 50% alone, means crowded longs (Hyperliquid's baseline is about 11% a year); below −10% crowded shorts. Crowded sides are liquidated first in a cascade.
- Add when it matters: `funding_history` across venues, `portfolio_risk` (what you already hold that moves with it), ChainGPT news/signals as context only.

## 2. Use it *(default)*
- Size multiplier: risk-on 1.0, neutral 0.5, risk-off 0.25 — multiply the risk per trade from `size_position` (or the agent's per-trade limit) by it, never above the limits.
- Trend-up and not crowded: normal size; trail stops rather than taking profit early.
- Chop: smaller size; mean-reversion and range LP fit better than breakouts (lp-management for ranges).
- High-vol or trend-down: risk-off; no new leverage, widen stops only by cutting size, check liquidation distances (liquidation-defense).
- Crowded longs in an uptrend: neutral; avoid adding leveraged longs; funding carry (funding-basis-carry) may pay instead.

## 3. Rules
- Re-read before acting; regimes change. An agent on a schedule reads it once per run, not once per day.
- Never present the regime as a forecast or a reason to buy or sell; it only scales risk. Every trade still goes through its review card and the agent's limits.
- Report: the regime, stance and multiplier per coin, the reasons with their numbers, what couldn't be read (funding or open interest), and the time.

Sources: Kaufman, "Trading Systems and Methods" (efficiency ratio); Hyperliquid funding docs, https://hyperliquid.gitbook.io/hyperliquid-docs/trading/funding ; OKX open-interest statistics API, https://www.okx.com/docs-v5/en/#trading-statistics-rest-api-get-contracts-open-interest-and-volume .
