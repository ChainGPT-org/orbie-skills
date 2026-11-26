---
name: protocol-risk-review
description: Rate a DeFi protocol's risk before depositing, lending, staking, farming or bridging into it: red-flag triage from protocol_info (DefiLlama TVL, audits, oracles, hacks), governance and admin keys, oracle, economic design, contracts, bridges and operations, a fixed aggregation rule, and a comparison with known hack patterns. Use before a first deposit into any protocol, vault, pool or farm not on Orbie's list, when a yield looks unusually high, or when the user asks whether a protocol is safe.
---

# Protocol risk review

Output: one overall rating (LOW / MEDIUM / HIGH / CRITICAL) with the evidence behind it and what you could not verify. It is a risk review, not an audit. Never call a protocol safe.

## 0. Triage (always, before anything else)
`protocol_info` with the DefiLlama slug or name (pick from `candidates` if it returns several). Read `redFlags`, `tvlChangePct`, `peakTvl`, `ageDays`, `audits`, `oracles`, `hacks`, `chains`.
Any of these → triage **CRITICAL**: stop and tell the user before a deeper look:
- TVL $0, or down more than 50% in 30 days;
- no audits listed (`audits.defillamaCode` "0" and no links);
- under 6 months old with more than $50M TVL;
- anonymous team with no track record;
- closed-source contracts (unverified on the explorer counts);
- `safety_token_risk` on its token: honeypot, hidden owner, owner can change balances.
Also note (not critical alone): TVL down > 30% in 90 days, a hack in the last 2 years, DefiLlama warnings, `forkedFromIds` (a fork inherits its parent's bugs).

## 1. Rate each sub-category (exactly one level each; no "low-medium")
**A. Smart contract**
- Audits: who, when, scope; recent audits on the deployed code count more than old ones. A listed audit is not a clean bill.
- Upgradeability: proxy? read the EIP-1967 admin and implementation slots with `contract_read`; upgraded after the last audit = higher.
- Oracle: provider(s) (`oracles`), fallback, TWAP vs spot; a spot DEX price or a single source for thin collateral = HIGH.
- Economic design: liquidation path, bad-debt handling, insurance fund vs TVL, withdrawal limits; what pays the yield.
- Collateral (lenders): the worst collateral accepted, not the average (LRTs, bridged and synthetic tokens), and whether each has a binding cap.
- Cross-chain: bridges it depends on for minting or reserves; counts double when deployed on 5+ chains.
**B. Off-chain**
- Governance and admin (counts double): what the admin can do (pause, upgrade, list collateral, change oracles, drain), multisig threshold (2-of-N with small N = HIGH), timelock length and who can bypass it (pause-only bypass is fine; an upgrade bypass is HIGH).
- Team and operations: public team and record, incident response, bug bounty.
- Key management: how admin keys are held (hot wallets, MPC, hardware).
**C. Track record**
- Incidents (`hacks`): what, how much, returned, fixed how; time since.
- Battle-tested: months live × TVL held. TVL stability (`tvlChangePct`, `peakTvl.belowPeakPct`). Audit recency.

## 2. Aggregate (mechanical; don't override with judgment)
1. Each dimension (A, B, C) = the highest of its sub-categories. Governance counts double in B (HIGH governance alone makes B HIGH); cross-chain counts double in A when on 5+ chains. Unknowns you could not check are listed, not rated LOW.
2. Overall: any dimension CRITICAL → **CRITICAL**; 2+ dimensions HIGH → **HIGH**; 1 HIGH, or 2+ MEDIUM → **MEDIUM**; otherwise **LOW**.

## 3. Hack-pattern check (3+ ticks in one row → name the pattern in the report)
- *Governance + oracle*: admin lists collateral or changes oracles without a timelock; low multisig threshold; anonymous signers.
- *Oracle / economic manipulation* (Mango, Euler): thin collateral, single oracle without TWAP, no circuit breaker, small insurance fund.
- *Bridge key compromise* (Ronin, Harmony): few validators, hot keys, no rotation.
- *Flash-loan governance* (Beanstalk): voting power at vote time, proposal and execution close together.
- *Reentrancy + flash loan* (Cream, bZx): rebasing or fee-on-transfer collateral, callbacks before state updates.
- *Compiler bug* (Curve/Vyper 2023): niche compiler or a version with known issues.
- *Algorithmic depeg* (UST): a stablecoin backed by its own token; redemption sells the collateral.
- *Bridge message spoofing + cascade*: a single messaging layer mints or releases tokens with no rate limit, and the token is collateral elsewhere.

## 4. Report and act
- Overall rating on one line, then a table: sub-category, rating, key fact, source (link) and verified Y/N; the hack patterns ticked; information gaps.
- Map it to size *(default)*: LOW → normal position-sizing; MEDIUM → at most half; HIGH → only a small amount the owner explicitly accepts; CRITICAL → don't deposit, and say why.
- If the user goes ahead, deposit with the protocol's own `*_prepare_*` tool or `vault_prepare_deposit` → review card → `execute`. A protocol not on Orbie's list says so on its card; that warning stays. This review never authorises skipping a card, a limit or the owner's approval.

Method adapted, and rewritten for Orbie, from truenorth-lj/crypto-project-security-skill (MIT, © 2026 LJ Li); see NOTICE.md in this folder. Data: DefiLlama (api.llama.fi), GoPlus via `safety_token_risk`.
