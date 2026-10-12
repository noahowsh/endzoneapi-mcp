---
name: prop-modeling
description: "Builds NFL player prop models for passing yards, passing touchdowns, rushing yards, receiving yards, receptions and anytime touchdown, using prop history, usage, snap share, game script from the spread and total, and injuries. Use when user asks about player props, whether a player goes over a yardage line, prop projections, anytime TD odds, or the historical over rate against a line. Do not use for team game prediction, see feature-engineering. Do not use for browsing odds across books for a game side, see odds-explorer. Do not use for player profiles without a line, see player-scouting."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Prop Modeling

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`).
> `get_player_props` (10 credits) for lines, `get_prop_history` (10 credits for up to 200 games) for how a player did against his line, `get_snap_counts` and `get_player_stats` (5 credits each) for usage, `get_games` (5 credits) for the game environment.
> Props coverage starts in the 2023 season. Samples are small by nature.

You are an expert in player prop modeling. Your goal is to project a player's stat for one game as a distribution, compare it with the book's line and price, and be honest about how little data a single player-season gives you.

## When to Use

- "Will Mahomes go over 250.5 passing yards?"
- "What has this receiver done against his line this year?"
- "Project a running back's yardage"
- "What is the fair price on an anytime touchdown?"
- "Which book has the best number on this prop?"

## When NOT to Use

- Which team wins, see `feature-engineering`
- A team's spread price, see `odds-explorer`
- General scouting, see `player-scouting`

## Commands Available

| Command | What It Does | Credits |
|---------|-------------|---------|
| `get_player_props` | Props by book. `game_id` for all players in a game, or `player_id` plus `season`. `market`, `bookmaker`, `snapshot_type` (closing or opening), `estimate_only` | 10 per game returned |
| `get_prop_history` | One player and market: a row per game with `line`, `open_line`, prices, `actual`, `result`, `margin`, plus an over-rate summary. `from_date`, `to_date`, `bookmaker`, `limit` | 10 for up to 200 games |
| `get_snap_counts` | Snaps and shares by game, `player_id` (PFR id), or `team` and `season` | 5 |
| `get_player_stats` | Weekly lines for the season | 5 |
| `get_injuries`, `get_depth_chart` | Availability and role | 2 each |
| `get_games` | Spread and total for the game environment | 5 |
| `get_advanced_stats` | Tracking and charting context | 5 |
| `search_players` | The ids (GSIS for props and stats, PFR for snaps) | 2 |

## Markets

`get_player_props` markets: `player_pass_yds`, `player_pass_tds`, `player_rush_yds`, `player_reception_yds`, `player_receptions`, `player_anytime_td` (over means he scores).
`get_prop_history` adds `player_pass_completions`, `player_pass_attempts`, `player_pass_interceptions`, `player_rush_attempts`.
Books on props: DraftKings, FanDuel, BetMGM, ESPN BET, Pinnacle, Bovada, BetOnline.

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_player_projection` | Build it as below |
| `get_dfs_projections` | Not provided |
| `get_prop_edges` | Compute edge from your distribution and the line |
| Props before 2023 | Not covered |

## Real Data Shapes

`get_prop_history(player_id="00-0033873", market="player_pass_yds", from_date="2025-09-01", limit=3)` (Mahomes) returned:

| game | line | open | over | under | books | actual | result | margin |
|------|------|------|------|-------|-------|--------|--------|--------|
| 2025_01_KC_LAC | 250.5 | 244.5 | -120 | -110 | 7 | 258 | over | +7.5 |
| 2025_02_PHI_KC | 245.5 | 245.5 | -110 | -121 | 7 | 187 | under | -58.5 |

with `summary` = overs 1, unders 2 of 3 graded, `over_rate` 0.333, `avg_line` 243.5, `avg_actual` 223, and `next_from_date` to continue. Three games is an illustration, not a signal.

`get_player_props(game_id="2025_17_DEN_KC", market="player_pass_yds")` returned 14 rows, each `{player_name, gsis_id, market, bookmaker, line, over_odds, under_odds, captured_at, source}`; for example Bo Nix at 228.5 with -115 on both sides at BetMGM. One row per player per book.

## How It Works

**Step 1: Establish the role.** Starter or not (`get_depth_chart`), injury status, snap share and target share over the last 4 to 6 games (`get_snap_counts`, `get_player_stats`). Prop outcomes follow opportunity.

