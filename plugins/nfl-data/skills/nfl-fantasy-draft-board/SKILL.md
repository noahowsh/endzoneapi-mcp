---
name: nfl-fantasy-draft-board
description: "Use when someone is preparing a fantasy football draft: 'build me a draft board', 'tiers for running backs', 'who is overrated at his ADP', 'sleepers for this year', 'value over replacement', 'best available at pick 24', 'superflex rankings', 'who falls in my draft', 'Underdog or best ball board'. Builds tiered position rankings from last season's production, usage and depth chart, with value over replacement for the user's league size, and compares them with an ADP list the user pastes. Also for bare follow-ups with no names: load this skill first, then ask for what is missing, rather than answering from memory. Not for in-season waivers (see nfl-fantasy-waivers), weekly start/sit, trades, or DFS salary lineups. Cannot know ADP unless pasted."
metadata:
  version: 1.0.0
  author: SportsDataHQ
---

# NFL fantasy draft board

A draft board is tiers plus a value-over-replacement number, set against where the room is taking players. This skill builds the first two from the data and asks for the ADP, which is not in the data.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server. `season` is the year the season starts; `week` is the NFL week. Player ids are GSIS ids from `search_players`; snap counts use the PFR id that `search_players` also returns.
- ADP, rookie projections and contract news are not in the data. Ask the user to paste ADP (rank, name) from the site they draft on; without it, give tiers and value only and say where ADP would change the call.
- Ask for league size, starters (QB, RB, WR, TE, FLEX, superflex) and scoring; default 12 teams, 1 QB, 2 RB, 2 WR, 1 TE, 1 FLEX, PPR, and say so.
- Props are posted per game, so there are no season-long market projections in this data. The board rests on last season's results and usage.

## Default path (about 40 to 70 credits)
1. **Last season's scorers.** `get_player_leaders(stat: fantasy_points_ppr, position: RB, season, limit: 40, min_games: 8)` (5 cr), then WR (limit 50), TE (limit 20) and QB (limit 24). Use the prior full season.
2. **Usage.** `get_player_leaders(stat: targets ...)` and `get_player_leaders(stat: carries ...)` (5 cr each) for the volume behind the points.
3. **Roles this year.** `get_depth_chart(team)` (2 cr) for teams where a top player changed teams or a starter is hurt; `get_injuries(team)` (2 cr) for current status.
4. **Team context.** `get_team_stats(team, season)` (5 cr) for the teams of the borderline calls: pass and rush volume.

## Projection method
- Points per game, not totals: fantasy points per game over games played (n shown). Players under 8 games are marked thin.
- Replacement level by position for the league size: the Nth player at the position, where N is starters x teams (12 teams, 1QB: QB12, RB 2 starters + half the flex = RB30, WR 2 + half = WR30, TE12; adjust to the user's settings and show N).
- Value over replacement (VOR) = player's points per game minus the replacement player's, times 17 games. Show both.
- Tiers: sort by VOR and cut where the gap to the next player exceeds the average gap of the position by half again; show the break sizes. Players near a break are a coin flip inside the tier.
- Regression flags, from the same tools: touchdowns far above what usage supports (TDs per touch well above the position average), a rookie or new-team role, and age or injury history where the data shows missed games.
- With pasted ADP: value = ADP rank minus our VOR rank (positive means the room is taking him later than the board). Report only gaps of 10 or more picks.

## Output template
```
Draft board (<n> teams, <scoring>), based on <season> (replacement: <pos N>)
Tier 1 (<pos>): <player: pts/gm, VOR, gms> ...
Tier break: <gap size>
Targets vs ADP (if pasted): <player: ADP <x>, board rank <y>, gap>
Fades vs ADP: <player: reason from usage or role>
Thin samples: <players with under 8 games>
Data: get_player_leaders, get_depth_chart, get_injuries, get_team_stats | season <yyyy>
```

## When there is no game or no line
If a player missed most of last season, rank from the last full season and say so. Rookies have no NFL sample: list them under 'no NFL data' and place them only by the depth chart and the user's ADP. If the new season has no games yet, base everything on the prior season; if it has started, blend by weeks played and say how.

## Rules
- Fantasy rankings and advice only: no betting language. A prop line is used here only as the market's median projection of a player's stat.
- The user's scoring and roster settings come from the user; if unstated, say which default you used and show the formula so they can swap it.

- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size and season convention.
- Do not add names, roles, injuries or storylines from memory; if a tool did not return it, leave it out.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Sequences (last 5, recent form, a run of results) are quoted in the order the tool returns them, with dates; never retell them from memory. Lists in the answer must match the tool rows.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.
