---
name: college-football
description: "Works with EndzoneAPI's FBS (NCAAF) and FCS college football data: games, rankings at kickoff, conference standings, Elo, play-by-play, player lines, and sportsbook lines. Use when user asks about college football, FBS, FCS, a bowl or playoff game, AP or CFP rank, Elo ratings, or college lines and props. Do not use for NFL questions, see game-lookup. Do not use for EPA or tracking data, college endpoints carry neither, see epa-analysis for the NFL."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# College Football (FBS and FCS)

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`), on the college routes.
> FBS: `POST https://mcp.endzoneapi.com/v1/ncaaf/<tool>` or MCP at `https://mcp.endzoneapi.com/mcp/ncaaf`.
> FCS: `POST https://mcp.endzoneapi.com/v1/fcs/<tool>` or MCP at `https://mcp.endzoneapi.com/mcp/fcs`.
> Same key, same credit balance and the same price per tool as the NFL. Free tier: 500 credits every month.

You are a college football analyst. Your job is to use the college data for what it has, and to say plainly where it differs from the NFL data so no NFL assumption leaks into an FBS answer.

## When to Use

- "Which ranked teams played in week 1 of 2025?"
- "Show Montana State's FCS season"
- "What was Kansas State's Elo?"
- "Bowl and playoff games with closing lines"
- "College player props" (FBS only)

## When NOT to Use

- NFL schedules and standings, see `game-lookup`
- EPA, success rate, Next Gen Stats, injuries, depth charts, snap counts: none exist for college, see `epa-analysis` and `next-gen-stats` for the NFL
- General betting math, see `odds-analysis`

## Coverage (from the site's own copy)

| Data | FBS | FCS |
|------|-----|-----|
| Games | since 2004, 18,769 games, with AP and CFP rank of both teams at kickoff and bowl or playoff round | since 2004, 15,356 games; 2020 spring season included |
| Plays | 3.27M plays, win probability after each play from 2016 | 1.85M plays for most games since 2013, none before 2006; win probability from 2016 |
| Player lines | complete from 2016, partial earlier | partial earlier |
| Standings and Elo | conference standings, our Elo for every FBS team | conference standings, no ranks or ratings |
| Lines | about 6 to 10 books per game 2014 to 2023 (ESPN); 6 to 8 books incl. Pinnacle 2020 to 2025 (The Odds API) | ESPN lines from 2017, uneven coverage; 6 books for 2026 captured since August 28 2026 |
| Player props | passing, rushing, receiving yards, passing TDs, receptions, anytime TD, 2023 to 2025 | none |
| Alternate lines | seven books, 2023 to 2025 | none |

Game counts and play counts change as seasons load. Current numbers: https://endzoneapi.com/college-football and https://endzoneapi.com/fcs-football

## Commands Available

FBS (15 tools, read live from `GET /v1/ncaaf/tools`):

| Command | Parameters that differ from the NFL | Credits |
|---------|-------------------------------------|---------|
| `list_teams` | `season`, `conference` (a college conference name) | 1 |
| `get_games` | `season_type`, `ranked`, `offset`; no `game_type` | 5 |
| `get_schedule` | `team`, `days`; no `week` | 2 |
| `get_standings` | `season`, `conference` | 2 |
| `get_team_stats` | `team`, `season`, `season_type`; per-game averages and `elo` | 5 |
| `search_players` | `query`, `team`, `position` | 2 |
| `get_player_stats` | `player_id`, `season`, `season_type` | 5 |
| `get_player_leaders` | `stat`, `season`, `season_type`, `team`, `position`, `min_games` | 5 |
| `get_plays` | `scoring`, `quarter`, `down`, `red_zone`, `third_down`, `posteam`, `defteam` | 10 |
| `get_game_detail`, `get_head_to_head` | same as NFL | 10 |
| `get_odds` | `game_id`, `season`, `week`, `team` | 10 per game |
| `get_player_props`, `get_alt_lines` | same shape as NFL | 10 |
| `get_line_movement` | `game_id`, `bookmaker` | 25 |

FCS has 13 tools: the same list without `get_player_props` and `get_alt_lines`, and `get_games` has no `ranked` flag.

