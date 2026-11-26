---
name: liquidation-defense
description: Watch and defend borrowed or leveraged positions before they are liquidated: health-factor and margin alerts, how liquidation works on Aave, Spark, HyperLend, Compound v3, Morpho, Kamino, MarginFi, Hyperliquid, Aster and GMX, and the defense ladder (add collateral, repay with stables, deleverage, close). Use when a position_health or near_liquidation alert fires, when the user borrows or opens leverage, or asks how close a position is to liquidation or how to protect it.
---

# Liquidation defense

Liquidation costs more than the price move: a penalty or bonus to the liquidator, and on perps the margin itself. Act early and in small steps, every one through Orbie's review cards and the agent's limits. Thresholds marked *(default)* are Orbie's judgment.

## 1. Set the watch (do this whenever a borrow or leveraged position opens)
- `trigger_create` with `when: position_health`, `venue` (aave, spark, hyperlend, compound, morpho, kamino, marginfi, hyperliquid, aster, gmx) and the wallet. Defaults *(default)*: lenders warn at health factor (HF) 1.3, critical 1.1; perps warn 10% and critical 5% from liquidation. It fires once per step down, critical without waiting for the cooldown, and always notifies the owner (Mac notification, toast, Telegram when linked).
- `action: wake` hands the alert to the agent that owns the position (an agent can only wake itself); use `notify` when nobody should act unattended. Waking spends model budget.
- Tighter where moves are fast *(default)*: volatile collateral or > 5x leverage → warn at HF 1.5 / 15%. Morpho watches Orbie's market list only; say so if the position is elsewhere.
- For a borrow the owner wants defended without the model: `lend_guard_prepare` (Aave, Spark, HyperLend, Compound, Morpho, Kamino). Below `trigger_hf` *(default 1.25)* Orbie repays the debt asset from the wallet up to `target_hf` *(default 1.5)*, within `cap_usd`, every 15 s, only when the simulation shows health rising. One card approves the policy and cap. It runs only while Orbie runs (near the trigger it keeps the Mac awake; a closed lid still stops it): say so. There is no flash-loan deleverage.
- Lenders (supply side): `trigger_create` with `market_utilization`, `liquidity_below_supply`, `market_frozen`, `peg_off` (a stable vs $1, an LST/LRT vs the lender's oracle) or `protocol_hacked` (DefiLlama's hacks list, heuristic). When one fires, `lend_market_risk` then `lend_prepare_exit` withdraws while liquidity exists; it never swaps.
- Don't rely on someone watching a screen: cascades move faster than people (October 2025 saw over $19B of perps liquidated, per Galaxy's coverage).

## 2. When it fires: read, don't guess
- Re-read the position: `aave_positions`, `spark_positions`, `hyperlend_positions`, `comet_positions`, `morpho_positions`, `kamino_positions`, `marginfi_positions`, `hl_account`, `aster_account`, `gmx_positions`. Report HF or liquidation price, distance in %, collateral, debt, and the price that liquidates.
- Compare the distance with recent moves (`mkt_candles`: 14-day average daily range, worst day this year). Aim to restore at least 3 average daily ranges of room *(default)*, or HF ≥ 1.5.

