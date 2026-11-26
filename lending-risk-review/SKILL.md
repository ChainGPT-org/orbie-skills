---
name: lending-risk-review
description: Risk review for supplying, borrowing, looping or withdrawing on Aave (aave_* tools), Kamino, MarginFi or another lending market: health factor, liquidation price, thresholds and rate risk. Use when the user wants to borrow, post collateral, leverage by looping, chase a supply yield, or asks how close a position is to liquidation.
---

# Lending risk review

0. **Protocol.** `protocol_info` on the lender (aave-v3, sparklend, compound-v3, morpho-blue, kamino-lend, marginfi, or the one asked about): red flags, TVL trend, recent hacks. For a lender or market not on Orbie's list, run the full protocol-risk-review before any deposit.
1. **Now.** `aave_positions` per chain: collateral, debt, health factor (HF), available to borrow. `aave_markets` per asset: LTV, liquidation threshold (LT), supply and borrow APY, available liquidity, oracle price.
   On Solana: `kamino_positions` / `kamino_markets`; `marginfi_positions` / `marginfi_banks` for MarginFi (Project 0). MarginFi weighs each bank twice: *initial* weights decide what you can borrow, *maintenance* weights decide liquidation. Its health factor is Σ(collateral × maintenance weight) ÷ Σ(debt × maintenance weight), read by simulating MarginFi's own health check, so it is exact for the moment read; its app's "health %" is 1 − debt/collateral on the same weights.
2. **HF after the action.** HF = Σ(collateral × LT) ÷ debt. Compute it for the proposed change. Orbie refuses borrows and withdrawals that leave HF under 1.1; that is a floor, not a target. Suggest at least:
   - 2.0 for volatile collateral (ETH, BTC) against stablecoin debt;
   - 1.5 for correlated pairs (stETH/ETH, stable/stable).
   Below 1.3, say plainly that the position is close to liquidation.
3. **Liquidation price.** One volatile collateral against stable debt: liquidation price = current price ÷ HF. Express it as a % drop and compare with `mkt_candles` history (worst day and worst week in the last year). If an ordinary bad week liquidates it, it's too tight.
4. **What liquidation costs.** Under HF 1, liquidators repay up to half the debt (all of it under 0.95) and take collateral plus a bonus of roughly 5–10% depending on the asset. That loss comes on top of the price move.
5. **Rate risk.** Borrowing is variable rate; high utilization pushes it up fast. Net carry = collateral yield − borrow cost on the debt. Reward APY (`mkt_yields`) depends on emissions and token prices: not guaranteed.
6. **Other risks.** Oracle and depeg risk for pegged collateral (stETH, USDe, bridged assets; see stablecoin-depeg-response); withdrawal liquidity (at high utilization you may not get your supply back right away); isolation and e-mode caps; protocol and chain risk.
7. **Loops** (supply → borrow → swap → supply): exposure ≈ 1 ÷ (1 − LTV used). Model the final HF and liquidation price, not each step. Every swap leg follows the swap-execution checklist.
8. **Act.** `aave_prepare_supply` / `_withdraw` / `_borrow` / `_repay` → `execute` (approvals included). Kamino: `kamino_prepare_*`. MarginFi: `marginfi_prepare_deposit` / `_withdraw` / `_borrow` / `_repay` (amount or "all" for withdraw and repay; the card shows HF now → after from the simulated health check, and the floor is checked again right before signing). MarginFi's own initial weights may refuse a borrow before Orbie's floor does; quote the error. If Orbie refuses for health, quote the safe maximum from the error. Afterwards, re-read `aave_positions` and report the new HF.
9. **Monitor.** Every borrow gets an alert: `trigger_create` with `when: position_health` and the venue (warns at HF 1.3, critical at 1.1 by default; tighter for volatile collateral). `wake` the agent that owns the position to defend it within its limits (liquidation-defense), or `notify` the owner.

Report a before/after table: collateral, debt, HF, liquidation price and % drop, net APY.
