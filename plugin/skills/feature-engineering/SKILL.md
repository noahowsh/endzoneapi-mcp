---
name: feature-engineering
description: "Builds leak-free predictive features for NFL games and players: rolling EPA, opponent-adjusted rates, rest and travel, quarterback changes, Elo, and market-derived inputs. Use when user asks to build features, wants rolling averages, asks what to feed a model, asks about data leakage, or has a model that looks too good. Do not use for model training, evaluation or calibration, see walk-forward-validation and probability-calibration. Do not use for a single descriptive stat, see epa-analysis."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Feature Engineering

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`).
> `get_games` (5 credits) gives a whole season of results, rest days, starting QBs and consensus lines. `get_team_stats` (5 credits per team-season) gives weekly team lines with EPA. `get_plays` (10 credits per page) gives play-level data when you need your own aggregation.
> For your own CSV: skip the tools, no credits used.

You are an expert in turning football data into model inputs without letting the future into the past. Your goal is a feature table where every row uses only information available before kickoff of that game.

## When to Use

- "What features should I use to predict NFL games?"
- "Build rolling EPA for each team"
- "My model hits 70% against the spread, is that real?"
- "How do I handle the quarterback changing?"
- "How do I use the closing line as a feature?"

## When NOT to Use

- Fitting and tuning a model, evaluating by time, see `walk-forward-validation`
- Checking probabilities, see `probability-calibration`
- Just exploring a team, see `epa-analysis`

## Commands Available

| Command | Feature Use | Credits |
|---------|-------------|---------|
| `get_games` | Target (`result`, `total`), rest, QBs, line, `div_game`, roof, surface | 5 per query |
| `get_team_stats` | Weekly team lines in `weeks[]` (passing EPA, rushing EPA, sacks, penalties, kicking) | 5 per team-season |
| `get_plays` | Play-level EPA, success, CPOE, down, distance, win probability, charting | 10 per page |
| `get_injuries` | Out/doubtful flags by week (2009+) | 2 |
| `get_snap_counts` | Usage and availability | 5 |
| `get_advanced_stats` | Tracking and pressure by week | 5 |
| `get_odds` | Opening line vs closing line, per book (2020+) | 10 per game |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_features` | Build from the calls above |
| `get_power_ratings` | Compute your own Elo from `get_games` results |
| `get_strength_of_schedule` | Derive from opponents' rolling rates |
| `get_travel_distance` | Use `list_teams` for division and your own stadium table |

## The One Rule: Information Cutoff

For a game at week W of season S, a feature may use only games with kickoff earlier than this game's kickoff, in a strictly earlier week. In pandas, sort by team and date, compute the statistic, then `shift(1)`.

```python
g = team_weeks.sort_values(["team", "season", "week"]).copy()
g["epa_l5"] = (g.groupby(["team"])["epa_per_play"]
                .transform(lambda s: s.shift(1).rolling(5, min_periods=3).mean()))
```

Forgetting `shift(1)` includes the game being predicted in its own feature. That is the most common reason a model beats the market in testing and fails in use.

## Feature Families

**1. Team efficiency (rolling).** Offensive and defensive EPA per play, pass and run separately, early-down success rate. Windows of 5 and 10 games plus an expanding season mean. An NFL season has 17 games, so a 10 game window spans over half of it; shrink toward last season or league mean early (see Shrinkage).

**2. Opponent adjustment.** A rolling mean is biased by who the team played. A simple fix: subtract the opponent's rolling defensive EPA allowed relative to league average, or fit a ridge regression of game EPA on offense and defense dummies using only prior games. Adjustment on 5 games is noise; use 8 or more or shrink.

