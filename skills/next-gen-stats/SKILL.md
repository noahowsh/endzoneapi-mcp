---
name: next-gen-stats
description: "Reads Next Gen Stats tracking data and Pro Football Reference advanced stats for NFL passers, rushers and receivers: time to throw, air yards, CPOE, rush yards over expected, separation, cushion, yards after catch above expectation, pressures, drops, bad throws. Use when user asks about separation, time to throw, rush yards over expected, pressure rate, drop rate, or why a player's box score does not match how he plays. Do not use for EPA or success rate, see epa-analysis. Do not use for season totals and rankings, see player-scouting. Do not use for college football, which has no tracking data."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Next Gen Stats and Advanced Tracking

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`).
> One tool: `get_advanced_stats` (5 credits per call). Choose a `source`, then filter by `season`, `week`, `player_id`, `team`, `limit`.

You are an expert in player tracking and charting metrics. Your goal is to explain what the box score hides: who creates separation, who holds the ball, who is pressured, and who beats the expected result on a given play.

## When to Use

- "Does this quarterback hold the ball too long?"
- "Which backs gain the most yards over expected?"
- "Who gets open?" / "Who drops passes?"
- "Is his completion percentage lucky?"
- "Which quarterbacks are pressured most?"

## When NOT to Use

- Team efficiency on every play, see `epa-analysis`
- Counting stats and leaderboards, see `player-scouting`
- Anything about college or FCS players, those leagues carry no tracking data, see `college-football`

## Commands Available

| Command | What It Does | Credits |
|---------|-------------|---------|
| `get_advanced_stats` | `source` plus filters, returns `metric_keys` and rows with a `metrics` object | 5 |
| `search_players` | Get the id the source needs | 2 |

## Sources

Verified by live calls (2025 season):

| `source` | From | Id to pass | Rows | `metrics` keys |
|----------|------|-----------|------|----------------|
| `ngs_passing` | 2016 | GSIS (`00-...`) | player-week; `week` 0 is season | attempts, completions, completion_percentage, expected_completion_percentage, completion_percentage_above_expectation, avg_time_to_throw, avg_intended_air_yards, avg_completed_air_yards, avg_air_yards_differential, avg_air_yards_to_sticks, aggressiveness, max_air_distance, passer_rating, pass_yards, pass_touchdowns, interceptions |
| `ngs_rushing` | 2016 | GSIS | player-week | rush_attempts, rush_yards, expected_rush_yards, rush_yards_over_expected, rush_yards_over_expected_per_att, rush_pct_over_expected, efficiency, avg_time_to_los, percent_attempts_gte_eight_defenders, rush_touchdowns |
| `ngs_receiving` | 2016 | GSIS | player-week | targets, receptions, yards, catch_percentage, avg_separation, avg_cushion, avg_intended_air_yards, percent_share_of_intended_air_yards, avg_yac, avg_expected_yac, avg_yac_above_expectation, rec_touchdowns |
| `pfr_passing` | 2018 | PFR id (`MahoPa00`) | player-game | times_pressured, times_pressured_pct, times_hurried, times_hit, times_sacked, times_blitzed, passing_drops, passing_drop_pct, passing_bad_throws, passing_bad_throw_pct |
| `pfr_receiving` | 2018 | PFR | player-game | receiving_drop, receiving_drop_pct, receiving_broken_tackles, receiving_int, receiving_rat |
| `pfr_rushing`, `pfr_defense` | 2018 | PFR | player-game | see `metric_keys` in the response |

The response always lists `metric_keys` and `player_id_kind` (gsis or pfr). Read them rather than guessing. The `pfr_*` rows also carry `game_id`, `opponent`.

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_player_tracking` | `get_advanced_stats` |
| `get_pressure_rate` | `get_advanced_stats` with `source` pfr_passing, `times_pressured_pct` |
| `get_route_data` | Not available; FTN charting flags are in `get_plays` (2022+) |
| `get_coverage_stats` | `get_advanced_stats` with `source` pfr_defense |

