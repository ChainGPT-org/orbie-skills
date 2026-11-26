# Orbie skills

Trading and DeFi playbooks from [Orbie](https://orbie.chaingpt.org), ChainGPT's AI crypto wallet. Each skill is a plain-language
procedure an AI agent follows when a task matches it: how to size a position, check a token before buying, review a
lending market, defend a position from liquidation, respond to a drained wallet, and more. They are written to be
read by people too.

The skills use [Claude Code's skill format](https://docs.claude.com/en/docs/claude-code/skills): one folder per skill
with a `SKILL.md` (a short description that says when it applies, then the procedure). Every threshold is marked as
*(default)* when it's Orbie's choice or *(sourced)* when it comes from a protocol's docs or research, and each skill
lists its sources.

## Skills

| Skill | What it's for |
|---|---|
| [`airdrop-farming`](airdrop-farming/SKILL.md) | Decide whether a points program, testnet or "farm the X airdrop" plan is worth it, and run it with one honest wallet: cost/benefit against an uncertain, dilutable reward, protocol risk, the program's sybil rules, wallet hygiene and safe claims. |
| [`backtest-protocol`](backtest-protocol/SKILL.md) | Protocol for testing a trading strategy honestly before real money: no lookahead, fees and slippage, walk-forward, then paper, then small live. |
| [`bridge-risk`](bridge-risk/SKILL.md) | Judge a cross-chain move before it is sent: prefer canonical and issuer routes, read the bridge LI.FI picked (L2BEAT type and trust model, DefiLlama age, audits and hacks), the destination chain's L2BEAT stage, split large amounts, and track arrival without resending. |
| [`contract-safety-review`](contract-safety-review/SKILL.md) | Safety review for writing, deploying or calling smart contracts: OpenZeppelin building blocks, Foundry tests, Slither, weird ERC-20 behaviors and deployment steps. |
| [`copy-trading-safety`](copy-trading-safety/SKILL.md) | Vet a trader before copying or following them, pick copy targets from the Hyperliquid leaderboard, read smart-money vs crowd positioning, set safe limits for copy_follow (Solana) and hlcopy_follow (Hyperliquid), and decide when to go from paper to real. |
| [`defi-yield-ladder`](defi-yield-ladder/SKILL.md) | Rank yields by what can break them, not by APY: place each candidate on a tier ladder (T0 staking to T4 farms) with DefiLlama's outlier, history and volatility fields and protocol_info checks, split a yield budget across tiers with caps, and name the risk behind any APY far above its peers. |
| [`execution-quality`](execution-quality/SKILL.md) | Execute size well and check afterwards: compare the order with order-book depth (cex_orderbook) or quotes across DEX aggregators (dex_compare), split when it's large against ±1% depth, keep flow private where possible, then measure the fill with execution_report (quoted vs filled, vs market, fees, sandwich check). |
| [`funding-basis-carry`](funding-basis-carry/SKILL.md) | Plan, run and unwind a delta-neutral funding carry (cash-and-carry): short a perp that pays funding and hold the same amount long in spot (or a cheaper perp), entry and exit rules on multi-day realised funding averages, all four fees, borrow costs, liquidation distance on the short leg, venue risk and how to unwind. |
| [`grid-and-exits`](grid-and-exits/SKILL.md) | How to put exits on a position (stop-loss, take-profit, time limit, trailing stop), staged exits on a spot token (sell 50% at 2×, trail the rest), and how to run a grid from backtest to paper to live, with the cap approved once. |
| [`lending-risk-review`](lending-risk-review/SKILL.md) | Risk review for supplying, borrowing, looping or withdrawing on Aave (aave_* tools), Kamino, MarginFi or another lending market: health factor, liquidation price, thresholds and rate risk. |
| [`liquidation-defense`](liquidation-defense/SKILL.md) | Watch and defend borrowed or leveraged positions before they are liquidated: health-factor and margin alerts, how liquidation works on Aave, Spark, HyperLend, Compound v3, Morpho, Kamino, MarginFi, Hyperliquid, Aster and GMX, and the defense ladder (add collateral, repay with stables, deleverage, close). |
| [`lp-management`](lp-management/SKILL.md) | Open, watch and rebalance concentrated liquidity (Uniswap V3/V4, PancakeSwap V3) so fees actually beat impermanent loss and LVR: simulate the range first (fee APR from volume and in-range liquidity, IL vs holding, LVR, width table, rebalance cost), size the width from volatility, set an out-of-range alert, and rebalance only when it pays. |
| [`market-regime`](market-regime/SKILL.md) | Read the market regime (trend-up, trend-down, chop, high-vol) and crowding before sizing risk-on trades, setting agent schedules or answering "is now a good time": price vs 50/200-day averages and 30-day efficiency, 30-day realized volatility against its past year, funding and open-interest heat, then a stance (risk-on / neutral / risk-off) with a size multiplier for position sizing. |
| [`onchain-forensics`](onchain-forensics/SKILL.md) | Trace who is behind an address before trusting it: funding trails (first funder, exchange, bridge, mixer), deployer history and contracts deployed, linked wallets and clusters, sniper and insider wallets at launch, exchange deposits during pumps, and address poisoning; with how to state uncertainty. |
| [`perp-trade-plan`](perp-trade-plan/SKILL.md) | Plan a leveraged perpetual futures trade before placing it: risk, size, margin vs notional, leverage, liquidation distance and funding. |
| [`position-sizing`](position-sizing/SKILL.md) | Size any new position from a risk budget before preparing it: risk per trade, volatility targeting, capped fractional Kelly, concentration and correlation caps, and cutting size after losses, mapped to the agent's limits and loss protection. |
| [`prediction-market-risk`](prediction-market-risk/SKILL.md) | Check a Polymarket market before any order: the resolution rules and source, UMA's optimistic oracle and dispute risk, liquidity and spread at your size, time to resolution, correlated bets, and sizing as a binary bet. |
| [`protocol-risk-review`](protocol-risk-review/SKILL.md) | Rate a DeFi protocol's risk before depositing, lending, staking, farming or bridging into it: red-flag triage from protocol_info (DefiLlama TVL, audits, oracles, hacks), governance and admin keys, oracle, economic design, contracts, bridges and operations, a fixed aggregation rule, and a comparison with known hack patterns. |
| [`research`](research/SKILL.md) | Answer a crypto research question from the right source and cite it: which research connector (Messari, Dune, Token Terminal, Nansen, LunarCrush, Santiment, Arkham) or Orbie tool to use for fundamentals, onchain queries, protocol financials, smart money, social sentiment, behaviour metrics or address attribution, what each costs, and how to cite and distrust what comes back. |
| [`stablecoin-depeg-response`](stablecoin-depeg-response/SKILL.md) | What to do when a stablecoin trades off its peg, or before holding stables as collateral, debt, LP or vault deposits: classify the stablecoin (fiat-redeemable, crypto-collateralised, synthetic like USDe, algorithmic), read the signals (prices on several venues, Curve/Uniswap pool imbalance, redemption status, issuer news, lenders freezing it), find every position exposed, and follow the playbook for its type (wait, exit or hedge). |
| [`swap-execution`](swap-execution/SKILL.md) | Checklist to run before every token swap or bridge, up to execute and verification. |
| [`tax-lot-records`](tax-lot-records/SKILL.md) | Tax-ready records from what Orbie recorded and what the owner imported (Coinbase, Kraken, Binance, wallet CSVs, Form 1099-DA checks): realized and unrealized gains by FIFO, LIFO, HIFO or specific ID, income (staking, LP fees, funding, airdrops, interest), fees, derivatives P&L, and CSV exports for Koinly, CoinTracker, CoinLedger and Cryptio. |
| [`token-due-diligence`](token-due-diligence/SKILL.md) | Research a token before anyone buys, swaps into, copies or holds it. |
| [`tokenomics-review`](tokenomics-review/SKILL.md) | Review a token's supply and holder structure before holding it for more than a trade: float (circulating vs total), FDV vs market cap, emissions and unlock schedule pressure vs trading volume, insider/team/VC concentration, liquidity depth at the user's size, listing quality and revenue vs valuation. |
| [`wallet-incident-response`](wallet-incident-response/SKILL.md) | What to do in the first minutes after a wallet is drained or at risk: an unexpected outflow, a bad approval, a leaked key or recovery phrase, a permit/Permit2/approve-for-all signature the owner regrets, a phishing site, or an agent or connected site acting oddly. |

## Use them

- **In Orbie**: Customize → Skills → Import, and paste this repository's link (or a link to one skill's folder).
  Orbie shows what the skill asks for before you turn it on. The built-in copies already ship with the app.
- **In Claude Code**: copy a skill's folder into `~/.claude/skills/` (or your project's `.claude/skills/`), or add this
  repository as a plugin; `.claude-plugin/plugin.json` points at the skills.
- **Elsewhere**: the `SKILL.md` files are Markdown; any agent framework that loads instructions from files can use them.

Tool names in the procedures (`prepare_swap`, `paper_order`, `trigger_create` and so on) are Orbie's. Outside Orbie,
read them as "the tool that does this" and map them to your own.

## Not financial advice

These are risk procedures, not recommendations. They don't promise returns, they can be wrong, and markets change
faster than any playbook. Check the sources, test on paper first, and never risk money you can't afford to lose.

## Contributing

Issues and pull requests are welcome. A new or changed threshold needs a source (a protocol's docs, an audit, a paper
or measured data) or an explicit *(default)* with the reasoning.

## License

MIT, see [LICENSE](LICENSE). Three skills adapt other open-source work (MIT or Apache-2.0); their `NOTICE.md` files carry the attribution.
