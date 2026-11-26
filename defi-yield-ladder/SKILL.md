---
name: defi-yield-ladder
description: Rank yields by what can break them, not by APY: place each candidate on a tier ladder (T0 staking to T4 farms) with DefiLlama's outlier, history and volatility fields and protocol_info checks, split a yield budget across tiers with caps, and name the risk behind any APY far above its peers. Use for "best yield", parking stables or ETH, comparing vaults, lending markets, Pendle or farms, or before moving a yield position.
---

# DeFi yield ladder

APY is the price of a risk. Find the risk first, then compare APYs within the same tier. Thresholds marked *(default)* are Orbie's judgment; DefiLlama's fields are sourced.

## 1. Candidates
`mkt_yields` (any chain) or `vault_find` (single-asset pools on Orbie's networks), filtered by asset, chain and a TVL floor. Then the protocol readers for live numbers: `aave_markets`, `spark_savings_info`, `morpho_markets`, `pendle_markets`, `ethena_info`, `stake_lido_info`, `vault_info`.

Each pool carries DefiLlama's statistics [DefiLlama yield-server]:
- `history.days`, `history.meanApyPct` (geometric mean APY over the pool's life), `history.volatilityPct` (annualized volatility of its daily return);
- `outlier`: the long-run APY or its volatility is outside the normal range across all pools (median ± 1.5 IQR). Excluded by default; `outliers: "only"` lists them when you need to know why an APY is extreme;
- `apyMean30d` and `apyChangePct` (7 and 30 days), `rewardSharePct` (rewards depend on emissions and the reward token's price);
- `prediction`: DefiLlama's model guess whether the APY holds or falls over 4 weeks, confidence 1–3. Not calibrated: context, never a reason on its own;
- `flags` and `tierHint`: Orbie's default reading of the above.

## 2. Tier each candidate *(default ladder)*
- **T0** native staking and liquid staking (Lido, Rocket Pool, cbETH, Jito…): validator and LST-depeg risk.
- **T1** blue-chip lending supply (Aave v3, Compound v3, SparkLend): smart-contract, oracle and bad-debt risk; withdrawals can wait at full utilization.
- **T2** curated vaults (Morpho), fixed-rate PTs (Pendle), savings rates (sUSDS), Fluid, Kamino: curator choices, market oracles and LLTVs fixed at creation are the depositor's risk [Morpho docs]; PTs pay fixed only if held to maturity.
- **T3** synthetic yield (sUSDe: funding can turn negative), restaking and LRTs, HLP, concentrated LP.
- **T4** incentive farms, new protocols, points, and anything `tierHint` pushes there: rewards ≥ 50% of APY, under 30 days of history, an outlier, or today's APY over 2× its 30-day mean.
`tierHint` is a starting point. Confirm the tier with `protocol_info` (category, age, TVL trend, audits, hacks, red flags) for every protocol you'd use, and run protocol-risk-review for anything not on Orbie's list. A protocol-risk-review rating of HIGH moves it to T4; CRITICAL removes it.

## 3. Read every yield
- Base vs reward APY: count only base APY as durable; rewards are paid in a token whose price you carry.
- Today vs the 30-day and lifetime mean, and the volatility: a spike decays; a high-volatility pool is not a savings rate.
- **APY far above its peers in the same tier (e.g. 2× the tier median *(default)*): name the risk you're paid for** (leverage loop, thin collateral, a young curator, a depeg-prone asset, emissions) before depositing, or don't deposit.
- LP yield is not income until fees beat impermanent loss and LVR.
- Liquidity out: withdrawal queues (Lido, sUSDe cooldown), utilization on lenders, PT exit before maturity at market price.

## 4. Build the ladder *(defaults)*
Caps per tier as a share of the yield budget: T0 up to 100%, T1 up to 60%, T2 up to 40%, T3 up to 20%, T4 up to 5%; and no single protocol above 40% of the budget. Same asset first (stables in stables, ETH in ETH); a different asset is a trade, not a yield.
Present it as a table: tier, protocol and pool, base / reward APY, 30-day mean, history and volatility, TVL, the main risk in a few words, the amount, and the sources with their time.

## 5. Act and watch
Deposit with the protocol's own `*_prepare_*` tool or `vault_prepare_deposit` → review card → `execute`. Then watch: `trigger_create` on the asset's price or peg where it applies; re-run `mkt_yields` on the pools monthly and step down a tier when a pool turns outlier or its APY collapses.

Sources: DefiLlama yields API and its enrichment code (github.com/DefiLlama/yield-server, src/handlers/triggerEnrichment.js); Morpho risk docs (docs.morpho.org/learn/resources/risks); protocol_info via DefiLlama.
