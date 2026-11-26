---
name: swap-execution
description: Checklist to run before every token swap or bridge, up to execute and verification. Use whenever the user asks to swap, buy, sell, convert or bridge tokens (prepare_swap, sol_prepare_swap), and whenever an agent is about to trade.
---

# Swap execution checklist

Go top to bottom. At any failed gate, stop and tell the user why.

1. **Intent.** Chain (and destination chain for a bridge), from token, to token, amount in from-token units. Resolve both tokens to addresses (`get_token`). An unfamiliar to-token gets token-due-diligence first. Never invent a missing amount or chain: ask once.
2. **Funds.** `get_portfolio` (EVM) or `sol_portfolio`. Enough input token, plus native gas left afterwards for the approval and the swap. Don't spend the last ETH or SOL.
3. **Prepare.** `prepare_swap` (EVM via LI.FI; Sepolia via Uniswap v3; `to_chain` bridges) or `sol_prepare_swap` (Jupiter). Nothing is sent.
4. **Read the quote, then tell the user in one or two lines:**
   - Route or venue; for bridges the arrival chain and that arrival takes minutes.
   - Expected output and **minimum received**. The minimum is binding: if the price moves below it, execute refuses and you prepare again.
   - Effective price vs `mkt_price`: the gap is price impact plus fees. Over 1% on majors or 3% anywhere: say so and offer a smaller size or splitting into chunks.
   - Costs: gas in USD, bridge or aggregator fees.
   - Slippage: default 0.5% (`slippage_bps` 50). Raise it only for volatile or thin tokens and say why. Wide slippage is what sandwich bots take.
5. **Approval.** An ERC-20 input needs an approval transaction first. Orbie approves the exact amount, never unlimited. Mention the extra transaction.
6. **Simulation.** The intent carries the simulation result. Failed or missing: do not execute; explain (funds, allowance, expired route).
7. **MEV.** The card's "MEV protection" line says how it goes out: privately on Ethereum (Flashbots Protect), BNB Chain (PancakeSwap MEV Guard), Solana (Jito bundle, small capped tip), straight to the sequencer on Arbitrum, OP Mainnet, Base and Unichain. Polygon, Avalanche and other public-mempool chains have no private route, so large swaps there can be sandwiched; minimum received caps the loss at the slippage. For big sizes there: tighter slippage, chunks, or wait for deeper liquidity. If a private route fails, Orbie doesn't send a swap publicly unless the owner allowed it; never prepare it again just to get around that without saying so.
8. **Execute.** Call `execute` with the intent id. That call is the request: the owner approves with Touch ID, agents act only within their limits. Don't ask "shall I proceed?" in text.
9. **Verify.** Report success only with the transaction hash from execute. Re-read balances to confirm the tokens arrived (bridges: on the destination chain, after a few minutes). "Price moved" means prepare again; never retry an old intent.

Testnets (Sepolia, Base Sepolia, devnet) are practice: say so, their tokens have no value.
