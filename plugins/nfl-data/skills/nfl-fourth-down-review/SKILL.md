---
name: nfl-fourth-down-review
description: "Use when the user wants to review fourth-down decisions in the NFL: 'what did the Lions do on fourth down', 'how often does Baltimore go for it', 'review that fourth-and-2 call', 'fourth down tendencies this season', 'did they punt in plus territory', 'go-for-it rate when trailing'. Pulls fourth-down plays with the situation filters and reports what teams did and what the numbers say about those plays (conversion, EPA, win probability). Not for a game preview (see nfl-game-preview), league EPA rankings (see nfl-epa-leaderboards) or building a decision model (see model-building)."
metadata:
  version: 2.0.0
  author: SportsDataHQ
---

# NFL fourth-down review

Fourth-down decisions are easy to argue about and easy to measure. This skill lays out the decision context for each play and what happened, from EndzoneAPI play-by-play. It reports the numbers the API returns; it does not grade a coach beyond them.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server (`https://mcp.endzoneapi.com/mcp`). Call them; never fill in a number from memory.
- `season` is the year the regular season starts (2025 = the 2025-26 season). Team codes are standard abbreviations (LA, WAS, LV, LAC, JAX); `list_teams` (1 cr) if unsure.
- `get_plays` filters are `season`, `week`, `posteam`, `defteam`, `down`, `quarter`, `min_wp`/`max_wp` (pre-play win probability for the offense), `red_zone`, `play_type` and `game_id`. Each call also returns a per-query EPA and success-rate summary. Paginate with `offset` (up to 1,000 plays a page).
- Classify the decision from the play: pass or run is a go for it, punt is a punt, field_goal is a kick. Penalties and no-plays are not decisions; exclude them and say how many were dropped.
- Read the yard line and yards to go from the rows themselves. If a field is missing from a row, say the context is incomplete instead of reconstructing it.

## Default path (about 20 to 30 credits)
1. **Set the scope.** One team-season, one game, or a league-wide filter (for example `down: 4`, `max_wp: 0.3`). Say the filter back to the user.
2. **Pull the plays.** `get_plays(season, posteam, down: 4)` (10 cr per page). Add `quarter`, `min_wp`/`max_wp` or `red_zone` for situations; for one game use `game_id`.
3. **Count the decisions.** Go, punt and kick counts, then go-for-it rate = go / (go + punt + kick) with the denominator shown.
4. **Outcomes.** For go-for-it plays: conversion rate, and EPA and success rate from the query summary. For punts and kicks: the same summary fields where returned.
5. **One play in depth (when the user names one).** `get_plays(game_id, quarter)` (10 cr) for the play and its neighbours; report down, distance, field position, score, time and pre-play win probability as returned.
6. **League context (optional).** Repeat step 2 without `posteam` or with `defteam` swapped to compare, or `get_team_stats(team, season)` (5 cr) for the team's overall EPA per play.

## What the numbers can and cannot say
State the decision context first (down, distance, field position, score, time, win probability), then what happened and what similar plays did in the sample. Do not label a call right or wrong from one outcome: a converted fourth down can still have been a poor decision and the reverse. Do not recommend what a coach should do; say what the sample shows. Samples of fewer than 20 plays in a situation are descriptive only.

## Output template
```
<Team or scope> | fourth down | <season, filters> | plays counted <n> (penalties and no-plays excluded: <k>)
Decisions: go <a> (<%>), punt <b>, field goal <c>
Go for it: converted <x>/<a> (<%>), EPA per play <v>, success rate <%>
Situation split: <e.g. trailing, win probability under 30%: go <a>/<n>; opponent territory: ...>
Play in question (if asked): <down and distance, yard line, score, clock, pre-play win probability, result>
Read: 2-3 bullets stating what the sample shows and its size
Data: get_plays[, get_team_stats] | season <yyyy>, filters <list>
```

## Rules
- EPA excludes kneels and spikes; it will not match official box-score yards.
- Early-season samples (weeks 1-4) are small; say so.
- Describe, do not instruct: no "should have gone" framing beyond what the returned numbers support.
- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Odds and prices are information about the market only: no picks, leans, stakes, payouts or sportsbook links.

## Example prompts
- "What did the Ravens do on fourth down this season?"
- "How often do teams go for it when trailing in the fourth quarter?"
- "Review the fourth-and-1 in the Lions game last week"
