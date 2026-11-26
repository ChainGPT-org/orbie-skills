---
name: copy-trading-safety
description: Vet a trader before copying or following them, pick copy targets from the Hyperliquid leaderboard, read smart-money vs crowd positioning, set safe limits for copy_follow (Solana) and hlcopy_follow (Hyperliquid), and decide when to go from paper to real. Use when the user wants to copy, mirror or follow a trader, "smart money", a whale or an influencer wallet, asks "who should I copy?", "is 0x… any good?", "where is smart money positioned?", or wants to switch a copy from paper to real money.
---

# Copy-trading safety

A copier buys after the leader, at worse prices. The leader's PnL is not yours. The job is telling a proven edge from a lucky streak, then sizing small. Start in paper. Gates marked *(Senpi)* adapt Senpi's trader-research skill (NOTICE.md); *(default)* are Orbie's judgment.

## 1. Pick candidates (Hyperliquid)
- `hl_leaderboard` (window `month` or `allTime`, sort `pnl`). It drops accounts under $10k and market-maker-like books (month volume ≥ 200× account value *(default)*): their P&L is quoting, not views you can copy. Sort by `roi` only with a high `min_account_value`; small accounts make ROI noise.
- Two clocks *(Senpi)*: month/all-time P&L says whether they are good; day/week says whether they are hot. Hot with no record (`hot_not_proven`) is the classic trap. Say which clock each number is on.
- Read the flags: `all_time_loss`, `hot_not_proven`, `in_pullback` (week down inside an up month), `roi_pnl_conflict` (don't lead with the ROI). `drawdownProxy` is only the worst day/week; the real drawdown comes from `hl_trader`.
- Leaderboard figures can include sub-accounts or vaults; `hl_trader` reads the address itself and may differ. Cite both when they do.

## 2. Vet one trader (`hl_trader`, any address)
Lead with its `vetting.verdict`, then the gates behind it:
- **not_yet**: `thin_track_record` *(Senpi)*: under 5 closed trades or 7 active days is noise, however good the ROI. Never recommend a copy off it.
- **avoid**: any of `all_time_loss`, `blowup_risk` (worst drawdown ≤ −83% of the account *(Senpi)*), `liquidated_before` *(default)*, `critical_margin_usage` (> 90% of the account in margin *(Senpi)*). Copying into a book near liquidation inherits it on day one.
- **proceed_with_care**: `high_margin_usage` (> 80%) *(Senpi)*, `concentrated_book` (one position > 60% of notional) or `single_position` *(Senpi)*, `currently_in_drawdown` *(Senpi)*, `high_leverage` (effective leverage > 10x *(default)*), `high_turnover` (> 8 round trips a day *(Senpi)*: fees and delay eat the copy), `dormant` (no trade in 30 days *(Senpi)*), `losing_this_month`, `erratic_pnl` *(default)*.
- **candidate**: no gate tripped. Still a past record, not a promise.
- Also read: win rate with average win and loss and profit factor together (a 70% win rate with losses three times the wins loses); average and median hold (minutes-long holds can't be copied by a person or an agent waking on alerts); fees; liquidation distance per position. `partial_fill_history` means stats cover the latest 2,000 fills only.
- Composing the answer: one honest sentence first ("I'd pass: +210% but 6 trades in 4 days"), then the numbers with their windows, then the risk to size around.

## 3. Smart money vs the crowd (`hl_smart_money`)
- Cohorts *(Senpi)*: wallets with all-time P&L ≥ $1M (proven) vs $10k–$100k (the crowd). Senpi uses realized P&L; Orbie uses the leaderboard's all-time P&L, which includes open positions. Say so.
- Lead with `divergences` where `oppositeSides` is true, then `smartLeaning`. Always pair a bias with its member count: −0.9 across 7 wallets is not −0.3 across 3. A few whales can dominate net USD; members count heads.
- It is positioning now, not a timing call: the proven cohort is often early. Use it as context for a copy target (is the leader with or against the proven money?) and for the user's own book (perp-trade-plan), never as a trade signal on its own.
- It reads up to 100 accounts; don't loop it on a schedule. Once or twice a day at most, and say each run costs model time when an agent runs it.

## 4. Copy a Hyperliquid leader (`hlcopy_*`)
One target per agent, followed without the model: Orbie reads the target's book every 5 s and mirrors entries, adds, cuts and exits.
- **Start on paper** with `hlcopy_follow` (agents may only start or change paper copying). It vets the target first; real money needs a `candidate` or `proceed_with_care` verdict, and only the owner turns it on, in the agent's Details with Touch ID.
- **Sizing** is proportional: the target's change × (agent equity ÷ target account value), scaled down when the target runs more effective leverage than `max_leverage`. Then capped per trade (never above the agent's per-trade limit), per coin, by total leverage and by open positions. Leverage per position is never above the target's own. Shares under the $11 minimum wait until later changes add up.
- **Exits** follow the target's state: a cut is the same fraction of ours, a close or flip closes ours, and a copy the target no longer holds is closed even if the change was missed. Every copied position gets a protective stop (`stop_loss_pct` from entry, or the target's own stop when tighter).
- **Skipped on purpose**: entries whose book walk is worse than the slippage limit, entries placed after the price ran 1.5% past the target's entry, builder-deployed (`xyz:`) markets, anything while paused. Say so; a copier fills later and worse.
- **Auto-pause** *(default)*: re-vetted every 2 h; pauses on an `avoid` verdict, dormancy, a month drawdown 10 points worse than at the start (and at least −15%), or effective leverage at 2× the start (and at least 10x). Paused copies open nothing but still mirror exits. Resuming real money needs Touch ID.
- **Review** with `hlcopy_status`: copied vs skipped and why, held orders, the copy's positions with their stops and its P&L. Paper results are labelled paper. Stop with `hlcopy_stop`; positions stay open with their stops.
- Real orders are Hyperliquid intents run only within the agent's limits and loss protection; anything above them is held for the owner, and a held exit tells the owner. A trader's book can also inform the user's own plan (perp-trade-plan) instead: size at the user's risk, not the leader's.