## 3. How each venue liquidates
- **Aave v3 / SparkLend / HyperLend** (Spark and HyperLend are Aave v3 forks): below HF 1 a liquidator repays up to 50% of one debt and takes collateral plus a bonus (about 5–10% by asset); at HF ≤ 0.95 up to 100% [Aave `LiquidationLogic.sol`]. Newer versions also allow 100% when the position is small (around $2k; not verified for every deployment).
- **Compound v3**: below the liquidation factor the protocol absorbs the whole account, all collateral, and the borrower loses the liquidation penalty (Orbie's `comet_markets` shows it per asset) [Compound III docs].
- **Morpho Blue**: each market is isolated with a fixed LLTV and oracle; above LLTV anyone can repay and seize collateral with an incentive of min(15%, 1/(0.3·LLTV + 0.7) − 1) [Morpho docs].
- **Kamino**: at HF 1 it liquidates part of the collateral at a penalty (see `kamino_markets` for the reserve; the exact bonus schedule is in Kamino's docs, not read by Orbie).
- **MarginFi** (Project 0): liquidatable when maintenance-weighted collateral is below maintenance-weighted debt (HF < 1). Liquidators take collateral at a discount: classic liquidation pays the liquidator 2.5% plus 2.5% to insurance by default (set per debt bank); receivership liquidation lets them unwind the account and keep up to 10%. Either way only until health is back to 1, so a small breach loses only part of the collateral [0dotxyz/marginfi-v2 guides: CLASSIC_LIQUIDATION.md, RECEIVERSHIP_LIQUIDATION.md]. Prices are Pyth or Scope oracles; a stale oracle blocks borrows and withdrawals, not repays and deposits.
- **Hyperliquid**: maintenance margin is half the initial margin at max leverage (1.25% at 40x … 16.7% at 3x). Positions over $100k are liquidated 20% at a time; below ⅔ of maintenance the liquidator vault takes the position and the remaining margin is not returned; cross positions share margin and can be taken together [Hyperliquid docs: margining, liquidations].
- **Aster, GMX**: read the liquidation price from the position (`aster_account`, `gmx_positions`; GMX's is an estimate: price impact and fees still to accrue move it). GMX keepers close the whole position.
- **Isolated vs cross**: isolated risks only that position's margin; cross uses the whole account, so another position's loss moves this liquidation price.

## 4. Defense ladder (in order; stop when the target is restored)
1. **Add collateral** of the same or a less volatile asset, or margin: `aave_prepare_supply`, `spark_prepare_supply`, `hyperlend_prepare_supply`, `comet_prepare_supply`, `morpho_prepare_supply_collateral`, `kamino_prepare_supply`, `marginfi_prepare_deposit`; perps: move USDC to perp margin (`hl_prepare_class_transfer`, `hl_prepare_deposit`).
2. **Repay with stablecoins** already in the wallet: `aave_prepare_repay`, `spark_prepare_repay`, `hyperlend_prepare_repay`, `comet_prepare_repay`, `morpho_prepare_repay`, `kamino_prepare_repay`, `marginfi_prepare_repay` ("all" closes the debt). Repay the debt that raises HF most per dollar (usually the largest).
3. **Deleverage**: sell some collateral for the debt asset (swap-execution checklist), then repay; on perps reduce size (`hl_prepare_close` with a partial size, `aster_prepare_close`, `gmx_prepare_close`). Partial first.
4. **Close** the position entirely when the plan's stop is hit, the critical level is breached and steps 1–3 can't restore room within limits, or the market is gapping.
- Perps also get a stop on the venue where Orbie can place one (`hl_prepare_protect`, `aster_prepare_protect`, `gmx_prepare_tpsl`).

## 5. Rules
- Every step is a `*_prepare_*` → review card → `execute`. The owner approves with Touch ID; an agent acts only within its per-trade and daily limits and its loss protection. If a step is refused, tell the owner what it would take and stop; never split a step to fit under a limit or retry an old intent.
- Orbie refuses borrows and withdrawals that leave HF under 1.1: that is a floor, not a target.
- Never add leverage or borrow more to "defend". Never move collateral away from a position near liquidation.
- Report before/after: HF or distance, liquidation price, collateral, debt, and the transaction hashes from `execute`.

Sources: Aave v3 LiquidationLogic.sol, https://github.com/aave/aave-v3-core/blob/29ff9b9f89af7cd8255231bc5faf26c3ce0fb7ce/contracts/protocol/libraries/logic/LiquidationLogic.sol and https://help.defisaver.com/protocols/aave-v3/liquidations-on-aave ; Hyperliquid https://hyperliquid.gitbook.io/hyperliquid-docs/trading/margining and https://hyperliquid.gitbook.io/hyperliquid-docs/trading/liquidations ; Compound III https://docs.compound.finance/liquidation/ ; MarginFi https://github.com/0dotxyz/marginfi-v2/tree/main/guides/RISK_AND_LIQUIDATORS ; Morpho https://docs.morpho.org/learn/concepts/liquidation/ ; Galaxy, crypto leverage Q3 2025, https://www.galaxy.com/insights/research/crypto-leverage-q3-2025-defi-cefi-lending-digital-asset-treasury-debt-futures-perpetuals .
