---
name: endzoneapi-tool
description: "Router and reference for every EndzoneAPI tool: names, parameters, credit costs, coverage by season, the response envelope, and the college football and FCS endpoints. Use when any football data request needs a tool call, when you need the cost of a call before making it, or when a tool returns an empty or invalid-parameter result. Do not use for analysis method, see the skill each tool row points to."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# EndzoneAPI Tool Reference

> **Default data tool:** EndzoneAPI. This skill is the map. Load it alongside whichever analysis skill you are using.
> Free tier: 500 credits every month, no card. Paid plans and wallet top-ups: https://endzoneapi.com/pricing
> Docs for each tool: https://endzoneapi.com/docs

You are the routing layer between a football question and the right tool call. Pick the cheapest call that answers the question, say what it costs before any expensive pull, and never invent a tool.

## When to Use

- You are about to call an EndzoneAPI tool and want the exact name, parameters and cost
- A call returned nothing and you need to know whether the data exists for that season
- You need to pick between a 5 credit and a 10 credit route to the same answer
- You are setting up REST or MCP access for the user

## When NOT to Use

- Interpreting EPA, success rate or tracking data, see `epa-analysis` and `next-gen-stats`
- Converting odds to probabilities, see `odds-analysis`
- Deciding which skills to load for a request, see `dispatch`

## Connecting

| Route | How |
|-------|-----|
| MCP (NFL) | `https://mcp.endzoneapi.com/mcp` with the key as `?key=`, an `x-api-key` header, or `Authorization: Bearer` |
| MCP (college) | `https://mcp.endzoneapi.com/mcp/ncaaf` for FBS, `https://mcp.endzoneapi.com/mcp/fcs` for FCS |
| REST (NFL) | `POST https://mcp.endzoneapi.com/v1/<tool>` with a JSON body and header `x-api-key` |
| REST (college) | `POST https://mcp.endzoneapi.com/v1/ncaaf/<tool>` or `/v1/fcs/<tool>`; `GET /v1/ncaaf/tools` lists that league's tools |