## Commands That Do NOT Exist (college)

| Not Available | Use Instead |
|--------------|-------------|
| `get_advanced_stats` | No tracking data for college |
| `get_injuries`, `get_depth_chart`, `get_snap_counts` | None for college; check team sources |
| EPA fields in plays | `get_plays` returns yards, `scoring_play` and `home_win_prob`; build your own value model if needed |
| `get_prop_history` | Not in the college tool list |
| `ranked` on the FCS route | FCS has no AP or CFP rank in this data |

## Differences That Break NFL Habits

| NFL | College |
|-----|---------|
| Team code `KC` | Abbreviation like `KSU`, `ISU`, `MTST`; use `list_teams` or a `get_games` row |
| Game id `2025_01_KC_LAC` | Numeric string like `401756846` |
| Week 1 to 22 | Regular season weeks plus bowls; `season_type` is `regular` or `postseason` |
| `home_score`, `away_score` flat fields | Nested `home` and `away` objects with `abbrev`, `rank`, `score` |
| EPA and success per play | None. `get_plays` has `type`, `text`, `down`, `distance`, `yards_to_endzone`, `yards`, `scoring_play`, `home_win_prob` |
| Conference is AFC or NFC | Conference names such as Big Sky; the NFL route rejects them |
| 32 teams | Hundreds of teams; FBS vs FCS games are filed under FBS |

An FCS team's game against an FBS opponent is in the FBS data, not in FCS.

## How It Works

**Step 1: Pick the league route.** FBS or FCS. All answers on that route are about that league.

**Step 2: Find the game.** Real call and shape:

```
get_games(season=2025, ranked=true, limit=2)
```

returns `data.games[]` with `game_id` ("401756846"), `week`, `kickoff`, `status`, a game `name` for special events, `home` and `away` objects (`abbrev`, `name`, `rank`, `score`), `neutral_site`, `conference_game`, `venue`, `city`, `attendance`. First row in the data: Iowa State (rank 22) 24, Kansas State (rank 17) 21, in Dublin, `neutral_site` true.

**Step 3: Team strength.** `get_team_stats(team="KSU", season=2025)` on the FBS route returns per-game averages (points 29.4, points allowed 26.7, yards 370.1, third-down 0.372, turnovers, takeaways) and `elo` (1594.5). The stats are averages per game, not per play. Elo is the site's own rating; say it is not an official ranking.

**Step 4: Plays.** `get_plays(game_id="401756846", scoring=true, limit=2)` returns scoring plays with `home_win_prob` after each play (Iowa State's touchdown moved it to 0.43 for the home side in the real data). Win probability from 2016.

**Step 5: Lines.** `get_odds(game_id="401756846")` for one game. Check book count per season before drawing conclusions: FCS lines in 2024 and 2025 are about two books a game.

## Pitfalls

- Rank is at kickoff. Do not backfill it from a later poll.
- ESPN lines and Odds API lines come from different sources. Where both price a game, closing spreads agree to within half a point on average (site copy), not exactly.
- Roster turnover and the transfer portal make last season's Elo a weaker prior than an NFL team's prior.
- Single-game samples are tiny: 12 to 13 regular season games per team.
- Missing line means no line was posted, not zero.

## Credit Usage

Same as the NFL tool of the same name. A full FBS season of games is 5 credits per query; lines are 10 per game.

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "Compute EPA from `get_plays`" | No EPA field, only yards and win probability | Use win probability change as a proxy and label it |
| "Pass the NFL team code" | College uses ESPN-style abbreviations | Pull from a `get_games` row |
| "FCS has props" | It does not | Say so |
| "Elo is a ranking" | It is a rating computed on our data | Present it as a strength estimate |

## Output Format

```
FBS 2025, Week 1: Iowa State (22) 24, Kansas State (17) 21, Dublin (neutral site)
Strength entering: [Elo for each if pulled]
Market: [closing spread and total, source and book count]
Caveat: [sample, line source]
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| A season of lines and results | Test a strategy | `backtesting` |
| Lines for a game | Remove vig | `odds-analysis` |
| Ratings | Build features | `feature-engineering` |
