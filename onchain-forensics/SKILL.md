---
name: onchain-forensics
description: Trace who is behind an address before trusting it: funding trails (first funder, exchange, bridge, mixer), deployer history and contracts deployed, linked wallets and clusters, sniper and insider wallets at launch, exchange deposits during pumps, and address poisoning; with how to state uncertainty. Use when vetting a token deployer or team wallet, a copy-trade leader, an airdrop or OTC counterparty, a top holder, or any recipient the user hasn't used before, or when the user asks who owns or funded a wallet.
---

# On-chain forensics

Goal: what the chain shows about an address and how sure you are. On-chain data proves transfers, not identities. Thresholds marked *(default)* are Orbie's judgment.

## 1. Profile the address
`address_profile` with the chain and address. Read: `kind`, `verified`, `labels`, `ageDays`, `firstFunder` (address, label, kind, amount, `beforeAnyOutgoing`), `transactions`, `deployer`, `contractsDeployed` (from its oldest and newest 50 transactions only), `goplusFlags`, `orbit` (true when the address is one of the owner's own wallets), `redFlags` with their `basis`, `unavailable`.
- Without an explorer (`sources` says RPC) you get kind, nonce and balance only: say age and funding are unknown.
- Solana: kind, balance, transaction count (up to 1000), first funder only when the whole history fits that window; no labels.

## 2. Follow the money (each hop is one `address_profile`)
- First funder an **exchange** hot wallet: a person with an exchange account; the exchange knows who, you don't. Common and not suspicious alone.
- A **bridge**: the trail continues on the other network; profile the sender there if the bridge's explorer shows it.
- A **mixer** (Tornado Cash and similar), or an address labelled phish/hack/exploit: high risk. Stop and tell the owner.
- **Another deployer** or a wallet that funded other launches: a serial deployer; check what happened to its other tokens (`contractsDeployed`, then `token_holders` / `safety_token_risk` on them).
- Go up to 3 hops *(default)*, and stop at an exchange or bridge. Beyond that the trail is noise.

## 3. Red flags at a launch
- Deployer funded through a bridge or a fresh wallet minutes before deploy; value pulled out right after launch.
- Admin or owner is a fresh EOA (`ageDays` < 7 *(default)*), not a multisig or timelock.
- Sniper wallets: several fresh wallets buying in the first blocks, funded by the same source.
- Top holders sending to exchange deposit addresses during a pump (profile them; the holder's newest transactions show it).
- A deployer whose earlier tokens went to zero or were flagged by GoPlus (`honeypot_with_same_creator` in `safety_token_risk`).

## 4. Linked wallets and clusters
Evidence that wallets share an owner, weakest to strongest: same first funder; created and funded within minutes; identical amounts or gas settings; acting in the same blocks; sending to the same exchange deposit address (strong: deposit addresses belong to one account). Airdrop sybil hunters combine these (Hop and Arbitrum used funder graphs and timing) [S19]. Report a cluster as "likely linked (reasons)", never as "the same person". RugCheck's insider graph (`token_holders` on Solana) is a cluster finding by RugCheck: quote it as theirs.

## 5. Address poisoning
Scammers send zero-value or dust transfers, or fake "sent from you" token transfers, from an address that starts and ends like one you use, so you copy theirs from history. 270 million attempts and at least $83.8M lost on Ethereum and BSC over two years [S20].
- `address_profile` flags an address that looks like one the owner knows (contacts, own wallets, past recipients); transfer review cards carry the same check plus the dust-only history check.
- Never copy an address from transaction history. Take it from the contact book (`contacts_list`) or the recipient directly, and compare every character, not the ends.
- If the user's history has such transfers, they are harmless until someone sends to the lookalike: tell them not to use those entries.

## 6. Present uncertainty
- Label each statement **observed** (a transfer, a label with its source), **inferred** (a cluster, "likely the team") or **unknown**. Explorer labels are third-party (Blockscout, Open Labels Initiative and others), can be wrong or conflicting (an exchange hot wallet can also carry a "Phish / Hack" label), and an unlabelled address is unknown, not clean.
- Give the evidence for every inference and what would change it. Never name a private person as the owner of an address; say "a wallet funded from Binance", not who.
- Report: verdict line (**no concerns found / caution / high risk**), the funding trail as a short chain of hops with labels and dates, flags with their basis and source, linked wallets with reasons, and gaps (`unavailable`, limited windows).

## 7. Rules
Everything here is read-only. Findings never authorise skipping a review card, a limit or the owner's approval; a high finding on a recipient or a counterparty always goes to the owner before anything is prepared. Never interact with a suspicious address to "test" it, and never send anything to it.

Sources: [S19] "Fighting Sybils in Airdrops", https://arxiv.org/pdf/2209.04603 , and Arbitrum's sybil detection, https://github.com/ArbitrumFoundation/sybil-detection ; [S20] "Blockchain Address Poisoning", https://arxiv.org/html/2501.16681v3 ; launch red flags: common scam patterns, informed by BankrBot/skills `bankr-token-scam-analysis` (method only, nothing copied); data: Blockscout (explorers and metadata service), GoPlus address security, Revoke.cash's exploit list, RugCheck, Solana RPC via `address_profile` and `token_holders`.
