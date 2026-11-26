---
name: nfl-game-preview
description: "Use when the user wants a preview or breakdown of one NFL game: 'preview Bills vs Chiefs', 'break down Sunday's Eagles game', 'what should I know about the Ravens game', 'who's hurt for the Lions game', 'matchup analysis for TNF'. Produces a one-page report: records and form, efficiency (EPA, success rate), injuries, the line and what it implies, and head-to-head. Not for a whole week's slate of player props (see nfl-player-props-research), fantasy usage (see nfl-weekly-usage-report) or building a model (see model-building)."
metadata:
  version: 2.0.0
  author: SportsDataHQ
---

# NFL game preview

A shareable one-page preview of a single game, built from live EndzoneAPI calls. It covers the NFL; college games are not available through these tools, so say so if asked.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server (`https://mcp.endzoneapi.com/mcp`). Call them; never fill in a number from memory.
- `season` is the year the regular season starts (2025 = the 2025-26 season). Game ids look like `2025_12_BUF_KC` (season, week, away, home).
- Team codes are standard abbreviations (LA for the Rams, WAS, LV, LAC, JAX). Use `list_teams` if unsure.

## Default path (about 31 credits)
1. **Find the game.** `get_schedule(team, week)` (2 cr) returns the game id, kickoff and the current line. For a past game use `get_games(season, week, team)` (5 cr).
2. **The game file.** `get_game_detail(game_id)` (10 cr): both teams, team EPA and success rate so far, consensus and per-book lines, weather, surface, officials.
3. **Records and form.** `get_standings(season)` (2 cr): records, streaks, point differential.
4. **Injuries.** `get_injuries(team, week)` (2 cr) for each team: Out, Doubtful, Questionable, with practice participation. Flag starters (QB, OL, top receivers, edge rushers, CBs) using `get_depth_chart(team, week)` (2 cr) when it matters.
5. **What the market implies.** `get_game_probabilities(game_id)` (5 cr): average implied and no-margin win probability, spread and total. Report it as the market's view, not a prediction of ours.
6. **History (optional).** `get_head_to_head(team1, team2)` (10 cr) only if the user asks or the teams met recently; head-to-head records are weak predictors, say so.

## Output template
```
<AWAY> (<record>) at <HOME> (<record>) | <date, kickoff ET> | <venue, surface, weather>
Line: <spread> / total <total> (consensus close or latest) | Market: <home no-margin win %>
Form: <last 3-5 results, point differential>
Efficiency (season to date): offense EPA/play and success rate, defense EPA/play allowed, for both teams
Injuries that matter: <player, position, status> (practice: DNP/limited/full)
What to watch: 2-4 bullets tied to the numbers above (e.g. pass rush vs a backup tackle)
Data: get_schedule, get_game_detail, get_standings, get_injuries, get_game_probabilities | season <yyyy>, week <n>
```

## Rules
- Every number names its tool (the template's Data line, plus inline where it could be doubted).
- State sample size for rates ("EPA/play over 11 games"). Early-season (weeks 1-4) efficiency is noisy; say so.
- EPA excludes penalties, kneels and spikes; it will not match official box-score yards.
- Odds are information about the market. Never suggest a side, a stake or a bet.
- If a tool returns nothing (game not scheduled yet, no line posted), say what is missing instead of estimating it.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.

## Example prompts
- "Preview Bills at Chiefs this Sunday"
- "Who's out for the Eagles game and what's the line?"
