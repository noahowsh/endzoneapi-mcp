---
name: odds-explorer
description: "Compares NFL spreads, totals and moneylines across sportsbooks, finds the best price, reads opening-to-closing movement, and browses alternate lines. Use when user asks about the line on a game, which book has the best number, how a line moved, Pinnacle versus the consensus, alternate spreads, or key numbers like 3 and 7. Do not use for devigging or implied probability math, see odds-analysis. Do not use for model versus market edge, see edge-detection. Do not use for player props, see prop-modeling."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Odds Explorer

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`).
> `get_odds` for one game's lines by book (10 credits). `get_line_movement` for the hourly series (25 credits). `get_alt_lines` for ladders (10 credits). `get_games` already carries the consensus closing line (5 credits per query).

You are an expert in football betting markets. Your goal is to show where the best price is, what the market says, and how it moved, in numbers the user can check.

## When to Use

- "What was the closing line for Denver at Kansas City?"
- "Which book had the best number on the Chiefs?"
- "Did the total move after the injury news?"
- "What does a 6.5 point alternate spread cost?"
- "How often do games land on 3 or 7?"

## When NOT to Use

- Removing vig and converting to probability, see `odds-analysis`
- Comparing your model to the market, see `edge-detection`
- Player props, see `prop-modeling`
- Season-long strategy tests, see `backtesting`

## Commands Available

| Command | What It Does | Credits |
|---------|-------------|---------|
| `get_odds` | `game_id`: consensus close plus each book's opening, closing, latest. Or `season` (and optional `team`): a page of per-book closing lines | 10 per game |
| `get_line_movement` | Hourly snapshots grouped by `bookmaker` for one `game_id` | 25 |
| `get_alt_lines` | Alternate spread and total ladders, 2023 to 2025, 6 books | 10 |
| `get_games` | Consensus closing spread, total, moneylines for many games | 5 |
| `get_schedule` | Upcoming games with consensus lines where posted | 2 |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_best_line` | Pull `get_odds`, compare the books yourself |
| `get_public_betting` | Not available. Infer nothing about bet percentages from line moves alone |
| `get_live_odds` | `get_odds` latest snapshot; not an in-game feed |
| `get_futures` | Not available |

## Data Coverage

- Consensus closing spread and total: every game since 1999. Moneyline since 2010.
- Per-book closing: 2014 onward (4 to 9 books through 2019, closing only).
- Opening, closing and latest per book: 2020 onward, up to 8 books including Pinnacle. Each line carries its `source`.
- Books: draftkings, fanduel, betmgm, espnbet, pinnacle, bovada, betonlineag, mybookieag.
- Alternate ladders: one game at a time, 2023 to 2025 seasons.
- Hourly snapshots in season begin 2026. Earlier games have opening and closing only.

## Sign Conventions (the first bug everyone ships)

Two conventions meet in this API. Real example, 2025 Week 17 DEN at KC, where Denver was the favorite:

| Where | Field | Value | Meaning |
|-------|-------|-------|---------|
| `get_games`, `consensus_closing` | `spread_line` | -13.5 | Home-side number: negative means the away team is favored |
| Pinnacle book object | `spread_home` | 13.5 | Book convention: the home team gets +13.5 |
| Pinnacle book object | `spread_away` | -13.5 | Away team lays 13.5 |

Rule: `spread_line` is positive when home is favored. `spread_home` is negative when home is favored. They are negatives of each other. Convert before you compare.

Home covers if `result > spread_line` where `result` is home score minus away score. In the example, KC lost 13 to 20, `result` = -7, which is greater than -13.5, so KC covered.

## How It Works

**Step 1: Get the game id.** From `get_games` or `get_schedule`. Format `2025_17_DEN_KC`.

**Step 2: Pull the book lines.** `get_odds(game_id="2025_17_DEN_KC")`. Add `bookmaker="pinnacle"` to narrow. Real shape of one book entry: `opening`, `latest`, `closing`, each with `ml_home`, `ml_away`, `spread_home`, `spread_home_odds`, `spread_away`, `spread_away_odds`, `total`, `total_over_odds`, `total_under_odds`, `captured_at`, `source`.

**Step 3: Compare.** For each market, the best price for each side independently. Spreads: a half point matters more at 3 and 7 than elsewhere. Moneylines: higher number is better on both sides.

**Step 4: Movement.** From the real Pinnacle entry: total opened 36.5 at -107/-107 and closed 37.5 at -114/+101. A one point total move, with the over price rising, means money or information pushed the total up. Spread stayed at 13.5, only the price moved (-103 to -109 on the home side). Report all three: the number, the price, the time window (`captured_at`).

**Step 5: Key numbers.** In the NFL, 3 and 7 are the most common margins of victory, then 6, 10, 14. Moving a spread across 3 is worth far more than crossing 2 or 4. Measure the actual frequency from `get_games` results (`result` column) over the seasons you have rather than quoting a remembered figure.

**Step 6: Alternate lines.** `get_alt_lines(game_id="2025_17_DEN_KC", market="alternate_spreads", bookmaker="pinnacle")` returns the ladder with `side` (home or away) and each point's price. Buying points has a price; compare to the main line's implied probability change (see `odds-analysis`).

## Reference Line

Pinnacle is the sharpest book in the feed and the one to treat as the benchmark for closing-line value (see `backtesting`). It is a reference, not a recommendation: its availability and limits differ by user.

## Credit Usage

| Operation | Credits |
|-----------|---------|
| Consensus closing for a whole season | 5 (`get_games`) |
| One game, all books | 10 |
| One game, hourly movement | 25 |
| One game, alternate ladders | 10 |
| A season of per-book lines | 10 per game, 1,000 for a 100-game page |

Use `estimate_only=true` on `get_odds` to price a season pull before running it. It charges nothing.

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "Spread -13.5 means the home team is favored" | `spread_line` is home-perspective, positive = home favored | Check against the final score of a known game |
| "Latest line equals closing line" | Latest is the newest snapshot, which may be hours before kickoff | Use the `closing` snapshot for historical work |
| "Any 0.5 move is equal" | Moves through 3 and 7 carry real probability | Weight by key number |
| "Books agree, so skip the comparison" | Prices differ even when numbers match | Compare the odds, not just the line |

## Output Format

```
DEN @ KC -- Dec 25, 2025 (kickoff 01:15 UTC Dec 26)
Consensus close: DEN -13.5, total 37.5, ML DEN -1100 / KC +700
Pinnacle: KC +13.5 (-109), DEN -13.5 (-102); total 37.5 (O -114 / U +101)
Movement: total 36.5 -> 37.5 (+1.0), spread unchanged, DEN price -109 -> -102
Note: spread_line is -13.5 (home-side), the book shows KC +13.5.
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| Two-sided price | Remove vig | `odds-analysis` |
| A number and a model estimate | Compute edge | `edge-detection` |
| A season of lines | Test a rule | `backtesting` |
| Large move | Check injuries and news in the matchup | `game-preview` |
