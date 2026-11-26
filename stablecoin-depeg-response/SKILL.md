---
name: stablecoin-depeg-response
description: What to do when a stablecoin trades off its peg, or before holding stables as collateral, debt, LP or vault deposits: classify the stablecoin (fiat-redeemable, crypto-collateralised, synthetic like USDe, algorithmic), read the signals (prices on several venues, Curve/Uniswap pool imbalance, redemption status, issuer news, lenders freezing it), find every position exposed, and follow the playbook for its type (wait, exit or hedge). Use when a price alert on a stable fires, the user asks whether a stable is safe or depegging, or before parking large amounts in one stable.
---

# Stablecoin depeg response

Speed matters, but selling a redeemable stable into a panic locks in the loss. Classify first, then act. Thresholds marked *(default)* are Orbie's judgment.

## 1. Classify the stablecoin
- **Fiat-redeemable** (USDC, USDT, PYUSD, FDUSD): an issuer redeems at $1 from cash and bills. Risk = the reserves and the redemption channel. USDC fell to about $0.88 in March 2023 while $3.3B of its reserves sat at the failed SVB, and returned to $1 once redemption at par was assured [Kaiko; Federal Reserve note].
- **Crypto-collateralised** (DAI/USDS, LUSD, crvUSD, GHO): over-collateralised debt; only as good as its collateral. DAI/USDS hold USDC in the peg stability module, so a USDC depeg passes through.
- **Synthetic / delta-neutral** (USDe, sUSDe): backed by spot plus short perps. Risks: negative funding (the reserve fund covers it only up to its size), exchange or custodian failure, liquidation of the hedge, the backing assets themselves [Ethena docs]. sUSDe unstaking has a cooldown.
- **Algorithmic** (backed by its own token): a run sells the collateral that backs it. UST did not come back. Treat any depeg as terminal.
- Bridged copies (USDC.e, multichain USDT) add the bridge's risk on top: see the bridge before the issuer.

## 2. Read the signals (quote each with its source and time)
- Price on several venues: `mkt_price` (DefiLlama), `cex_ticker` on two exchanges, and a `dex_compare` quote at your size, both directions. One venue off = a liquidity hole; all venues off = the market's view.
- Pool imbalance: Curve 3pool and the main stable pools (`contract_read` on `balances(i)`); one coin above about 50–60% of a balanced pool *(default)* means holders are dumping it (in March 2023 3pool went to about 2% USDT as holders fled USDC [Kaiko]).
- Redemption: is mint/redeem open? Issuer status page and announcements (`browser_open` the issuer's own site, not social posts), Sky PSM (`spark_savings_info`), Ethena (`ethena_info`).
- Lenders: `aave_markets` shows a reserve `frozen` or `paused`, and the oracle price the market uses for it (some feeds hold $1 while the market trades below).
- Watch it: `trigger_create` `price_below` at 0.995 and 0.99 *(default)* on the stable, `notify`.

## 3. Find every exposure
Start with `stablecoin_exposure`: every stable held, supplied, borrowed, in LP or vaults, on exchanges and as Hyperliquid margin, by issuer, type, chain and venue, with what it couldn't read. Then confirm the large ones in their own tools: wallet balances (`get_portfolio`, `sol_portfolio`); lending collateral and debt (`aave_positions`, `spark_positions`, `comet_positions`, `morpho_positions`, `kamino_positions`); LP (`dex_lp_positions`); vaults (`vault_positions`); `pendle_positions`, `ethena_positions`; exchange balances (`cex_balances`).
- Collateral in the depegging stable: the health factor falls with its oracle price → see liquidation-defense now.
- Debt in it: the debt gets cheaper, unless the oracle holds $1; no rush.
- LP with it: the pool sells you more of the falling coin; that is where the loss concentrates.

## 4. Act by size of the move *(default thresholds)*
- **< 0.5% off**: noise. No trade.
- **0.5–1% for over an hour**: watch; read the issuer and redemption status; no new deposits into it.
- Watch it: `trigger_create` with `peg_off` (vs $1, or with a venue vs the lender's oracle) and, for supply you lend out, `liquidity_below_supply`. Withdrawing lent supply while liquidity exists is `lend_prepare_exit`; it moves no depegged asset through a swap.
- **> 1%**: alert the owner; check every exposure above; withdraw LP that holds it; restore health factors where it is collateral.
- **> 3%**: act by type:
  - *Fiat-redeemable, redemption working, no insolvency news*: don't sell into the hole. Hold, or at most cut to your concentration cap; avoid leverage on it.
  - *Fiat-redeemable, redemption suspended or reserves impaired*: exit in parts to a stable of another issuer and type.
  - *Crypto-collateralised*: follow its collateral. If the collateral is the problem, exit; if it is passing through a USDC move, treat it like USDC.
  - *Synthetic*: exit or reduce; unstaking waits out the cooldown (`ethena_prepare_cooldown`), selling now costs the discount (`ethena_prepare_sell`): show both.
  - *Algorithmic*: exit now, in parts; don't wait for recovery.
- Exits go through `prepare_swap` / `dex_prepare_swap` / `sol_prepare_swap` and the swap-execution checklist. In a depeg, liquidity is thin: split size, check price impact both ways, keep slippage tight and let minimum received refuse a bad fill. Diversify into a stable of a *different* type and issuer; swapping USDC for DAI during a USDC depeg is not an exit.
- Hedging: there is rarely a perp on a stable. The hedge is diversification across issuers and types, and keeping leverage off pegged collateral.

## 5. Rules
- Every step is a `*_prepare_*` → review card → `execute`; the owner approves, an agent acts only within its limits and loss protection. Never split a trade to slip under a limit, never retry an old intent, and report only hashes from `execute`.
- Report: the stable and its type, prices by venue with times, pool balance, redemption status, each exposure with its USD amount, what you did or recommend, and what you could not verify.

Sources: Kaiko, USDC depeg, https://research.kaiko.com/insights/usdc-depeg-bots ; Federal Reserve, SVB and stablecoins (2025-12-17), https://www.federalreserve.gov/econres/notes/feds-notes/in-the-shadow-of-bank-run-lessons-from-the-silicon-valley-bank-failure-and-its-impact-on-stablecoins-20251217.html ; Ethena risks, https://docs.ethena.fi/solution-overview/risks .
