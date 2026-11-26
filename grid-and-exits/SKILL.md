---
name: grid-and-exits
description: How to put exits on a position (stop-loss, take-profit, time limit, trailing stop), staged exits on a spot token (sell 50% at 2×, trail the rest), and how to run a grid from backtest to paper to live, with the cap approved once. Use when the user asks for a trailing stop, a time stop, "close it if…", a stop on spot, a grid or range bot, or to protect a position that's open.
---

# Grids and exits

Both are executors: Orbie runs them itself every 15 s, without the model, on the rules Hummingbot's strategy_v2 uses (ported, Apache-2.0; see NOTICE.md). The same code runs in backtests, paper and live, so a backtest of a grid or of an exit set is the thing that will trade, minus what a backtest can't model.

## Exits on a position (`exit_prepare`)

One policy per position, measured from the entry:
- **Stop-loss** `stop_loss_pct`, **take-profit** `take_profit_pct`: on perps (Hyperliquid, Aster, exchanges that take them) they become trigger orders **on the venue** and work while Orbie is off. On spot, or where the venue won't hold them, Orbie watches them.
- **Time limit** `time_limit_minutes` (from when the policy starts) and **trailing stop** (`trailing_activation_pct` or `trailing_activation_price`, plus `trailing_delta_pct`): always watched by Orbie. The trailing stop arms once the position is that far in profit, then closes it when profit falls `trailing_delta_pct` points below its best; it never moves back.
- Every exit Orbie makes is a **market** close (reduce-only on perps; spot sells at most the covered amount). A resting limit for a stop or a time limit might never fill.
- Exits Orbie watches only work **while Orbie runs**. Say so whenever a policy relies on them, and prefer a venue-held stop-loss for the hard floor.

Steps: read the position (`hl_account`, `aster_account`, `cex_positions`/`cex_balances`), propose levels that fit the plan (the `perp-trade-plan` and `position-sizing` skills), `exit_prepare`, show the card's prices, `execute`. Check with `exit_list`. A policy ends itself when the position closes; `exit_prepare_cancel` stops Orbie watching (trigger orders it placed stay on the venue until cancelled there). If the user adds to the position, the perp policy still closes the whole position at its levels from the first entry; prepare a new one if the plan changed.

## Staged exits on a spot token (`exit_prepare_staged`)

For a Solana or EVM token the wallet holds, typically right after a buy: take-profit steps (`take_profits`: sell `sell_pct` of the covered amount at `multiple` × entry), a trailing stop on the rest (`trailing_pct`: sold once the price falls that % below its peak; `trailing_arms_at` to arm it only from a multiple) and a stop-loss (`stop_loss_pct`: everything still covered). Measure from `entry_price` = what was paid per token (from the buy's card); left out it is the price now, and say so.
- On Solana selling into USDC, take-profits of at least $5 rest on **Jupiter** as limit orders: they fill while Orbie or the Mac is off. Everything else (trailing stop, stop-loss, EVM steps, small steps) is **watched by Orbie** every 15 s and sold at market, **only while Orbie runs**. The card lists which is which; repeat it to the user.
- It sells only into SOL, USDC or USDT on Solana, and the network's coin, its wrapped coin, USDC, USDT or DAI on EVM networks (Orbie's pinned addresses); anything else is refused. The card is valued at the covered holding, so an agent arms it alone only within its limits.
- Each sell is a normal swap (simulated, checked; on Solana Lighthouse-guarded and sent at the owner's landing speed). Default slippage 0.5%; on thin memecoins a stop may need 1–3%, and a wider one is what gets sandwiched, so raise it only with the user.
- Every level must be ahead of the price; to sell now, use a swap. The stop cancels the plan's Jupiter orders and sells what comes back. `exit_list` shows each step's state; `exit_prepare_cancel` with the plan id (`sx…`) ends it and cancels its Jupiter orders.

## Grids

Config: `side` (long; short needs perps), `lower`/`upper`, `levels` (evenly spaced, both ends included), `size_usd` per level, `take_profit_pct` per level, `max_open_orders` (resting entries, nearest the price, only on the side they rest on), `stop_price` beyond the losing edge (cancel everything, close what the grid holds at market). **Cap = levels × size_usd**, the most it holds if every level fills.

1. **Backtest:** `grid_backtest`. Report it like any backtest (`backtest-protocol` skill): window, fees, return next to buy-and-hold, max drawdown, round trips (under 30 is an anecdote), max exposure against the cap, what it still holds at the end. Bars are walked open → favourable extreme → adverse extreme → close, so no bar makes a round trip it might not have. Not modelled: funding, slippage, queue position, partial fills.
2. **Paper:** `grid_paper_start`, read with `grid_list` and `paper_account`. Days, not minutes.
3. **Live:** `grid_prepare` on Hyperliquid (perps or spot) or a connected exchange. The card shows the cap: that is what the owner approves (or the agent's per-trade limit must cover) and what counts against the daily limit, once. Orbie places each order through the venue's own checks and never past the cap; every order shows in Activity. Stop with `grid_prepare_stop` (`close_position` to sell what it holds).

A grid makes money in a range and loses in a trend: below the range a long grid holds a falling position, every level at a loss. Always set `stop_price` unless the user decides otherwise in writing, and size the cap so the stop's loss (roughly cap × distance to the stop, plus fees) fits their plan. Take-profit per level should clear fees on both fills several times over. A paused agent's grid keeps its take-profits and stop but places no new entries. Backtests and paper are simulations, not promises.