## 5. Solana wallets (`copy_follow`)
1. **Vet the leader (read-only).** Recent trades from public explorers and analytics pages (Solscan, Birdeye, GMGN) via WebFetch or the browser; cite links. The same gates apply: want 30+ days and 50+ trades *(default)*; under 5 trades or 7 days is no record *(Senpi)*.
   - Realized vs unrealized PnL; win rate with average win and loss. A few lucky 100x hits among many losses won't repeat for a copier.
   - Holding time: a leader who flips within seconds can't be copied. Copies land seconds later; leader trades older than 2 minutes are never copied.
   - The leader as the product: creating tokens and buying them (dev wallets), many fresh tokens with tiny liquidity, buying right before posting about a token (you become exit liquidity), coordinated buys across related wallets at launch.
   - Scale: a whale's small position can be a large one for you.
2. **Configure `copy_follow` conservatively.**
   - `mode: paper` (the default) until 20+ copied trades have played out.
   - `size_usd`, fixed and small (about 1–2% of the copy budget per buy). Avoid `size_pct` of a large wallet.
   - `slippage_bps` 300–500 for memecoins. It is also the max price impact: higher means worse fills, not more fills.
   - `max_trades_per_hour` low (5–10) so a burst of spam buys can't drain the wallet.
   - `deny` mints you never want; `allow` to restrict to a known list. Tokens that fail token-due-diligence go on `deny`.
3. **Review the paper run** with `copy_list`: copied vs skipped (no route, impact, rate limits), average delay, and copier PnL vs the leader's on the same trades. The gap is the cost of being late; if it eats the edge, stop.
4. **Going real** only if paper results hold: in a dedicated agent whose own wallet holds only the copy budget, with per-trade and daily limits that match the sizes above. Every real copy is a Jupiter swap intent within those limits.

## 6. Exit rules, written up front
Stop after a drawdown of X% of the copy budget or N losses in a row (the agent's loss protection does this for real Hyperliquid copies), and `copy_unfollow` / `hlcopy_stop` when the leader's style changes, its verdict drops to avoid, or it is liquidated.

Label paper results "paper" every time.
