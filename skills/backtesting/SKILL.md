---
name: backtesting
description: "Simulates an NFL betting strategy against historical closing lines without lookahead: flat and Kelly staking, ROI, drawdown, bootstrap intervals, and closing-line value against Pinnacle. Use when user asks whether a strategy would have been profitable, to test a system over past seasons, what a season of data costs, closing-line value, or whether an edge persists across seasons. Do not use for live pick sizing, see edge-detection. Do not use for model evaluation without bets, see walk-forward-validation. Do not use for prop tests, see prop-modeling."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Backtesting

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`).
> **Cheap route:** `get_games` (5 credits per query) returns a whole season with scores and the consensus closing spread, total and moneylines. **Per-book route:** `get_odds` with `season` costs 10 credits per game returned. Price it free with `estimate_only` first.
> Data length: consensus closing lines since 1999; per-book closing since 2014; opening, closing and latest per book since 2020.

You are an expert in honest historical testing. Your goal is to find out whether a strategy's edge is real, and your default is to doubt it. The test must use only prices that were available at bet time and a model fit only on earlier data.

## When to Use

- "Would betting home underdogs after a bye have made money?"
- "Does my model beat the closing line over 2021 to 2025?"
- "What is my ROI with a confidence interval?"
- "How much will a season of odds cost?"
- "Does the edge hold across seasons?"

## When NOT to Use

- Today's bet decision, see `edge-detection`
- Evaluating predictions without betting, see `walk-forward-validation`
- Player prop tests, see `prop-modeling`

## Commands Available

| Command | Use Here | Credits |
|---------|---------|---------|
| `get_games` | Season of results, scores, consensus close, `result`, rest, QBs | 5 per query |
| `get_odds` | `season` (optional `team`, `bookmaker`, `snapshot_type`, `limit`, `offset`, `estimate_only`): per-book lines with scores | 10 per game |
| `get_line_movement` | Intraweek movement, in-season 2026 onward hourly | 25 per game |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_backtest` | This skill, in your own code |
| `get_historical_odds` | `get_odds` with `season` |
| `get_closing_line_value` | Compute it from your bet price and the closing snapshot |

## Cost Planning

| Pull | Games | Credits |
|------|-------|---------|
| One season consensus (`get_games`, `limit` 500) | about 285 | 5 |
| Five seasons consensus | about 1,400 | 25 |
| One season per-book, 100 per page (`get_odds`) | 100 | 1,000 |
| One full regular season per-book | 272 | 2,720 |
| Pinnacle only, one season | 272 | 2,720 (the cost is per game, not per book) |

Real number: `get_odds(season=2025, estimate_only=true)` reported 1,000 credits for the default 100-game page. Always estimate first, then decide whether consensus lines are enough. For most spread and total systems they are.

## Rules of an Honest Backtest

1. **Bet price must be a price that existed.** A consensus closing line is a price you could not always get. Use an actual book's `opening` or `closing` snapshot for the odds, and say which.
2. **Decision time before bet time.** If the rule uses information from Sunday morning, the price used cannot be the Tuesday opening. Match snapshot and information.
3. **Model fit only on earlier data.** Walk-forward (see `walk-forward-validation`). A rule found by scanning all seasons and then tested on the same seasons is the definition of overfitting.
4. **Count the rules you tried.** Trying 40 situational angles and reporting the best is the best of 40 noise draws. Report the count.
5. **Include pushes.** A push returns the stake. On 3 and 7 point spreads they are common enough to move ROI.
6. **Use the price.** -110 does not equal an even payout. ROI on a spread bet at -110: win +0.909, lose -1, break-even 52.38%.

## Method

```python
def settle_spread(side, line_home, result, odds):
    # line_home: home-side line, positive = home favored (spread_line)
    margin = result - line_home if side == "home" else line_home - result
    if margin == 0: return 0.0                       # push
    win = margin > 0
    b = 100 / -odds if odds < 0 else odds / 100
    return b if win else -1.0
```

`result` is home score minus away score. With `spread_line` the home-side line, the home team covers when `result > spread_line`. A sportsbook object's `spread_home` has the opposite sign, so convert first (see `odds-explorer`).

**Step 1:** Pull seasons of `get_games`. **Step 2:** Generate the rule's bets using only fields known before kickoff. **Step 3:** Settle at an honest price. **Step 4:** Report units won, ROI, number of bets, win rate, push rate. **Step 5:** Bootstrap.

## Significance

At -110, one bet has a standard deviation near 0.95 units. After N bets the standard deviation of ROI is about 0.95 / sqrt(N). With 300 bets that is about 5.5 percentage points. A 52.4% win rate (break-even) versus 55% over 300 bets is well inside noise.

```python
rng = np.random.default_rng(1)
rois = [rng.choice(units, len(units)).mean() for _ in range(5000)]
lo, hi = np.percentile(rois, [2.5, 97.5])
```

Report the interval. Also resample by week, not by bet, because bets on the same Sunday are correlated.

## Closing-Line Value (CLV)

CLV asks whether your bet price beat the closing price. For a spread bet, the point difference between your number and Pinnacle's close; for a moneyline, the difference in devigged probability. Positive CLV over hundreds of bets is the best leading evidence of skill, because it does not depend on game outcomes. Requires per-book data (2020 onward; Pinnacle in the feed). If you only have consensus lines, you can compute CLV between the consensus open and close if both exist for the games you tested, otherwise say CLV cannot be measured.

## Persistence

Split results by season. An edge that exists in one season and not the others is more likely noise. Show the per-season table beside the pooled one. Check for compression: lines have become more efficient over time, so an edge from 2005 to 2012 says little about now.

## Credit Usage

See Cost Planning. Strategy testing on consensus lines for ten seasons: 50 credits.

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "Home dogs after a bye won 56%" | Found by scanning many angles | Count angles, test out of sample |
| "Used the closing line to bet" | You often cannot get it | Use opening or a named book's snapshot |
| "ROI is +4%" | With 250 bets that is noise | Report the interval |
| "Drop pushes" | Overstates win rate | Settle them at 0 |
| "Pool all seasons" | Hides decay | Per-season table |
| "Full season of `get_odds` for every idea" | 2,720 credits each | Consensus first |

## Output Format

```
Strategy: [rule], seasons 2021-2025, bets N, pushes P
Units: +X.X | ROI: +x.x% (95% CI -x.x% to +x.x%, weekly bootstrap)
Win rate: xx.x% vs break-even 52.4%
By season: 2021 .. 2022 .. 2023 .. 2024 .. 2025 ..
Rules tried before this one: K
CLV vs Pinnacle close: +x.x points average (n bets with data)
Verdict: [supported / not separated from noise]. Past results do not promise future returns.
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| Positive and persistent | Size cautiously, paper trade | `edge-detection` |
| Noise | Stop or re-specify with a new hypothesis | `feature-engineering` |
| Prop angle | Test with prop history | `prop-modeling` |

## Reference

Python walkthrough: backtesting against closing lines: https://endzoneapi.com/guides/backtest-nfl-closing-lines-python
