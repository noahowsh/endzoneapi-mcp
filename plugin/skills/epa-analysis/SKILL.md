---
name: epa-analysis
description: "Measures NFL team and unit efficiency with expected points added (EPA) per play, success rate, win probability added and situational splits, from play-by-play and team stats. Use when user asks about EPA, success rate, which offense or defense is best, third-down or red-zone efficiency, pass versus run efficiency, early-down tendencies, play action or motion effects, or how good a team really is. Do not use for player tracking metrics, see next-gen-stats. Do not use for score or schedule lookup, see game-lookup. Do not use for college football, which has no EPA field, see college-football."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# EPA Analysis

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`).
> `get_plays` (10 credits) returns plays plus a per-query EPA and success-rate summary. `get_team_stats` (5 credits) returns season EPA per play and weekly lines. `get_player_leaders` (5 credits) ranks players on `passing_epa`, `rushing_epa`, `receiving_epa`.

You are an expert in football efficiency metrics. Your goal is to answer "how good is this offense or defense" with a rate, a sample size, and a split that explains it.

## When to Use

- "Which offenses were best on third down in 2025?"
- "Is Kansas City's problem passing or running?"
- "How does play action change efficiency?"
- "What is a team's EPA per play when trailing?"
- "Rank quarterbacks by EPA"

## When NOT to Use

- Separation, pressure, time to throw, see `next-gen-stats`
- Win/loss records and schedules, see `game-lookup`
- FBS or FCS, which do not carry EPA, see `college-football`
- Turning EPA into a prediction model, see `feature-engineering`

## Commands Available

| Command | What It Does | Credits |
|---------|-------------|---------|
| `get_plays` | Plays with `epa`, `wpa`, `wp`, `success`, `yards_gained`; filters for down, quarter, play type, red zone, win probability band, charting flags; returns `summary` | 10 |
| `get_team_stats` | `totals.passing_epa_per_play`, `rushing_epa_per_play`, yards per attempt and carry, plus `weeks[]` | 5 |
| `get_player_leaders` | Rank by `passing_epa`, `rushing_epa`, `receiving_epa` | 5 |
| `get_games` | Join game results to your EPA numbers | 5 |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_team_epa` | `get_team_stats`, or `get_plays` with `posteam` and read `summary.epa_per_play` |
| `get_defense_epa` | `get_plays` with `defteam` set instead of `posteam` |
| `get_epa_by_situation` | Filter `get_plays` (`down`, `quarter`, `red_zone`, `third_down`, `min_wp`, `max_wp`) |
| `get_success_rate` | `summary.success_rate` in `get_plays` |
| `get_cpoe` | `get_plays` returns CPOE per play; `get_advanced_stats` with an ngs_passing source has season CPOE |

## Definitions

- **EPA (expected points added):** points the offense is expected to score on the drive after the play minus before it. A 6 yard gain on 3rd and 10 can be negative.
- **EPA per play:** mean EPA over a set of plays. The unit is points per play. League-average offense sits near zero by construction; 0.1 per play is a very good offense, -0.1 a bad one.
- **Success:** a binary per play (`success` 1 or 0). Success rate is the mean. It ignores play size, so it is steadier than EPA and misses explosive plays.
- **WPA:** change in win probability from the play. Context-dependent; do not use to rate talent.
- **CPOE:** completion probability over expected, in percentage points, on pass plays.

Real example from the data: the first 2025 KC third-down pass returned was 3rd and 6, incomplete, `epa` -1.57, `wpa` -0.048, `success` 0. A 40-play pull of that split gave `epa_per_play` -0.112 and `success_rate` 0.375. Forty plays is a sample, not a verdict; it shows the shape.

## How It Works

**Step 1: Choose the scope.** Whole-season rate for a team: `get_team_stats(team="KC", season=2025)`. Any split: `get_plays` with filters.

**Step 2: Pull the split with the summary.**

```
get_plays(season=2025, posteam="KC", third_down=true, play_type="pass", limit=1000)
```

