---
name: lp-management
description: Open, watch and rebalance concentrated liquidity (Uniswap V3/V4, PancakeSwap V3) so fees actually beat impermanent loss and LVR: simulate the range first (fee APR from volume and in-range liquidity, IL vs holding, LVR, width table, rebalance cost), size the width from volatility, set an out-of-range alert, and rebalance only when it pays. Use when the user wants to provide liquidity, asks about an LP position's range, APR or IL, when an lp_out_of_range alert fires, or before any dex_prepare_lp_open / dex_prepare_lp_rebalance.
---

# LP management

A range position is a short-volatility trade: it earns fees and pays for it in impermanent loss (IL) and loss-versus-rebalancing (LVR). Thresholds marked *(default)* are Orbie's judgment.

## 1. Know what you pay
- **IL** is the loss against simply holding the two tokens; a range ends up all in whichever token fell. Concentration multiplies both fees and IL.
- **LVR** is the running cost of arbitrageurs trading against the pool at stale prices: σ²/8 of pool value per year for full range, times the range's capital efficiency [Milionis, Moallemi, Roughgarden, Zhang 2022]. Fees must beat LVR, not just the IL you see at the end.
- Evidence: in 17 large Uniswap V3 pools (May–Sep 2021) IL (~$260M) exceeded fees (~$199M), and about half of LPs lost against holding [Loesch et al., Bancor-commissioned].

## 2. Simulate before opening
- `lp_simulate` with the pool (base_token, quote_token, fee_pct) or an existing token_id. Read: `current.feeAprPct` vs `current.lvrAprPct` (`feeMinusLvrAprPct` must be clearly positive), `sim.netVsHoldPct` (p5 / median / p95 over simulated paths), `sim.timeInRangePct`, `sim.leftRangePct`, and the `widths` table.
- Quote the assumptions: fees from today's 24h volume (GeckoTerminal) and the position's share of in-range liquidity, volume held constant; no drift; the quote token's price held constant.
- If fee APR < LVR APR at every width, say so: holding (or a lending yield) beats this pool now. Don't open it to "earn APR".
- Stable/stable pools (USDC/USDT): IL is small until a depeg; the risk is the depeg (stablecoin-depeg-response), not volatility.

## 3. Pick the range *(default)*
- Width ≥ `rule.minWidthPct`: ± at least twice the expected move over your rebalance horizon (σ√t). Narrower earns more per dollar while in range but leaves it far more often.
- Prefer the width with the best `netVsHoldPct.median` whose p5 you can accept; avoid widths where `leftRangePct` > 50% over the horizon unless you will actively rebalance.
- Size: an LP position is exposure to both tokens; count it in `size_position` and `portfolio_risk` like the tokens themselves.

## 4. Open and watch
- `dex_prepare_lp_open` (review card states the range, deposit, minimums and the out-of-range risk) → owner approval or the agent's limits → `execute`.
- Then `trigger_create` `when: lp_out_of_range` with `lp_venue`, `token_id`, `chain` (warns 5% from an edge *(default)*, fires again when it leaves). `notify` unless an agent should act; `wake` hands it to the agent that owns the position.

## 5. When it leaves the range (or nears an edge)
1. Re-read: `dex_lp_positions` (in range? uncollected fees?), `lp_simulate` with the token_id for the current range and a new one.
2. Rebalance only if both hold *(default)*: the position is out of range (or within 1% of an edge with momentum against it), and the fees it should earn in the new range before its expected exit exceed the cost (`rebalance.totalUsd`; `breakEvenDaysInRange` well under `sim.medianDaysToExit`). Otherwise wait for the price to return, or close.
3. Don't chase: a range re-centred after every move turns IL into realised losses plus costs. If it left twice in a week, widen or stop.
4. `dex_prepare_lp_rebalance` with the new range: one card closes (fees collected, NFT burned), swaps the surplus token on the best direct venue and reopens. Each step is sized from the previous step's minimum, so leftovers stay in the wallet; the steps are simulated together and the card can't be sent unless that passes. If a step fails on-chain, the earlier ones stay done and the tokens wait in the wallet: report that, don't resend.
5. Replace the alert for the new token_id (the old one goes quiet once the position is closed).

## 6. Rules
- Every action is `*_prepare_*` → review card → `execute`, within the owner's approval or the agent's per-trade and daily limits. Never split an action to fit under a limit; never retry an old intent.
- Uniswap V4 pools with hooks are not rebalanced by Orbie; close and reopen them by hand.
- Report: range (price units), in range or not, fees earned, value vs holding, what you did and its hashes from `execute`, and what the simulation assumed.

Sources: Milionis, Moallemi, Roughgarden, Zhang, "Automated Market Making and Loss-Versus-Rebalancing" (2022), https://arxiv.org/abs/2208.06046 ; Loesch, Hindman, Richardson, Welch, "Impermanent Loss in Uniswap v3" (2021), https://arxiv.org/abs/2111.09192 ; Uniswap v3 whitepaper §6 (liquidity and ranges), https://uniswap.org/whitepaper-v3.pdf .
