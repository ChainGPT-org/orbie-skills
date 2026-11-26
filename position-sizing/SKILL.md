---
name: position-sizing
description: Size any new position from a risk budget before preparing it: risk per trade, volatility targeting, capped fractional Kelly, concentration and correlation caps, and cutting size after losses, mapped to the agent's limits and loss protection. Use when the user or an agent opens or adds to a position, sets up an agent that trades, or asks how much to put in, what size or leverage to use.
---

# Position sizing

Size first, then prepare. `size_position` does the arithmetic; this skill says which inputs to feed it. Thresholds marked *(default)* are Orbie's judgment, not a rule from a source: say so when you quote them.

1. **Read the rules you trade under.**
   - `list_agents`: the agent's `limits`: per-trade and daily USD caps, and its loss protection (`maxLosses` in `lossWindowHours`, `maxDrawdownUsd`, `maxDailyLossUsd`). Loss protection pauses the agent when a rule trips; resuming is the owner's. Never size so one ordinary loss trips it.
   - `get_performance` (7d and 30d) for the wallet: current drawdown from peak. `get_net_worth` for equity and what is already held.
   - Perps: `hl_account` / `aster_account` / `gmx_positions` for free margin and open positions.
2. **Equity.** Size from the capital at risk: the agent wallet's value, or the budget the owner names. Not the owner's whole net worth unless they say so.
3. **Stop first.** From structure or volatility (e.g. 1.5–2× the 14-day average daily range from `mkt_candles`), never from the leverage. No stop = no size.
4. **Risk per trade.** 0.5–2% of equity lost at the stop (1% *(default)*). Then cap it by the agent's rules *(default)*:
   - loss at the stop ≤ ¼ of the room left before `maxDrawdownUsd`, and ≤ ⅓ of `maxDailyLossUsd`;
   - notional ≤ `perTradeUsd` (Orbie enforces it anyway; a bigger plan just stops at the owner).
   - With `maxLosses` = N, assume N losses in a row can happen: N × risk must be survivable.
5. **Volatility target.** Pass `symbol` (EWMA λ 0.94 of 90 daily closes) or `daily_vol_pct`. Size ∝ target vol ÷ forecast vol; the gain is smaller tails and drawdowns, not higher return [Harvey et al. 2018]. Target 20% a year per position *(default)*.
6. **Kelly, capped.** f* = p − (1−p)/b, b = average win ÷ average loss. Use at most ½ Kelly, and only from 100+ out-of-sample trades (`bt_walk_forward`, paper, live), never from a backtest alone. Half Kelly keeps about ¾ of the growth at half the volatility [Thorp; c(2−c) approximation]. `size_position` ignores Kelly under 100 trades and returns 0 when there is no edge.
7. **Concentration and correlation.** One non-major token ≤ 5–10% of equity *(default 10%)*; BTC/ETH may go higher if the owner says so. Correlated positions (alts vs ETH, an LST and ETH, two memecoins) are one bet: pass their total as `existing_exposure_usd`.
8. **After losses.** −10% from peak: halve new risk; −20%: no new risk until the owner reviews *(default brakes)*. Pass `drawdown_pct`. Per-agent kill at 1.5× the backtest's worst drawdown (backtest-protocol). After a loss protection pause, propose smaller limits rather than resuming at the old size.
9. **Run `size_position`** and take its `sizeUsd` (the smallest candidate binds; say which one). Perps: leverage only sets margin (`marginUsd`); keep liquidation at least twice as far as the stop (perp-trade-plan) and set a `position_health` alert (liquidation-defense).
10. **Show it** in one small table: equity, stop and distance, risk % and USD, binding method, size (USD and units), margin and leverage, loss at the stop, which limit would stop it. Then follow swap-execution or perp-trade-plan and `execute`: the review card and the agent's limits still decide.

Never raise risk %, leverage or a cap to make a trade fit, and never size to "win back" a loss. Every number here is a ceiling, not a target.

Sources: Harvey, Hoyle, Korgaonkar, Rattray, Sargaison, Van Hemert, "The Impact of Volatility Targeting" (JPM 2018), https://people.duke.edu/~charvey/Research/Published_Papers/P135_The_impact_of.pdf ; Thorp, "The Kelly Criterion in Blackjack, Sports Betting and the Stock Market", https://sites.oxy.edu/lengyel/M330/thorp/paper.htm . RiskMetrics λ = 0.94 is the standard daily EWMA decay.
