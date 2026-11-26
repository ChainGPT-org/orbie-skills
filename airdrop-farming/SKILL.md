---
name: airdrop-farming
description: Decide whether a points program, testnet or "farm the X airdrop" plan is worth it, and run it with one honest wallet: cost/benefit against an uncertain, dilutable reward, protocol risk, the program's sybil rules, wallet hygiene and safe claims. Never splits funds across wallets to look like many users. Use when the user or an agent mentions points, airdrops, farming, testnets, eligibility, or claiming a drop.
---

# Airdrop farming

Points and airdrops are an uncertain reward for real costs and real protocol risk. Orbie farms with **one wallet per real user, used honestly**. Thresholds marked *(default)* are Orbie's judgment, not a rule from a source: say so when you quote them.

## Hard rule: no sybil farming
- Never create, fund or coordinate several wallets to look like several users, and never fan out from one funding wallet to "spread" activity. That is a sybil attack on the program's rules, and Orbie refuses to plan, prepare or run it, whoever asks.
- It also fails: programs cluster wallets by first funder, sweeps back to one address, same-block timing and identical amounts. Hop excluded 10,253 of 43,058 initially eligible addresses as sybils [S1]; Arbitrum's published method links funder and sweep transfers into graphs and flags clusters of more than 20 addresses [S2]; LayerZero flagged 803,093 addresses and paid self-reported sybils 15% of their allocation, redistributing the rest [S3][S4].
- An agent wallet is the owner's wallet, not another user. Agents never farm the same program from several Orbie wallets.

## 1. Is it worth it?
Write the cost side first; the reward is a guess.
- **Costs:** gas and bridge fees for every action (`dex_compare`, `bridge_risk` quotes), swap slippage both ways, and the capital's opportunity cost (what `mkt_yields` pays on the same money over the same months).
- **Protocol risk:** deposits sit in new, often unaudited contracts. Run `protocol-risk-review` and `protocol_info` (TVL, audits, hacks) first. Never put more in than you'd accept losing to an exploit.
- **Reward:** unknown in size, timing and terms. Points are not tokens: programs change rules, dilute with more participants, add vesting or cancel. Value the expected drop at a heavy discount; without published terms assume zero *(default)*.
- Go ahead only when you'd use the protocol anyway, or the costs are small enough to lose entirely *(default: total farming costs ≤ 1% of the wallet per program)*.

## 2. Read the program's rules
- From the project's own docs or site only, not a reply, a DM or a "checker" link.
- Note what counts (volume, days active, liquidity duration, referrals), what disqualifies (wash trading, self-referral, multiple wallets, bot patterns), and whether the rules allow automation. If they forbid bots, an agent may not farm it.
- No published rules = no farming plan, just ordinary use.

## 3. Use it like a real user
- Real activity you'd do anyway: a swap you want, a deposit you'd hold, liquidity you'd provide. No round-trip wash trades to inflate volume: they cost fees, break most programs' rules, and are often detected.
- Size with `position-sizing`; the points are not a reason to exceed it.
- Agents: a schedule (`trigger_create`, a cadence) is fine for real, rule-abiding activity within limits; the review card and limits apply to every action.

## 4. Wallet hygiene
- Farm from a wallet that holds only what the program needs, not the owner's main funds (an Orbie agent wallet, funded from the master, is a good fit).
- Approvals: exact amounts, revoke when done (`safety_list_approvals`, `safety_prepare_revoke`).
- Check every new contract and token with `safety_token_risk` and `address_profile` before interacting.

## 5. Claims: where drainers live
- Claim only through a link on the project's own domain, checked against its official X account or docs. Search results, ads, DMs and "claim now" replies are the usual phishing.
- A claim that asks for a token approval, `setApprovalForAll`, a Permit/Permit2 signature or a "verify" signature is a drainer *(default: refuse)*. A real claim is a call that sends tokens to you.
- Unsolicited tokens that appeared in the wallet: don't interact (see `token-due-diligence`).
- Drained or tricked? Follow `wallet-incident-response` at once.

Orbie's own checks: `rewards_inbox` and `rewards_prepare_claim` claim only from pinned contracts, and `points_tracker` shows official figures, never a value.

## 6. Report
Program, rules link, costs so far (gas, fees, slippage), capital and how long it's locked, protocol risk verdict, what remains uncertain about the reward. Never present expected airdrop value as income.

## Sources
- [S1] Hop sybil exclusion (coverage): https://decrypt.co/143231/hop-protocols-sybil-hunter-payout-unveils-powerful-new-airdrop-tool ; Hop's announcement: https://hop.mirror.xyz/AI5fOUR0X_l0mktShDOx3mwr-hsB24gp8GvTWtS-MBc
- [S2] Arbitrum Foundation, sybil detection method (README; no licence, method only): https://github.com/ArbitrumFoundation/sybil-detection ; "Fighting Sybils in Airdrops": https://arxiv.org/pdf/2209.04603
- [S3] LayerZero's preliminary sybil list (coverage): https://thedefiant.io/news/defi/layerzero-labs-identifies-800-000-potential-sybil-addresses-in-public-list
- [S4] LayerZero self-reporting at 15% (coverage): https://www.theblock.co/news/ecosystems/2024-05-03-layerzero-offers-sybil-farmers-chance-to-self-report-in-the-name-of-durability-292502
