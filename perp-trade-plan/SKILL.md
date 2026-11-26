---
name: perp-trade-plan
description: Plan a leveraged perpetual futures trade before placing it: risk, size, margin vs notional, leverage, liquidation distance and funding. Use when the user wants to long or short with leverage, open, add to, close or resize a Hyperliquid position (hl_prepare_order), or asks about margin, liquidation or funding on a perp.
---

# Perp trade plan

Build the plan from tool data, show it, then prepare. Always say testnet or mainnet.

1. **Market.** `hl_markets`: mark and oracle price, max leverage, size decimals, funding, open interest. `mkt_candles`: recent range and volatility (e.g. average daily range over 14 days).
2. **Account.** `hl_account`: account value, free margin, existing position in this coin (a new order adds to or nets against it), open orders.
3. **Risk first, then size.**
   - Max loss the user accepts. Suggest at most 1–2% of account value per trade; `size_position` applies the position-sizing caps (volatility, concentration, drawdown brakes).
   - Stop from structure or volatility, not from the leverage.
   - Notional = max loss ÷ distance to stop (as a fraction). Leverage only sets the margin that notional locks: **margin = notional ÷ leverage**.
   - Always report margin and notional separately, with leverage and cross or isolated. Never call notional "what you're putting in".
4. **Liquidation.** Roughly 1/leverage minus maintenance margin away (10x ≈ 9%, 20x ≈ 4%; earlier in cross margin when other positions lose). Use the estimate the prepared order returns. Liquidation should be at least twice as far as the stop. If the stop is beyond liquidation, the plan is wrong: lower the leverage, not the stop.
5. **Funding.** Hourly rate from `hl_markets` or `mkt_funding`. Cost = notional × hourly rate × hours held. Positive funding: longs pay shorts. State it in USD per day.
6. **Order type.** Market = IOC limit at mid ± slippage (default 1%; the review shows the worst fill). Limit for entries at a price. `reduce_only` for exits. Minimum order value $10. Read `hl_prepare_order`'s margin check and warnings.
7. **Show the plan** as a small table: side, entry, size (coin and USD notional), leverage, margin, stop, target, liquidation estimate, loss at stop, funding per day. Then `execute`.
8. **After.** Confirm with `hl_account` (size, entry, liquidation price). Put the stop on the exchange with `hl_prepare_protect`, and add a `position_health` alert (liquidation-defense).

Never raise leverage to make a small account trade a big size. New strategy: paper it first (`paper_order`).
