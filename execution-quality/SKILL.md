---
name: execution-quality
description: Execute size well and check afterwards: compare the order with order-book depth (cex_orderbook) or quotes across DEX aggregators (dex_compare), split when it's large against ±1% depth, keep flow private where possible, then measure the fill with execution_report (quoted vs filled, vs market, fees, sandwich check). Use for trades above about $10,000, thin tokens or markets, agents trading on a schedule, or when the user asks whether a trade filled well. Extends swap-execution.
---

# Execution quality

swap-execution covers every trade; this adds what matters with size. Thresholds marked *(default)* are Orbie's judgment.

## 1. Before: size against liquidity
- **Exchange orders:** `cex_orderbook` with the side and size (`amount` in base or `quote_amount`). Read the spread, the depth within ±0.1/1/2% of mid, the walked average price and `slippageBpsVsMid`, and `shareOf1PctDepth`. ±1% depth is the standard measure of a market's liquidity [Kaiko]; a high 24 h volume with thin depth suggests wash volume. A band marked `partial` means the fetched book ended first: the real depth is larger.
- **DEX swaps:** `dex_compare` across aggregators for the exact size, in both directions for thin tokens; the gap between the best quote and `mkt_price` is impact plus fees.
- **Split above 10% of the side's ±1% depth *(default)***, or when the walked slippage exceeds what the trade can afford: TWAP on Hyperliquid (`hl_prepare_twap`), DCA on Solana (`jup_prepare_dca`), several limit orders on an exchange, or a CoW Swap order (`dex_prepare_swap` with source "CoW Swap"; batch auctions match orders off-chain, so there's no public pool trade to sandwich). Under 10%: one order is reasonable.
- Market orders on an exchange fill through the book: the card's estimated price is the best level now, not the average you'll get.

## 2. During: keep it private
- Orbie sends privately where a route exists (owner setting, on by default; the card's "MEV protection" line): Ethereum via Flashbots Protect, which retries for up to 25 blocks [Flashbots docs], with MEV Blocker as backup; BNB Chain via PancakeSwap MEV Guard; Solana swaps as Jito bundles; Arbitrum, OP Mainnet, Base and Unichain straight to the sequencer. Elsewhere (Polygon, Avalanche…) the mempool is public: split and tighten slippage instead.
- L2 sequencers keep their mempool private, so sandwiches are rare there, but the sequencer still orders trades: keep slippage tight.
- Minimum received caps the loss at the slippage you allow. Don't widen slippage to make a big order go through; split it.

## 3. After: measure, don't assume
`execution_report` (default: the latest confirmed swap or order; or `activity_id` from `get_activity`):
- **EVM swaps:** `vsQuotedPct` (received vs the card's expected amount), `belowMinimum`, `vsMarketPct` (shortfall vs DefiLlama prices at that block's time: price impact plus every fee, the wallet's own included), gas in USD and `gasPctOfTrade`, and a **sandwich check** on the transactions right before and after in the block:
  - `likely`: both neighbours, from one sender, traded the same pool. Report it with the report's `suggestion` (private sending in Settings, or tighter slippage where there is no private route) and `sentVia`.
  - `possible`: a same-pool trade on one side, or from different senders; common on busy pools, not proof.
  - `none` / `unknown` (an RFQ or intent fill with no pool event: nothing to sandwich in the pool).
- **Exchange orders:** average fill vs the card's price and vs the 1-minute candle open when it was placed (arrival price), fee in bps, partial fills.
- Report the numbers as measured, with what is `unmeasured`. A shortfall vs quote beyond the slippage you allowed on a pool swap means look at the sandwich result; repeated shortfalls on a schedule mean the agent's size is too big for the market: halve it or move it to a TWAP.

## 4. Agents on a schedule
- Size each run below 10% of ±1% depth *(default)*; check `cex_orderbook` or `dex_compare` before each large run, not once.
- Run `execution_report` after each trade and stop the agent if `vsMarketPct` keeps exceeding its edge: costs are eating the strategy (see backtest-protocol, step 4).

Sources: Kaiko on exchange liquidity and 1% depth (kaiko.com/resources/understanding-centralized-exchange-liquidity-data); Flashbots Protect (docs.flashbots.net/flashbots-protect/overview); Jito low-latency sending (docs.jito.wtf/lowlatencytxnsend); DefiLlama historical prices (coins.llama.fi).
