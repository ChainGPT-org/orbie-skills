---
name: bridge-risk
description: Judge a cross-chain move before it is sent: prefer canonical and issuer routes, read the bridge LI.FI picked (L2BEAT type and trust model, DefiLlama age, audits and hacks), the destination chain's L2BEAT stage, split large amounts, and track arrival without resending. Use for any bridge (prepare_swap with to_chain, CCTP into Hyperliquid) above a small amount, when the user asks which bridge is safe, or when a bridge review card shows a bridge finding.
---

# Bridge risk

A bridge adds risk the two chains don't have: the bridge's own validators, keys and contracts. Thresholds marked *(default)* are Orbie's judgment; the rest is sourced.

## 1. Pick the route
- **Native or issuer routes first.** Rollup canonical bridges (verified by the rollup's own proofs), Circle CCTP for USDC (burn on one chain, native USDC minted on the other), Hyperliquid's own Arbitrum bridge for USDC into Hyperliquid. They deliver the native asset, not a wrapped copy.
- **Then intent / liquidity bridges** (Across, Relay, Mayan, Stargate…): a solver pays you on the destination from its own funds; your exposure is the minutes in flight plus the contracts you approve.
- **Avoid lock-and-mint wrapped tokens** you'd have to hold: never keep a bridged copy long-term when a native version exists *(default)*. A wrapped token is worth what the bridge holds behind it.
- For third-party bridges prefer proof-based verification over committees and multisigs, and upgrade delays of at least 7 days (L2BEAT uses 7 and 30 days as thresholds) [L2BEAT risk framework].

## 2. Read the card (or `bridge_risk` first)
`prepare_swap` with `to_chain` puts on the review card: **Bridge** (the tool LI.FI picked, L2BEAT's type and token-transfer type), **Bridge trust** (how transfers are verified, paraphrased from L2BEAT), **Bridge security** for canonical bridges (stage, state validation, exit window), **Bridge record** (DefiLlama: tracked since, audit links, hacks), **Destination chain** (its L2BEAT stage) and **Estimated time** (LI.FI's estimate). `bridge_risk` gives the same for any LI.FI bridge key or name, before preparing. It is a bundled snapshot (date in the result); open the L2BEAT and DefiLlama links for anything that decides a large transfer.

Findings *(defaults)*:
- exploited in the last 90 days → **high** (the owner decides; agents need the owner); in the last 2 years → medium;
- tracked by DefiLlama for under 180 days → medium (young);
- no audits listed on DefiLlama → medium;
- not reviewed by L2BEAT, or no data at all → medium: say so, keep the amount small;
- may deliver a wrapped token → low.
A clean card is not a safety rating: it means none of these were found.

## 3. Size and timing *(defaults)*
- Above $10,000 (the card notes it): split across time or across two routes; send a small test first on a bridge you haven't used.
- Destination Stage 0 or no stage: fine to pass through, but think twice before parking size there; the operator can change the rules.
- A canonical rollup withdrawal back to Ethereum can take days (challenge period); the card's estimated time says so. Use an intent bridge if speed matters, and accept its trust model.
- If LI.FI returns a different bridge on a refreshed quote, Orbie asks for a new review.

## Bitcoin
- Out of Bitcoin: `btc_prepare_bridge` (BTC to a token on an EVM network or Solana). Into Bitcoin: `prepare_swap` with `to_chain: "bitcoin"`, `to_token: "BTC"`. Both deliver only to this wallet's own address.
- Bitcoin bridges are deposit-address bridges: the bridge, not the transaction, ties the deposit to your destination, so the card always says that part is unchecked and agents need the owner. Orbie refuses to sign a PSBT with any output the quote doesn't explain.
- A deposit stuck in the mempool: `btc_prepare_speed_up` (RBF from the change). Never send a second deposit.

## 4. Send and track
- `execute` the intent; Orbie tracks arrival through LI.FI's status (or Circle's for CCTP into Hyperliquid) and keeps the activity pending until it arrives, is refunded or fails.
- **Never resend on an uncertain status.** Pending can last minutes to hours; "partial" means it arrived in another token; "refunded" returns funds minus fees. Open the LI.FI link before doing anything else.
- After arrival, check the received token is the native one you expected (`get_token` on the destination).

Sources: L2BEAT interop and scaling data (l2beat.com; code and data MIT), DefiLlama protocols and hacks (api.llama.fi), LI.FI tools and status. L2BEAT Risk Rosette framework: https://forum.l2beat.com/t/the-risk-rosette-framework/292
