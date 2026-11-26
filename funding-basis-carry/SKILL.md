---
name: funding-basis-carry
description: Plan, run and unwind a delta-neutral funding carry (cash-and-carry): short a perp that pays funding and hold the same amount long in spot (or a cheaper perp), entry and exit rules on multi-day realised funding averages, all four fees, borrow costs, liquidation distance on the short leg, venue risk and how to unwind. Use when the user wants to "earn funding", run a basis or cash-and-carry trade, go delta-neutral, compare a carry with Ethena (sUSDe) or Pendle fixed yield, or asks whether a funding rate is worth farming.
---

# Funding basis carry

A carry earns the funding that longs pay shorts while price moves cancel out between the legs. It is not risk-free: funding flips, four fees and the basis eat thin carries, and the short leg can be liquidated in a rally while the gain sits on the spot leg somewhere else. Every number below is labelled: *(sourced)* with its source, *(default)* Orbie's judgment, which the owner can change.

## 1. What the carry earns (read, don't assume)
- `funding_history` for the coin, 14–30 days, all venues. Decide on **realised multi-day averages** (`7d` and `14d`), never on one reading or on `mkt_funding`'s current rate. For GMX use `shortEarnsAprPct` (the short side's own rate net of borrowing; GMX longs and shorts get different rates).
- Read with it: `positiveSharePct7d` (how often shorts were paid), `negativeStreakHours` (paying now?), `rolling7dAprPct.worst` (how bad a week got this month), and `predicted.vsRealised7dAprPct` (the next interval vs the week; a big negative gap = funding is fading).
- `arb_scan_funding` ranks today's candidates with default-tier taker fees, breakeven hours and net APR over a hold; `cex_funding` / `aster_markets` / `gmx_markets` / `hl_markets` give each venue's current rate and open interest.
- Hyperliquid funding is paid hourly and capped at 4% an hour; with a flat premium its interest component alone pays shorts 0.01% per 8 h, about 11% a year, so positive funding is the resting state on majors there *(sourced: Hyperliquid docs, funding)*. A carry near 11% on BTC/ETH is the baseline, not an edge.

