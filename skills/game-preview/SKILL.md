---
name: game-preview
description: "Builds a pre-game preview for one NFL matchup: schedule, form, efficiency, head-to-head, injuries, depth chart, rest, weather and betting lines in one brief. Use when user asks to preview a game, break down a matchup, what to know before Sunday, or who has the edge in a specific game. Do not use for a full model, see feature-engineering. Do not use for price shopping, see odds-explorer. Do not use for college games, see college-football."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Game Preview

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`).
> Budget: about 45 credits for a full preview. A short version (schedule, team EPA for both sides, lines) is about 17.

You are a football analyst writing a preview. Your goal is a one-page brief that says who has the edge, why, how large the market thinks it is, and what could change it.

## When to Use

- "Preview Bills at Chiefs"
- "What should I know about this week's Eagles game?"
- "Who has the advantage in this matchup?"
- "Break down the Sunday night game"

## When NOT to Use

- Building a predictive model, see `feature-engineering` and `walk-forward-validation`
- Only the line, see `odds-explorer`
- Only one team's profile, see `epa-analysis`
- Player-specific questions, see `player-scouting`

## Commands Available

| Command | What It Does | Credits |
|---------|-------------|---------|
| `get_schedule` | Find the game, kickoff, consensus line | 2 |
| `get_team_stats` | Season EPA per play and totals, one team | 5 |
| `get_head_to_head` | Past meetings, record, average margin | 10 |
| `get_injuries` | Status by team | 2 |
| `get_depth_chart` | Starters | 2 |
| `get_game_detail` | Weather, surface, officials, per-book lines | 10 |
| `get_games` | Recent results with starting QBs and rest | 5 |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_game_preview` | This skill, assembled from the calls above |
| `get_matchup_rankings` | Compute from two `get_team_stats` calls |
| `get_win_probability_pregame` | The consensus line is the market's estimate, see `odds-analysis` |
| `get_referee_stats` | Not available; `get_game_detail` names the officials only |

## How It Works

**Step 1: Locate the game.** `get_schedule(team="BUF")` returns the next games with `game_id`. For a past game or a specific week, `get_games(season=2025, week=5, team="BUF")`.

**Step 2: Efficiency for both teams.** `get_team_stats(team="BUF", season=2025)` and the same for the opponent: `totals.passing_epa_per_play`, `totals.rushing_epa_per_play`, `yards_per_attempt`, `yards_per_carry`, plus `weeks[]` for trend. Early in a season, use the last season as a prior and say how many 2026 games exist.

**Step 3: Form and context.** `get_games(season=2025, team="BUF", limit=5)` is the last results with `away_rest`, `home_rest`, `away_qb_name`, `home_qb_name`. A team that has changed starting quarterback has a different recent form than its season line.

**Step 4: Availability.** `get_injuries(team="BUF")` and `get_depth_chart(team="BUF", formation="Offense")`. Report only players that change the matchup: the quarterback, top two receivers, the left tackle, the top pass rusher. The injury report is incomplete until the Wednesday to Friday practice reports are in.

**Step 5: History.** `get_head_to_head(team1="BUF", team2="KC", limit=5)`. Head-to-head carries little predictive weight across rosters and coaches; use it for color, never as the argument. Say so.

**Step 6: Market.** The consensus line from the schedule row, then `get_game_detail(game_id=...)` for weather, surface and per-book numbers. Convert the spread into an implied margin and the total into implied team totals:

```
home_implied = (total + spread_line) / 2
away_implied = (total - spread_line) / 2
```

where `spread_line` is positive when home is favored. Check with DEN at KC: total 37.5, `spread_line` -13.5, so KC implied 12.0, DEN implied 25.5. KC scored 13, DEN 20.

**Step 7: Write it.** Lead with a one-sentence read, then the evidence in this order: efficiency, availability, situation (rest, weather), market, history.

## What Moves the Needle

| Factor | How to Use It |
|--------|---------------|
| Starting quarterback | Largest single input, always confirm |
| Pass versus run EPA gap | A team that passes well against a defense weak in pass EPA allowed |
| Rest | Short week or bye, from rest days |
| Wind and surface | Strong wind tends to hurt the passing game; check `get_game_detail` |
| Injuries to the offensive line | Show up in sacks and pressure, see `next-gen-stats` |

## Credit Usage

| Operation | Credits |
|-----------|---------|
| Find game | 2 |
| Team stats x2 | 10 |
| Recent results x2 | 10 |
| Injuries x2 | 4 |
| Depth charts x2 | 4 |
| Head-to-head | 10 |
| Game detail | 10 |
| Total | 50 |

Skip head-to-head and depth charts to bring it near 36. Ask before running the full set during a busy slate.

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "They are 4-1 so they are better" | Records hide point differential and schedule | Show EPA per play next to the record |
| "They won the last three meetings" | Rosters turn over | Mention it once, weight it near zero |
| "The model says X" | You have not built one | Say what the line implies, not what you predict |
| "Injury report is empty" | It may not be published yet | State the report week and date |

## Output Format

```
BUF @ KC -- Sunday, Week 5
Read: [one sentence]
Efficiency: [away] pass EPA/play X, [home] pass EPA/play X (2025; 2026 sample: N games)
Availability: [QB and key names with status]
Situation: rest [away] X / [home] X, [weather]
Market: [spread and total] (implied [away] X, [home] X)
Risk to the read: [one concrete thing]
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| A view that differs from the line | Quantify and size | `edge-detection` |
| Player-specific angle | Prop projection | `prop-modeling` |
| Need the price across books | Shop | `odds-explorer` |
