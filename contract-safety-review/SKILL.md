---
name: contract-safety-review
description: Safety review for writing, deploying or calling smart contracts: OpenZeppelin building blocks, Foundry tests, Slither, weird ERC-20 behaviors and deployment steps. Use when writing or auditing Solidity (contract_compile, contract_prepare_deploy), before calling an unfamiliar contract (contract_prepare_call), or when a contract will hold funds, accept arbitrary tokens or use flash loans.
---

# Contract safety review

## Writing
- Build on OpenZeppelin (`npm i @openzeppelin/contracts` in the workspace): ERC20, SafeERC20, Ownable2Step or AccessControl, ReentrancyGuard, Pausable. Don't hand-roll them. Pin the compiler (`pragma solidity 0.8.x;`, exact).
- Checks-effects-interactions; `nonReentrant` on functions that move value; pull payments over push.
- Every admin function access-controlled. Who is the owner, can it become a multisig, is there a timelock? Never authorize with `tx.origin`.
- Math: 0.8 reverts on overflow, but watch division before multiplication, rounding direction (round against the caller) and decimals (6 vs 18).
- Prices: never a DEX spot price as an oracle (flash-loan manipulable). Chainlink with staleness checks, or a TWAP.
- External calls: check return values; `delegatecall` only into trusted code; upgradeable proxies need protected initializers and a stable storage layout.

## Weird ERC-20s (handle them all when accepting arbitrary tokens)
- No return value (USDT): use SafeERC20.
- Fee-on-transfer and rebasing tokens: measure the balance before and after; don't trust `amount`.
- Approve races and tokens that need approve(0) first: `forceApprove`.
- Blocklists and pauses (USDC, USDT): a transfer can revert; one failing user must not brick a loop over all users.
- Non-18 decimals, huge supplies, ERC-777 hooks (reentrancy), tokens with two addresses, flash-mintable tokens, and `permit` that silently does nothing (WETH's fallback).

## Testing (Foundry and Slither come with the owner's developer profile)
- The tools are on PATH when the owner turned on Settings → Developer → Developer tools. Never install them yourself (foundryup, curl | sh, pip install). If `forge` isn't found, ask the owner.
- Foundry: `forge init` (dependencies arrive as plain folders, no git repository); unit tests for every function including reverts (`vm.expectRevert`); fuzz tests for math; invariant tests (totals conserved, no free mint); fork tests against real state for integrations (`--fork-url`).
- Slither: `slither .`. Fix or explain every high and medium finding in writing; static analysis is noisy, so check each against the code.
- `contract_compile`: read every warning.

## AI second opinion (advisory only)
- Before deploying, or before calling a contract whose source you haven't read, offer `cgpt_audit`: ChainGPT's Smart Contract Auditor on the source, a workspace `.sol` file, or a deployed address (verified source from Sourcify). It uses plan credits.
- Call it an "AI second opinion", never an audit. It's advisory: it never approves anything, never replaces Foundry tests, Slither or a human review, and a good score is not a reason to skip any step here. Its findings are leads to check in the code, not facts.
- An address without verified source is refused: treat the contract as hostile and don't call or approve it.
- The opinion shows on that contract's deploy or call review card as an advisory line with its time. Code changed since then isn't covered: run it again.

## Deploying
1. Testnet first (Sepolia or Base Sepolia): `contract_prepare_deploy` → `execute`, then exercise every function with `contract_read` and `contract_prepare_call`.
2. Verification is automatic: Orbie publishes the source on Sourcify (Etherscan and Blockscout when set up) after the deploy confirms. Check `contract_deployments`; if one failed, fix what it says and run `contract_verify`.
3. Mainnet only on the owner's explicit go. Start with caps (max deposit, pausable) and move ownership to a multisig if it will hold real value.

## Calling someone else's contract
- Verified source? Read the function you're calling and what it can do with your approvals (`transferFrom` from you). Proxy = the code can change; check who controls upgrades.
- The prepared intent is simulated: unexplained token outflows mean stop.
- Approve exact amounts only, and offer to revoke (approve 0 via `contract_prepare_call`) when done.

Report findings by severity (critical, high, medium, low, info), each with location, impact and fix, plus what was tested and what wasn't.
