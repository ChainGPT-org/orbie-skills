---
name: prediction-market-risk
description: Check a Polymarket market before any order: the resolution rules and source, UMA's optimistic oracle and dispute risk, liquidity and spread at your size, time to resolution, correlated bets, and sizing as a binary bet. Use before pm_prepare_order, when the user or an agent wants to bet on, buy shares in or market-make a prediction market, or asks whether a market's price is a good bet.
---

# Prediction market risk

A Polymarket share pays $1 if its outcome wins and $0 if not. The price is the crowd's probability, not a guarantee, and the money is lost or won at **resolution**, not at the price. Thresholds marked *(default)* are Orbie's judgment, not a rule from a source: say so when you quote them.

## 1. Read the rules, not the title
- `pm_market`: its description (the rules) and resolution source. The market resolves on the rules, not the title [S1].
- Write down in one line: what exactly must happen, by when (time zone), judged by which source.
- Ambiguity is the main loss cause. Red flags: "officially", "announced", "credible reporting" with no named source; edge cases the rules don't cover (a delay, a partial event, a rename); a source that may not publish in time.
- Rules that depend on interpretation: **skip** *(default)*. For an agent, that's a reject.
- Polymarket may add "additional context" after trading starts, published on-chain; it can't change the question's intent [S1]. Check for it, and know it can arrive after you've bought.

## 2. How resolution can go wrong (UMA)
- Anyone proposes the outcome with a bond (check UMA's current amount); with no dispute it resolves about 2 hours later [S1].
- A dispute starts a second proposal; a second dispute goes to a UMA token-holder vote (about 48 hours). Disputed markets take about 4–6 days in total [S1].
- A market can resolve 50/50 (each share redeems $0.50) when the outcome is unknown [S1]. A position bought at $0.95 then loses ~47%.
- A price near $0 or $1 is not settlement. Never treat "it's at 99¢" as done; the last cent is the dispute risk premium.
- Oracle risk is real: the vote is by UMA token holders, not a court. Large positions in contested markets carry that risk on top of the event.

## 3. Liquidity and price at your size
- `pm_market`: best bid and ask per outcome, tick size, minimum shares. Spread above 3¢ = thin *(default)*: a market order pays half the spread or more.
- Your size against the book: if filling would move the price more than 2¢ *(default)*, use a limit order or a smaller size.
- Exit: can you sell before resolution? A thin bid means you're holding to the end.
- Liquidity rewards pay makers for two-sided quotes within a market's max spread and min size [S2]; they don't make a bad bet good.

## 4. Time to resolution
- `pm_markets` end date. Capital is locked until resolution plus any dispute days.
- Compare the edge with the time: a 3¢ edge on a 50¢ share is 6% for the whole term: about 12% a year if it resolves in 6 months, about 3% a year over 2 years, before fees and dispute risk. Compare with a stablecoin yield (`mkt_yields`). Long-dated low-edge bets: skip *(default)*.
- The end date is when trading stops, not always when the event is decided. Read the rules for which.

## 5. Correlated bets
- `pm_positions`: everything already held. Markets on the same event (a candidate winning a primary and the general, "by June" and "by December", several outcomes of one neg-risk event) are **one bet**.
- In a neg-risk event (one winner among many) a No in one market converts into Yes in all the others [S3]: price the whole event, not one leg.
- Sum the stakes on one underlying event; that total is the position.

## 6. Size it as a binary bet
- Max loss = the whole stake. Size from `position-sizing` with the stop at $0: risk per trade = the stake.
- One event ≤ 2% of the wallet *(default)*; correlated markets count together.
- Kelly for a binary bet at price p with your probability q: f* = (q − p) ÷ (1 − p). Use at most ½ Kelly, and only when q comes from evidence you can name (polls, data, a schedule), not a feeling [S4]. No edge you can state = no bet.

## 7. Before preparing
- Region: `pm_prepare_order` checks Polymarket's geoblock. A refusal is final; never work around it.
- Output for the owner: the rule in one line, resolution source, end date, spread and depth at the size, related positions, your probability vs the price and where it comes from, max loss.
- Then `pm_prepare_order` (a limit order at a price you chose) and `execute`. The review card, the agent's limits and any pre-trade checks still decide.

Never buy because a price "looks cheap" near 0, never average down into a market you no longer understand, and never treat 99¢ as free money.

## Sources
- [S1] Polymarket, resolution via UMA: https://docs.polymarket.com/developers/resolution/UMA
- [S2] Polymarket, liquidity rewards: https://docs.polymarket.com/developers/market-makers/liquidity-rewards
- [S3] Polymarket, negative-risk markets: https://docs.polymarket.com/advanced/neg-risk.md
- [S4] Thorp, "The Kelly Criterion in Blackjack, Sports Betting and the Stock Market": https://sites.oxy.edu/lengyel/M330/thorp/paper.htm
