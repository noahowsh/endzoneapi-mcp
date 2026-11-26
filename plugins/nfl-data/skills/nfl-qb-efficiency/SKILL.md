---
name: nfl-qb-efficiency
description: "Use when the user wants an efficiency profile of an NFL quarterback or a comparison of two: 'how efficient is Jordan Love', 'compare Burrow and Herbert', 'is Daniels' pressure rate a problem', 'QB EPA and success rate', 'is he better than his completion percentage', 'CPOE for Purdy'. Reports EPA per dropback, success rate, completion percentage above expectation where the data has it, air yards, time to throw and pressure splits with sample sizes. Not for QB prop lines (see nfl-player-props-research), league-wide EPA lists (see nfl-epa-leaderboards) or a game preview (see nfl-game-preview)."
metadata:
  version: 2.2.0
  author: SportsDataHQ
---

# NFL quarterback efficiency

Passer rating hides most of what matters. This skill profiles one quarterback, or compares two, on EPA, success rate and the tracking-based fields EndzoneAPI carries.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server (`https://mcp.endzoneapi.com/mcp`). Call them; never fill in a number from memory.
- `season` is the year the regular season starts (2025 = the 2025-26 season). Team codes are standard abbreviations (LA, WAS, LV, LAC, JAX); `list_teams` (1 cr) if unsure.
- Player ids are GSIS ids from `search_players` (2 cr). Next Gen Stats start in 2016 and the Pro Football Reference advanced tables in 2018; check the season is covered.
- Completion percentage above expectation (CPOE) comes from two places: `get_advanced_stats(source: "tracking_passing")` and the play-level fields on `get_plays`. Use whichever the tool returned and name it. If a query returns no CPOE field, say CPOE is not in that result and leave it out; do not compute it yourself.
- Pressure and hurry counts come from `get_advanced_stats(source: "charting_passing")`. Read the column names the tool returns before quoting a rate.

## Default path (about 22 credits per quarterback)
1. **Resolve the player.** `search_players(query)` (2 cr): GSIS id and PFR id.
2. **Season line.** `get_player_leaders(stat: "passing_epa", position: "QB", season, min_games)` (5 cr) for rank among qualifying quarterbacks, or `get_player_stats(player_id, season)` (5 cr) for the season totals and weekly lines.
3. **Tracking fields.** `get_advanced_stats(source: "tracking_passing", player_id, season)` (5 cr): time to throw, air yards, CPOE where present. Add `source: "charting_passing"` (5 cr) for pressure, drops and bad throws.
4. **Situations (optional).** `get_plays(season, passer_id, down or red_zone or third_down)` (10 cr): the query's EPA and success-rate summary on the filter.
5. **Comparison.** Repeat steps 1-4 for the second passer and put both on one table with the same season and filters.

## Reading efficiency
EPA per dropback and success rate describe results; CPOE and time to throw describe how they were produced. A high time to throw with high pressure points at the line as much as the passer. Do not assign a quarterback's number to his skill alone: receivers, protection and scheme sit inside every field. Under about 150 dropbacks, treat every rate as unstable.

## Output template
```
<QB> (<team>) | <season> | games <n>, dropbacks <n>
EPA per dropback <v> (rank <r> of <q> qualifying) | success rate <%>
CPOE <v> (source: <tool>) or "CPOE not returned for this query"
Air yards per attempt <v> | time to throw <v> | pressure rate <%> (source)
Splits: <third down, red zone: EPA per play, plays>
Comparison (if asked): same rows for <QB 2>
Read: 2-3 bullets stating what the numbers show and the sample
Data: search_players, get_player_leaders, get_advanced_stats[, get_plays] | season <yyyy>
```

## Rules
- Say which source each tracking number came from (NGS or PFR).
- Do not infer CPOE, pressure or time to throw from box-score fields.
- Playoff games are a separate sample (`season_type: "POST"`); do not blend them silently.
- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Odds and prices are information about the market only: no picks, leans, stakes, payouts or sportsbook links.
- Sequences (last 5, recent form, a run of results) are quoted in the order the tool returns them, with dates; never retell them from memory. Lists in the answer must match the tool rows.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.

## Example prompts
- "How efficient is Jordan Love this season?"
- "Compare Joe Burrow and Justin Herbert on EPA and CPOE"
- "Is Jayden Daniels' pressure rate a problem?"