**Step 2: Game environment.** From the spread and total, the implied team totals:

```
home_implied = (total + spread_line) / 2
away_implied = (total - spread_line) / 2
```

`spread_line` is positive when home is favored. A team implied for 26 points passes and scores differently from one implied for 17. Trailing teams pass more; favorites run late.

**Step 3: Decompose the stat.**

| Market | Decomposition |
|--------|---------------|
| Passing yards | attempts x yards per attempt (attempts depend on pace and game script) |
| Passing TDs | attempts x TD rate; or team implied points x passing share |
| Rushing yards | carries x yards per carry (carries depend on script and share of backfield) |
| Receiving yards | team pass attempts x target share x yards per target |
| Receptions | targets x catch rate |
| Anytime TD | expected scoring chances (red-zone usage) with Poisson probability `1 - exp(-lambda)` |

Project each component from rolling usage with shrinkage (see `feature-engineering`) and multiply, or fit a regression directly on past games from `get_prop_history` and `get_player_stats` (the history gives `actual` per game).

**Step 4: Distribution, not a point.** Yardage is right-skewed. Use a gamma or a log-normal fit to the player's game residuals, or a negative binomial for counts such as receptions. The variance is the major input: a projection of 250 with a standard deviation of 70 puts a 250.5 line near 50/50, nowhere near a lock.

```
P(over line) = 1 - F(line)      F: fitted distribution of the stat
```

Compute `P(over)` and `P(under)` and compare with the devigged line price (see `odds-analysis`). The line itself is the market median projection.

**Step 5: Compare.** Edge = your probability minus the no-vig probability at the price. Shrink toward the market (see `edge-detection`). If your projection is far from the line (say 15 percent), suspect missing information (injury, role change, game script) before trusting the model.

**Step 6: Shop.** Different books carry different lines for the same prop. `get_player_props` returns them side by side. A half-point on a yardage line moves the over probability only a little but matters on an even-priced bet.

## Sample Size

- Props cover 2023 onward: at most about 17 games a year per player, 3 seasons by 2025, so roughly 50 games for a durable starter.
- A player-market history of 17 games has an over rate with a standard error near 12 percentage points. A 60% over rate in 17 games is consistent with 50%.
- The line is already built from the same stats, so a historical over rate away from 50% is weak evidence unless it is large and persistent.
- Do not tune a prop model on the games you test. Use walk-forward (see `walk-forward-validation`).
- Anytime TD is a rare event: a back at 0.5 TDs a game scores in about 40 percent of games; one season holds too few for a stable rate. Use team red-zone share as a prior.

## Correlation

A quarterback's passing yards and his top receiver's receiving yards move together. Two props from the same game are not independent bets. Same-game parlays multiply the book's margin; evaluate each leg's edge, and avoid assuming independence.

## Credit Usage

| Operation | Credits |
|-----------|---------|
| Prop lines for a whole game | 10 |
| One player-market history | 10 |
| One player's props across a season | 10 per game returned |
| Usage (snaps, stats) | 5 each |
| Full model for one player | about 35 |

`estimate_only=true` on `get_player_props` shows the price first. Full-league prop history across seasons is expensive; start with the players that matter to the question.

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "Average is 262 so over 250.5" | Ignores variance, role, script | Fit a distribution |
| "He is 11 of 15 over" | Small sample, line already adjusts | Report with an interval |
| "Starter is out, use the old role" | Usage shifts | Reload the depth chart |
| "Parlay these two props" | Correlated, extra margin | Price legs separately |
| "Use opening line for a closing bet" | Different information | State snapshot used |

## Output Format

```
Mahomes, passing yards, KC vs [opp]
Environment: total X, KC implied Y
Projection: attempts A x YPA B = C (distribution: gamma, sd D)
Line 250.5 (best over: [book] -115, best under: [book] -110)
P(over) 0.xx vs no-vig 0.xx -> edge +/-x.x pts (before shrinkage)
History: over rate x/y games (not predictive alone)
Risks: [injury news, game script, line moves]
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| A projection gap | Calibrate and shrink | `probability-calibration` |
| Several prop ideas | Test on history | `backtesting` |
| A needed price | Convert odds | `odds-analysis` |

## Reference

Python walkthrough: prop lines by sportsbook: https://endzoneapi.com/guides/nfl-player-prop-lines-by-sportsbook-python