Listing tools works without a key. Calling one needs a key (https://endzoneapi.com/signup).

Every REST response is wrapped: `{"data": {...}}`. Read the payload from `data`. Errors come back as `{"error": "..."}` with the failing parameter named.

## Commands Available (NFL)

| Command | What It Does | Credits | Used by |
|---------|-------------|---------|---------|
| `list_teams` | 32 teams, conference, division, colours | 1 | `game-lookup` |
| `get_standings` | Standings with the real tiebreak step per rank | 2 | `game-lookup` |
| `get_schedule` | Unplayed games, next N days or one week | 2 | `game-lookup`, `game-preview` |
| `search_players` | Find a player, returns the GSIS and PFR ids | 2 | `player-scouting` |
| `get_injuries` | Official injury report by team and week | 2 | `game-preview`, `prop-modeling` |
| `get_depth_chart` | Depth chart by week | 2 | `game-preview`, `prop-modeling` |
| `get_games` | Games with scores, rest, starting QBs, consensus lines | 5 | `game-lookup`, `backtesting` |
| `get_team_stats` | Team season totals, EPA per play, weekly lines | 5 | `epa-analysis` |
| `get_player_stats` | One player: bio, season totals, weekly lines | 5 | `player-scouting` |
| `get_player_leaders` | Rank players on ~130 weekly stats | 5 | `player-scouting`, `epa-analysis` |
| `get_snap_counts` | Snaps and snap shares by game, player or team | 5 | `prop-modeling` |
| `get_advanced_stats` | Next Gen Stats and PFR advanced tables | 5 | `next-gen-stats` |
| `get_plays` | Play-by-play with EPA, WPA, win probability, charting | 10 | `epa-analysis`, `feature-engineering` |
| `get_game_detail` | One game: team stats, odds by book, weather, officials | 10 | `game-preview` |
| `get_head_to_head` | Two teams' meetings and record | 10 | `game-preview` |
| `get_odds` | One game's per-book lines, or a season of lines | 10 per game | `odds-explorer`, `backtesting` |
| `get_player_props` | Prop lines by book, one game or one player-season | 10 | `prop-modeling` |
| `get_prop_history` | A player's results against his prop line | 10 | `prop-modeling` |
| `get_alt_lines` | Alternate spread and total ladders | 10 | `odds-explorer` |
| `get_line_movement` | Hourly line snapshots by book | 25 | `odds-explorer` |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_team_epa` | `get_team_stats` for season EPA per play, or `get_plays` for any split |
| `get_win_probability` | `get_plays` returns `wp` and `wpa` on every play |
| `get_fantasy_projections` | `get_player_leaders` with `stat` set to `fantasy_points_ppr` for history only |
| `get_live_scores` | `get_games` with `status` set to LIVE |
| `get_weather` | `get_game_detail` carries weather and surface |
| `get_closing_line` | `get_games` (consensus) or `get_odds` (per book) |
| Team name as a parameter | Use the team code: `KC`, `BUF`, `LA` (Rams), `WAS` |

## Coverage Cheat Sheet (NFL)

| Data | From | Notes |
|------|------|-------|
| Play-by-play, EPA, win probability | 1999 | EPA and win probability model columns |
| FTN charting filters in `get_plays` | 2022 | play action, motion, RPO, screens, drops, blitzers |
| Next Gen Stats | 2016 | `week` 0 is the season aggregate |
| PFR advanced | 2018 | one row per player per game |
| Consensus closing spread and total | 1999 | moneyline from 2010 |
| Per-book closing lines | 2014 | 2014-2019 closing lines only, up to 8 books |
| Opening, closing, latest by book | 2020 | includes Pinnacle |
| Player props | 2023 | 7 books, six markets in `get_player_props` |
| Alternate lines | 2023 to 2025 | 6 books |
| Injuries 2009+, depth charts 2001+, snaps 2012+ | | |

Live coverage by season: https://endzoneapi.com/data/historical-odds, https://endzoneapi.com/data/player-props, https://endzoneapi.com/data/play-by-play

## Ids and Conventions

- **Game id:** `2025_17_DEN_KC` is season, week, away team, home team. College game ids are numeric strings.
- **Player id:** a GSIS id such as `00-0033873` from `search_players`. `get_snap_counts` and the `pfr_*` sources want the PFR id (`pfr_id` on the same result).
- **Season:** the year the regular season starts. The 2025 season ends in February 2026. In October 2026, the current season is 2026.
- **Weeks:** 1 to 18 regular season, 19 to 22 postseason (`WC`, `DIV`, `CON`, `SB` in `game_type`).
- **Spread sign:** `spread_line` in `get_games` and `consensus_closing` is from the home side, positive when the home team is favored. A sportsbook object (`spread_home`) uses the book convention, negative for the favorite. Check which one you hold before computing a cover. See `odds-explorer`.

## Cost Control

- `estimate_only: true` on `get_odds`, `get_player_props` returns the price and charges nothing. Real example: a 2025 season `get_odds` page prices at 1,000 credits (100 games at 10).
- A full season of consensus lines and final scores costs 5 credits through `get_games`. Per-book lines cost 10 per game. Use the cheap route unless you need a specific book.
- `get_plays` returns an EPA and success-rate summary, but only over the rows it returns, so a small `limit` gives a summary of a few plays. Set `limit` up to 1000 to cover a whole split in one 10 credit call.
- Do not loop `get_game_detail`. One `get_games` call returns the whole season.

## Worked Example

User: "What did KC's third-down passing look like in 2025?"

```
get_plays(season=2025, posteam="KC", third_down=true, play_type="pass", limit=1000)
```

Real response shape (trimmed): `data.plays[]` with `epa`, `wpa`, `wp`, `success`, `ydstogo`, then `data.summary` = `{"plays", "epa_total", "epa_per_play", "success_rate"}`, plus `data.next_offset`. Cost: 10 credits. Report `summary.epa_per_play` and `summary.success_rate` with `summary.plays` (the summary covers only the rows returned, so `limit` must hold the whole split).

## Anti-patterns

| Mistake | Why It's Wrong | Do This Instead |
|---------|---------------|-----------------|
| Passing `"Chiefs"` as `team` | Tools take team codes | Call `list_teams` once (1 credit) |
| Assuming odds exist before 2020 per book | Per-book opening/latest starts 2020, closing from 2014 | Check the table above |
| Asking for props before 2023 | No coverage | Say so, offer closing lines |
| Reading the response without `data` | Everything is under `data` | Unwrap first |
| Using NFL filters on college tools | `conference` takes AFC or NFC on the NFL endpoint | Call the `/v1/ncaaf/` route for college |

## What to Do Next

| What You Retrieved | Next Action | Skill |
|--------------------|-------------|-------|
| Plays or team stats | Compute EPA splits | `epa-analysis` |
| Tracking tables | Read CPOE, separation, RYOE | `next-gen-stats` |
| Odds | Remove the vig | `odds-analysis` |
| Prop lines | Project and compare | `prop-modeling` |
| College data | Handle the differences | `college-football` |