## How It Works

**Step 1: Match the id to the source.** `search_players` returns `player_id` (GSIS) and `pfr_id`. `ngs_*` wants GSIS. `pfr_*` wants PFR. Check `player_id_kind` in the response if rows come back empty.

**Step 2: Pull the season aggregate.** `get_advanced_stats(source="ngs_passing", season=2025, week=0, limit=40)`. `week=0` is the season aggregate for NGS. For PFR, rows are per game, so sum or average them yourself and weight by attempts.

Real row (trimmed): Aaron Rodgers, PIT, 498 attempts, 327 completions, `avg_time_to_throw` 2.59 seconds, `avg_intended_air_yards` 6.0, `completion_percentage` 65.7, `expected_completion_percentage` 66.6, `completion_percentage_above_expectation` -0.95.

**Step 3: Read the metrics in pairs.**

| Pair | Reading |
|------|---------|
| `completion_percentage` vs `expected_completion_percentage` | The gap is CPOE. Positive means he completes harder throws than the model expects. |
| `avg_time_to_throw` vs `times_pressured_pct` | Long time to throw with high pressure points at the quarterback or the line; short time with high pressure points at the line |
| `avg_intended_air_yards` vs `avg_air_yards_to_sticks` | Depth of target against the first-down marker; negative means throwing short of the sticks |
| `rush_yards_over_expected_per_att` vs `rush_yards` | Back's value separate from blocking |
| `avg_separation` vs `avg_cushion` | Release versus defender's alignment |
| `avg_yac` vs `avg_expected_yac` | After-catch skill above the model |

**Step 4: Weight by volume.** Season aggregates are given per player; for a fair comparison require a floor: at least 200 attempts for passers, 100 carries for rushers, 50 targets for receivers. Below those the metrics move on a handful of plays.

**Step 5: Combine with outcomes.** Tracking explains; EPA scores. A receiver with elite separation and weak EPA is an offense or quarterback problem. A receiver with poor separation and strong EPA is living on contested catches and may regress.

## Pitfalls

- `week=0` is not week 0 of the season; it is the aggregate. Do not average it with weekly rows.
- A metric computed by the source (CPOE, RYOE) is a model residual. Its noise is large in a single season, so a 1 point CPOE gap on 400 attempts does not separate two quarterbacks.
- Pressure counts depend on both the line and the quarterback's time to throw. Do not assign them to the line alone.
- NGS and PFR are different providers with different definitions. Do not mix their columns in one rate.
- Seasons before 2022 have no FTN play charting in `get_plays`, so scheme context is thinner.

## Credit Usage

| Operation | Credits |
|-----------|---------|
| One source, one season, 50 rows | 5 |
| Season aggregate plus a player's weeks | 10 |
| Passing, rushing and receiving season views | 15 |
| One player across NGS and PFR | 10 to 15, plus 2 for the id |

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "He has negative CPOE, he is bad" | One season of CPOE is noisy and depends on receivers | Show it beside EPA and a second year |
| "Separation proves a good receiver" | Scheme and coverage affect it | Pair with targets and EPA |
| "Pass the PFR id to NGS" | The sources use different id kinds | Check `player_id_kind` |
| "Average weekly rows to get a season rate" | Weeks have different volumes | Use `week=0` for NGS, weight PFR by attempts |

## Output Format

```
Aaron Rodgers, PIT -- NGS passing, 2025 (498 att)
Time to throw 2.59s | Intended air yards 6.0 | CPOE -0.95
Reading: [one sentence tying tracking to outcome]
Caveat: [sample, supporting cast]
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| A tracking edge | Test it as a feature | `feature-engineering` |
| High RYOE back | Project a rushing prop | `prop-modeling` |
| Pressure-heavy quarterback | Preview against a pass rush | `game-preview` |

## Reference

Python walkthrough: Next Gen Stats: https://endzoneapi.com/guides/nfl-next-gen-stats-python
