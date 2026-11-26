---
name: wallet-incident-response
description: What to do in the first minutes after a wallet is drained or at risk: an unexpected outflow, a bad approval, a leaked key or recovery phrase, a permit/Permit2/approve-for-all signature the owner regrets, a phishing site, or an agent or connected site acting oddly. Triage, Orbie's lockdown, revokes and sweeps, what never to do, SEAL 911 and the follow-up checklist.
---

# Wallet incident response

Speed matters more than a full diagnosis. Stop the bleeding first, then work out what happened.

## First minute
1. Stop everything that can act: run `safety_lockdown` (master conversations; the owner approves it and confirms with Touch ID), or the owner presses Lockdown in Wallet → Approvals. It pauses every agent and stops their runs before it reads anything, so no agent trades, signs or runs on a schedule while you work. It also stops live grids, and returns a venue card per wallet that cancels its open orders on Hyperliquid, Aster, GMX, exchanges, Polymarket and CoW (with `flatten: true` it also closes positions, reduce-only). If a leaked key could trade on a venue, run those cards early.
2. Close the suspicious site in the browser and don't sign anything else on it.
3. Note what happened, in the owner's words: which wallet, which network, which token, roughly when, and the transaction hash of any outflow (from Activity or the explorer).

## Triage: which kind of incident
Look at what left the wallet and how (the explorer, `safety_list_approvals`).
- **Bad approval** (an exploited contract, or a drainer you approved): one token, or the tokens you approved, leave through `transferFrom` by the spender, often hours or months after you approved. The key is fine. Revoke the approval; the wallet can stay in use.
- **Malicious signature** (EIP-2612 permit, Permit2 PermitSingle/PermitBatch/transfer, Seaport listing, approve-for-all): no transaction of yours, then the signed tokens or NFTs move. Usually one-time. Revoke the token's approval to Permit2 (that kills every Permit2 allowance for it) and any approve-for-all; an unused signature can sometimes be cancelled on revoke.cash's Signatures page, but the window is short. The wallet can stay in use afterwards.
- **Leaked key or recovery phrase**: native coin and many tokens leave, often on several networks, from transactions *you* "sent" (signed with your key), or funds you add disappear within seconds (a sweeper bot). The wallet is lost for good. Move what's left out now; revoking is secondary because the attacker can approve again. If the phrase leaked, every Orbie wallet (master and agents, all from the same phrase) is exposed.
- **Phishing site** with nothing signed yet: no loss. Leave the site; check approvals anyway in case something was signed earlier.
- Unsure? Treat it as the worse case (leaked key) for the wallet that lost funds.

## Steps with Orbie
**Bad approval or signature**
1. `safety_list_approvals` for the wallet (all networks). Rows flagged *Exploited* (Revoke.cash's exploit list) or high risk come first.
2. `safety_prepare_revoke` for each, then `execute`; or `safety_lockdown` without a sweep to get one revoke card per wallet and network for everything at once.
3. Approvals Orbie can't see (NFT approve-for-all, other apps' spenders): check revoke.cash for the address and revoke there from the wallet's own browser session if needed.

**Leaked key or phrase**
1. `safety_lockdown` with `sweep_from` = the compromised wallet and `sweep_to`:
   - a hardware wallet's account name if the *phrase* may have leaked (it doesn't share Orbie's phrase); a new phrase outside Orbie also works;
   - `master` or `new` only if a single key leaked and the phrase didn't (they come from the same phrase).
2. Execute the sweep cards immediately, biggest value first (the cards are already in that order). Each card moves the tokens, then the native coin minus what the fees need. Each amount is read again just before it goes, so a bot that got there first only costs what it took; a step that fails doesn't stop the others. Read the result: it names any step that failed, had nothing left or has no answer yet, and says when less moved than the card showed. Tell the owner, and check what's still in the wallet.
3. Bots usually watch a leaked wallet. If there's no gas left, sending gas to it will likely be taken before the token transfer lands. The real fix is a private bundle (gas and transfers in one block), which this wallet doesn't do: contact SEAL 911 for help with large amounts.
4. Then revoke on the other wallets (the lockdown cards), and stop using the leaked wallet for good. Funds that arrive later (airdrops, vesting, staking rewards) will be swept by the attacker too.
5. A phrase leak usually means malware or a phrase typed or photographed somewhere. Assume this Mac is suspect until checked; don't import the phrase anywhere again.

**Agent or site behaving oddly**: lockdown first (pauses everything), then read the agent's activity log and the review cards it prepared before resuming anything.

## What not to do
- Don't contact, threaten or negotiate with the attacker, and don't send them anything "to unlock" funds. Bounty negotiations are for protocols with lawyers, not individual wallets.
- Never pay a "recovery service", and ignore anyone who DMs offering help: real responders don't cold-message victims, and anyone promising to recover stolen funds for a fee is almost certainly running a second scam.
- Never share a recovery phrase or private key with anyone, including people claiming to be SEAL 911, an exchange or Orbie. SEAL says it never asks for them or for payment.
- Never run code, scripts or repos that claim to recover or "rescue" funds, and never sign a transaction a helper sends you.
- Don't send fresh funds into a wallet whose key leaked.
- Don't resume agents until you know how the incident happened.

## Getting help and reporting
- Active drain or a sweeper bot: SEAL 911, free and staffed around the clock, through the Telegram bot @seal_911_bot (securityalliance.org/seal-911). Have ready: the affected address(es), network(s), the attacker's address, the transaction hashes, and what you've already done.
- Stolen funds reaching an exchange: report to that exchange's support with the transaction hashes; they can sometimes freeze deposits.
- File a police report (in the US also the FBI's IC3). It rarely recovers funds but creates the record exchanges and insurers ask for.
- A newly exploited contract: report it to Revoke.cash's exploit list so others get warned.

## After the incident
- Every wallet, every network: approvals reviewed (`safety_list_approvals`, plus revoke.cash for NFTs and other apps).
- Exchange API keys rotated (Settings → Exchanges), connected sites disconnected, app passwords changed.
- Agents reviewed one by one before resuming: job, limits, recent cards, any site it used.
- Note what happened and when, with the hashes, for reports and taxes.
- If the phrase leaked: new phrase (ideally a hardware wallet), funds moved, and the old phrase never used again.
- Next time: exact-amount approvals only, revoke what you no longer use, and read permit and Permit2 cards as carefully as transactions.

Sources (summarised): Revoke.cash, "What to do when you get scammed" (revoke.cash/learn/security/what-to-do-when-scammed); SEAL 911 (securityalliance.org/seal-911 and /sos); SEAL incident-response contacts (frameworks.securityalliance.org/incident-management/incident-response-template/contacts); Revoke.cash approval exploit list (github.com/RevokeCash/approval-exploit-list, MIT).
