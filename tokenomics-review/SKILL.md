---
name: tokenomics-review
description: Review a token's supply and holder structure before holding it for more than a trade: float (circulating vs total), FDV vs market cap, emissions and unlock schedule pressure vs trading volume, insider/team/VC concentration, liquidity depth at the user's size, listing quality and revenue vs valuation. Use when the user asks whether a token is fairly valued, about its unlocks, supply, FDV, dilution or "tokenomics", or as part of token-due-diligence for any non-major token held longer than a trade.
---

# Tokenomics review

Goal: say how much selling pressure the supply structure puts on the price, from data you can cite, and say plainly what you could not get. Thresholds marked *(default)* are Orbie's judgment; *(heuristic)* are third-party rules of thumb; neither is a law. Never call a token undervalued or a buy: this is a risk read, not advice.

## 1. Supply and valuation
- `token_holders` with the chain and address (Solana: mint). Its `market` block is CoinGecko's (`/coins/{platform}/contract/{address}`): price, market cap, FDV, 24h volume, circulating, total and max supply, `mcToFdv`. Quote it with `asOf`. If `market` is missing, fetch the same CoinGecko endpoint with WebFetch or check the project's docs; say which.
- Float = circulating ÷ total (or ÷ max where max is set) = market cap ÷ FDV. Low-float, high-FDV launches face sustained selling as supply unlocks; Binance Research flagged the pattern in 2024 [S22]. `token_holders` raises a low-float flag below 30% *(default)*.
- Max supply null means uncapped: find the inflation rate (emissions per year ÷ circulating) in the project's docs.
- Compare valuation with revenue where a protocol earns fees: DefiLlama's free fees summary (`api.llama.fi/summary/fees/{slug}?dataType=dailyFees`, or `protocol_info` for the slug first). FDV ÷ annualised fees is a yardstick, not a price target.

## 2. Unlocks and emissions (the data gap)
- Orbie has no free unlock feed: DefiLlama's emissions/unlocks API is paid, and Tokenomist/CryptoRank need keys. Get the schedule from the project's own docs or token page (`browser_open`), and say it came from there and on what date; if you can't find one, say "unlock schedule unknown" in the report. Don't estimate one.
- For each unlock in the next 90 days: date, amount, USD value at the current price, who receives it (team, investors, ecosystem, community, staking).
- Unlock pressure = unlock USD ÷ 7-day average daily volume (`mkt_candles` 1d × 8, volume × close; or CoinGecko 24h volume as a weaker stand-in): **> 2.4× crisis, 1–2.4× strain, 0.3–1× digestible, < 0.3× trivial** *(heuristic adapted from Bankr's unlock monitor)*. Team and investor unlocks one tier worse, staking and community one tier better *(same heuristic)*.
- Emissions to farmers are sell pressure too: a reward APY paid in the token is paid by dilution.

## 3. Who holds it
- From `token_holders`: `concentration.top10ExPct` (top 10 outside pools, exchanges, bridges and burns, of unburned supply), `largestEx`, `insiderLikePct` (creator, vesting, locker, governance/treasury, multisig, RugCheck insiders), `creator.pct`, `holderCount`. Its flags: top-10 ex ≥ 30% medium, ≥ 50% high; one holder ≥ 10% medium, ≥ 20% high; creator ≥ 5%; under 500 holders *(all defaults)*.
- Read the kinds: a treasury or timelock under governance and a vesting contract are insider supply that can come to market; a voting-escrow or staking contract holds many users' locked tokens, not one insider's. An unlabelled `wallet` can be an exchange, team or market maker: say so rather than guessing.
- For each large unlabelled holder or the creator, run `address_profile` (onchain-forensics skill): when funded, by whom, whether it has been sending to exchanges.
- Solana: `rugcheckScore`, `authorities` (mint or freeze authority still set is high risk), and RugCheck's insider-graph flag.

## 4. Liquidity and listings
- `token_holders.liquidity`: DEX pools by liquidity, main-pool LP locked or burned share and who holds the rest. Depth at the user's size comes from quotes, not pool totals: `prepare_swap` / `sol_prepare_swap` / `dex_compare` buy and sell at size and 2× (price impact above 2% thin, above 10% you may not get out *(default, as in token-due-diligence)*).
- Listing quality: `cexListings` (GoPlus) and `cex_ticker` on the majors; `hl_markets` for a perp. A perp listing adds shorting pressure into unlocks. DEX-only on one pool with unlocked LP is the weakest form.
- 24h volume ÷ market cap far above peers with thin depth suggests wash trading; say it is a suspicion.

## 5. Report
- One line: **supply risk low / moderate / high / severe**, and the single biggest driver (e.g. "18% unlocks to investors in 3 weeks, 3.1× daily volume").
- Table: float and FDV/MC; next unlocks with pressure tier (or "unknown"); top-10 ex and largest insiders; liquidity at size; listings; revenue vs FDV if any; each with source and time. Then what you could not verify.
- If the user still wants in, size it with `size_position` (position-sizing) and follow swap-execution. This review never authorises skipping a review card, a limit or the owner's approval.

Sources: [S22] Binance Research "Low Float & High FDV" (coverage): https://cointelegraph.com/news/binance-urges-small-medium-projects-high-fdv-trend ; unlock-pressure tiers: adapted from BankrBot/skills `aeon-unlock-monitor` (method only, nothing copied); data: CoinGecko public API, DefiLlama (api.llama.fi), GoPlus token security, Blockscout, RugCheck via `token_holders`.