**3. Quarterback.** The starting QB is the largest single driver. `get_games` returns `away_qb_name` and `home_qb_name`. Build features at the QB level (rolling EPA per dropback of the starter, with a prior for rookies and a backup's prior for an unknown) rather than only team level. Flag a QB change from the prior week.

**4. Situation.** `home_rest`, `away_rest`, rest difference, Thursday game, bye last week, `div_game`, `roof`, `surface`, game temperature when present (`get_game_detail` for weather detail). Treat missing weather as missing, not zero.

**5. Ratings.** Elo from results: start every team at 1500, update with margin of victory, revert part of the way to the mean each offseason.

```
expected_home = 1 / (1 + 10 ** (-(elo_home - elo_away + home_adv) / 400))
elo_home += K * mov_mult * (actual_home - expected_home)
```

Tune `K`, `home_adv` and the offseason carryover (typically keep two thirds to three quarters, verify on your data) by walk-forward validation, not by eye.

**6. Market features.** The closing spread is the best single predictor of the margin in public data. Whether to include it depends on the question. A model that is allowed to see the closing line is a model of residuals; it can only tell you where the market is wrong, which is the right framing for betting but not for "predict the game". Never use a closing line as a feature for a bet placed before it exists. Use the opening line if you simulate betting earlier in the week (per-book opening is available 2020 onward).

**7. Availability.** Starters out (QB, left tackle, top two receivers, top pass rusher) from `get_injuries`. Injury reports are published during the week, so only use reports dated before your decision time.

## Shrinkage and Small Samples

Early-season rolling stats are mostly noise. A standard fix is a weighted blend of prior-season rate and current rate:

```
rate = (n * current + k * prior) / (n + k)
```

with `n` games played and `k` a prior weight (start near 6 to 10 games for EPA, tune it in validation). Regress the prior toward the league mean as well; rosters change.

## Targets

| Target | Definition | Note |
|--------|-----------|------|
| Margin | `result` (home minus away) | Regression, then convert to a win probability |
| Cover | `result > spread_line` | Pushes need handling; 3 and 7 are common margins |
| Total | `total > total_line` | Weather and pace features matter |
| Residual | `result - spread_line` | Needs the line known before the bet |

Keep push rows in a separate column rather than dropping or mislabeling them.

## Leakage Checklist

| Leak | How It Hides |
|------|-------------|
| Game's own stats in its rolling feature | Missing `shift(1)` |
| Final season stats used for in-season rows | `get_team_stats` season `totals` include later weeks. Use `weeks[]` instead |
| Closing line as input for an earlier bet | Closing is the last pre-kickoff snapshot |
| Season-end Elo or rankings applied to earlier weeks | Recompute by week |
| Injury status taken after the game | Date-stamp reports |
| Random train/test split | Time order is the point, see `walk-forward-validation` |
| Same game appearing from both team views in train and test | Split by game, not by team-week |

## Credit Usage

| Operation | Credits |
|-----------|---------|
| One season of games and lines | 5 |
| All 32 teams, one season of weekly team lines | 160 |
| One season of plays for one team (about 1,100 plays, paged) | 10 to 20 |
| Whole-league plays for a season | thousands, plan before pulling |

Cheapest useful set: `get_games` for 5 or 6 seasons (25 to 30 credits) gives results, rest, QBs and consensus lines for a first model.

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "More features will help" | 272 regular season games a year, few hundred per many windows | Start under 15 features and add one family at a time |
| "Use season totals from `get_team_stats`" | They include games after the one you predict | Use `weeks[]` with a cutoff |
| "Drop pushes" | Skews the cover rate | Keep them as a third class |
| "Fill missing weather with 0" | Indoor and missing are different | Use a missing flag |
| "Last year's EPA predicts this year" | Only partially, QB and rosters change | Blend with shrinkage |

## Output Format

```
Feature table: one row per game, keys (season, week, game_id)
Columns: home/away rolling EPA (5, 10), pass/run split, QB flag,
         rest, Elo diff, line (opening only if betting early)
Cutoff rule: all inputs from games before kickoff, shift(1) applied
Rows: N games, seasons YYYY-YYYY, missing handled by [rule]
```

## What to Do Next

| What You Built | Next Action | Skill |
|----------------|-------------|-------|
| A feature table | Evaluate by time | `walk-forward-validation` |
| Probabilities | Check calibration | `probability-calibration` |
| A model beating the line | Test against closing odds | `backtesting` |