Cost 10. Read `data.summary` = `{plays, epa_total, epa_per_play, success_rate}`. **The summary covers only the rows returned.** Tested live: with `limit` 1 the summary reported 1 play, with `limit` 40 it reported 40 (EPA per play -0.112, success rate 0.375 on those first 40 KC third-down passes of 2025). So set `limit` high enough (up to 1000) to hold the whole split, and check that `summary.plays` equals the count you expect before quoting a rate. If a split exceeds 1,000 plays, page with `offset` and `next_offset` and combine `epa_total` and `plays` yourself; never average the per-page rates.

**Step 3: Always compute both sides.** Offense is `posteam`. Defense is the same call with `defteam`. A defense's EPA allowed is the opponent's offensive EPA against it.

**Step 4: Split to explain.**

| Split | Filter | What It Shows |
|-------|--------|---------------|
| Pass vs run | `play_type` = pass or run | Where the efficiency comes from |
| Early down | `down` = 1 or 2 | Play-calling quality, sustainable |
| Third down | `third_down` = true | Noisier; depends on distance |
| Red zone | `red_zone` = true | Small samples, high leverage |
| Neutral situations | `min_wp` 0.2, `max_wp` 0.8 | Removes garbage time and clock-killing |
| Charting (2022+) | `is_play_action`, `is_motion`, `is_rpo`, `is_screen_pass`, `is_no_huddle`, `include_charting` | Scheme effects |

**Step 5: Exclude the noise.** Sacks, penalties and kneels are plays. Decide whether to keep them and say which. Filtering `play_type` to pass or run mixes in what the source labels as pass plays, which includes sacks; state it. Use the neutral win probability band for any "true talent" claim.

**Step 6: Judge the sample.** A team plays about 1,000 offensive plays a season. A split with 60 plays has a standard error roughly 0.1 EPA per play wide (play-level EPA standard deviation is near 1.3 to 1.5). Say "n = 60" next to every split. Third-down splits for one season are descriptive, not predictive.

**Step 7: Compare to league.** One call per team is expensive. For league context use `get_player_leaders` (5 credits) for player ranks, or compare a team to its own prior weeks via `weeks[]`.

## Season Data Check

Real figures from the docs example for 2025 Kansas City (6-11): `passing_epa_per_play` 0.039, `rushing_epa_per_play` 0.020, `yards_per_attempt` 6.76, `yards_per_carry` 4.21, 3,947 passing yards. The same payload lists `passing_epa` 24.9 and `receiving_epa` 109.3 for the team. They are computed on different play sets, so do not expect them to match. Pick one definition and cite it.

## Credit Usage

| Operation | Credits |
|-----------|---------|
| Team season efficiency | 5 |
| One situational split (up to 1,000 plays) | 10 |
| Offense and defense split for one team | 20 |
| All 32 teams, one split (offense only) | 320 |
| Player EPA leaderboard | 5 |

For league-wide work, filter the heavy `get_plays` pulls to one split and tell the user the cost first.

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "Third-down EPA shows clutch" | Year-to-year correlation is low | Use early-down EPA for prediction |
| "Better record means better EPA" | Fumbles, field position, kicking | Show both |
| "Mean EPA per play of a split of 40 plays" | Standard error is huge | Show n and a range |
| "Compare 2019 EPA to 2025 EPA directly" | League passing environment shifts | Compare to the league that year |
| "A QB's EPA is the QB's" | Includes line, receivers, scheme | Say so, or compare backups |

## Output Format

```
KC offense, 2025 regular season
Pass EPA/play 0.039 | Run EPA/play 0.020 (season totals)
3rd down passing: EPA/play X, success X% (n = N plays)
Read: [one sentence, with the weakest split named]
Caveat: [sample, starting QB, garbage time]
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| A team-level EPA gap | Add to a model | `feature-engineering` |
| A QB with high EPA | Check tracking data | `next-gen-stats` |
| An efficiency mismatch | Preview the matchup | `game-preview` |
| A rate to carry forward | Use rolling windows | `feature-engineering` |

## Reference

Python walkthrough: EPA per play: https://endzoneapi.com/guides/nfl-epa-per-play-python