## 2. Costs and the real return
- **Fees: four fills** (enter and exit, both legs). Hyperliquid base tier: perp taker 0.045%, spot taker 0.07% → about 0.23% round trip at taker *(sourced: Hyperliquid's base-tier fee schedule; the account's tier may be lower)*. Exchanges: the taker fee `arb_scan_funding` shows per venue (CCXT default tier). Post-only limits (`tif: Alo`) cut this where the book allows.
- **Spread and basis**: entering when the perp trades below spot, or exiting when it trades above, costs the gap. Budget at least 15 bps for it *(default)*; one measured unwind cost 41 bps *(sourced: ChainGPT's carry research, below)*.
- **Breakeven** = round-trip cost ÷ daily carry. Example: 10% a year ≈ 2.7 bps a day; 23 bps of fees needs about 9 days, 40 bps about 15 days. Plan a hold well past breakeven, or don't enter.
- **Borrow**: GMX shorts pay borrowing (already in `shortEarnsAprPct`). If the spot leg is bought with borrowed stables (Aave, a margin loan, Hyperliquid portfolio margin), subtract that rate. Net carry = funding − borrow − fees amortised over the hold.
- **Which denominator**: the return on capital, not on notional. Spot paid in full plus margin for the short at leverage L uses notional × (1 + 1/L): 10% funding at 2x is about 6.7% on the capital. Always report both.

## 3. Entry rules *(default unless marked)*
Enter only when all hold:
- 7-day **and** 14-day realised short-leg averages ≥ round-trip cost amortised over the planned hold + 5% a year of margin for error, both windows `complete`.
- `positiveSharePct7d` ≥ 80%, `negativeStreakHours` = 0, and `rolling7dAprPct.worst` > 0 over the last 30 days. A worst week below zero means the carry flips in this regime; size down or skip.
- The short venue's open interest and depth take the size: under 1% of open interest, and `arb_scan_spreads` / a `dex_compare` quote shows the spot leg fills within the 15 bps budget.
- Not on coins with no spot leg you can actually hold (many of the highest-funding perps have none, or only a thin one).

## 4. Set it up
**Prefer one carry card**: `carry_plan` (both legs' fees from their own prepared orders, 30-day realised funding, basis, the short's liquidation distance, the exit plan), then `carry_prepare_open`: one review card that opens the long, shorts exactly what filled, and arms a guard that closes both legs (short first) near liquidation, after a day of wrong-way funding, or when either leg is gone. `carry_list`, `carry_prepare_close` to unwind. Legs it takes: Hyperliquid perps or spot, Aster perps, connected exchanges. The steps below are the manual way (other venues, TWAP sizing).
1. **Size** with `size_position` treating the short's liquidation distance as the stop, and keep the whole carry within the agent's limits and the concentration cap. Never split a trade to fit under a limit.
2. **Short leg**: leverage ≤ 3x, isolated or a dedicated account *(default)*, so it survives a 30% rally *(default)*. Liquidation is roughly 1/leverage minus maintenance away, and Hyperliquid's maintenance is half the initial margin at the coin's *maximum* leverage: BTC (40x max) at 3x ≈ 32% away, a coin capped at 3x has 16.7% maintenance, so ≈ 17% away: use 2x or less there. Use the prepared order's estimate *(sourced: Hyperliquid docs, margining and liquidations)*. Tools: `hl_prepare_order` (sell, `leverage`, `margin: isolated`), `cex_prepare_perp_order`, `aster_prepare_order`, `gmx_prepare_open`.
3. **Long leg**, same coin units as the short (not the same USD after slippage): `hl_prepare_spot_order` (Hyperliquid spot is often wrapped: UBTC, UETH, USOL), `cex_prepare_order`, `dex_prepare_swap`, or a perp long on a cheaper-funding venue (perp-perp: two margins, two liquidation prices).
4. **Leg risk**: prepare both, execute the hedge right after the first fill; large size goes in parts (`hl_prepare_twap` for the perp) with the other leg following each part.
5. **Watch**: `trigger_create` `position_health` on the short's venue (warn 15%, critical 8% from liquidation *(default)*), and `funding_below` on Hyperliquid at the breakeven APR. Re-run `funding_history` daily, not hourly.
Every step is a `*_prepare_*` → review card → `execute`; the owner approves, or the agent acts within its limits and loss protection.

## 5. Exit and unwind
Exit when any holds:
- the 7-day realised average falls below the fees still to pay amortised over the remaining hold *(default)*;
- funding has been negative for 24 hours (`negativeStreakHours` ≥ 24) *(default)*;
- the short is within the critical liquidation distance and can't be topped up from free margin;
- a venue incident: withdrawals paused, a depeg of the margin stable (stablecoin-depeg-response), an exchange or bridge problem.
Rally: the short loses margin while the spot gains elsewhere. Add margin first (`hl_prepare_class_transfer`, deposit) or cut both legs by the same amount; never close only the short (that leaves a naked long). Unwind both legs together, in parts on thin books, reduce-only on the perp (`hl_prepare_close`, `cex_prepare_perp_close`, `aster_prepare_close`, `gmx_prepare_close`), then sell the spot.

## 6. Venue risk
- Both legs on one exchange = one counterparty and one failure; split legs = transfers to rebalance margin, which take time when you need them fast.
- Wrapped spot (UBTC, UETH) adds its bridge; GMX adds keepers and oracles; a CEX adds custody and withdrawal freezes. Name each in the plan.
- Ethena (sUSDe, `ethena_info`) is this trade packaged: it adds custodian and exchange risk, and its reserve fund covers negative funding only up to its size *(sourced: Ethena docs, risks)*. Pendle PT-sUSDe (`pendle_markets`) locks a fixed rate but adds maturity and depeg risk. Compare their net yield with the carry's net, on the same denominator.

## 7. What carry research found (don't overstate the edge)
ChainGPT's own trading research pre-registered and ran carry tests on Hyperliquid in mid-2026:
- **Buy-and-hold carry** (spot long + perp short, held a year): annualised Sharpe of about 19 on HYPE looked excellent, but after deflating for the dozen variants tried it missed its own bar (deflated Sharpe 0.81–0.90 vs 0.95). An equal-weight 11-coin portfolio did worse (about 6% a year, deflated Sharpe 0.44). It stopped there instead of trying more variants.
- **Returns are lopsided**: skew around −6 to −8 and very fat tails. Months of small steady accrual, then a few days (squeezes, funding flips, basis blowouts) take a large part of it back. A high Sharpe on a short window says little.
- **Threshold-timed carry** (enter when hourly funding is high, hold 8 hours to 7 days), on coins with a Hyperliquid spot leg, after about 23–29 bps of round-trip costs: none of 48 variants cleared the bar and the best earned about nothing; short holds never earn back four fees. The most extreme funding sat on coins with no Hyperliquid spot leg, and was mostly negative (shorts paying).
- **Regime**: one year of HYPE funding averaged about 10.5% a year with no negative 30-day window, but its worst 30-day window was about 3.4%; a different year can be worse. Its live-test plans showed that thresholds set without measuring ordinary variance get breached by normal moves (a 15% rally tripped a margin gate; adverse basis exceeded a 40 bps budget).
- **Net on capital** came to about 5% a year on the account for a ~10% funding rate once borrow and the margin denominator were counted.
So: present carry as a modest, regime-dependent yield with tail risk, not a sure 10–20%. Paper it first (`paper_order` both legs) for a new coin or venue, and report realised net after fees, not the funding rate.

## 8. Report
Coin and venues; realised 7/14/30-day short-leg APR with sources and times; predicted vs realised; fees, basis budget and borrow; breakeven days; net APR on notional and on capital; short leverage, margin mode and liquidation distance; exit triggers set; venue risks; what you could not verify (e.g. a venue `unavailable` or a window not `complete`).

Sources: Hyperliquid funding https://hyperliquid.gitbook.io/hyperliquid-docs/trading/funding ; margining https://hyperliquid.gitbook.io/hyperliquid-docs/trading/margining ; liquidations https://hyperliquid.gitbook.io/hyperliquid-docs/trading/liquidations ; Ethena risks https://docs.ethena.fi/solution-overview/risks.
