---
name: nfl-epa-leaderboards
description: "Use when the user wants NFL rankings built on expected points added: 'who leads the league in EPA per play', 'best rushing offense by EPA', 'top receivers by receiving EPA', 'which defense is best by EPA allowed', 'EPA leaders on third down', 'rank the teams by efficiency'. Builds a leaderboard with the filters stated, the sample size per row and penalties, kneels and spikes excluded. Not for one quarterback's efficiency profile (see nfl-qb-efficiency), a game preview (see nfl-game-preview) or fourth-down decisions (see nfl-fourth-down-review)."
metadata:
  version: 2.0.0
  author: SportsDataHQ
---

# NFL EPA leaderboards

EPA per play is the standard efficiency measure for football. This skill ranks players or teams on it, with the filters and the minimum sample stated so the list can be reproduced.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server (`https://mcp.endzoneapi.com/mcp`). Call them; never fill in a number from memory.
- `season` is the year the regular season starts (2025 = the 2025-26 season). Team codes are standard abbreviations (LA, WAS, LV, LAC, JAX); `list_teams` (1 cr) if unsure.
- Player rankings use `get_player_leaders(stat, ...)` with stats such as `passing_epa`, `rushing_epa` and `receiving_epa`. These are weekly stat columns: counting stats are summed and rate stats averaged, so check what the column is before calling it per play.
- Team and situational rankings use `get_plays` (it returns EPA and success-rate summaries per query) and `get_team_stats` (EPA per play for one team and its weekly lines).
- Exclude penalties, kneels and spikes from any EPA per play you compute from plays: drop `no_play`, `qb_kneel` and `qb_spike` and say so.

## Default path (about 15 to 25 credits)
1. **Pick the ranking.** Passing, rushing, receiving or team; offense or defense; regular season (`season_type: "REG"`, the default) or postseason.
2. **Player leaderboard.** `get_player_leaders(stat, season, position, min_games, limit)` (5 cr). Use `min_games` to set a floor and state it.
3. **Team leaderboard.** `get_team_stats(team, season)` (5 cr) for each team asked about, or for one filtered cut `get_plays(season, posteam, play_type, ...)` (10 cr) and use its EPA and success-rate summary.
4. **Situations (optional).** Add `third_down`, `red_zone`, `quarter` or a `min_wp`/`max_wp` band to `get_plays`. Say the filter in the table title.
5. **Volume check.** Show games (for players) or plays (for teams) next to every rate; flag rows near the minimum.

## Reading the table
A small gap in EPA per play is noise over a partial season. Say how many games or plays each row rests on, and do not rank rows whose gap is smaller than the spread you would expect from the sample. Passing and rushing EPA are not additive across a team's schedule strength; opponent adjustment is not in these columns, so say rankings are unadjusted.

## Output template
```
<Ranking name> | <season, season type> | filters: <position, min games, situation> | exclusions: penalties, kneels, spikes
1. <name, team> <EPA metric> over <games or plays>
2. ...
Notes: <rows near the minimum, small gaps, unadjusted for opponent>
Data: get_player_leaders[, get_team_stats, get_plays] | season <yyyy>, stat <column>
```

## Rules
- State the exact stat column, the minimum-games filter and the season type.
- Do not mix per-play rates with totals in one column.
- Early-season (weeks 1-4) leaders are noisy; say so.
- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Odds and prices are information about the market only: no picks, leans, stakes, payouts or sportsbook links.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.

## Example prompts
- "Who leads the NFL in passing EPA this season?"
- "Rank the best defenses by EPA allowed per play"
- "Top receivers by receiving EPA, minimum 8 games"
