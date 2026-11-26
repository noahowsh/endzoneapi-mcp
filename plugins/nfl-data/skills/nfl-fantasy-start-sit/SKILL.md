---
name: nfl-fantasy-start-sit
description: "Use when a fantasy football manager asks who to start: 'start or sit', 'Flex: Chase or Waddle', 'QB1 or streamer this week', 'should I start my RB2', 'who is better this week, X or Y', 'rank my WRs for Sunday', 'is he startable with that injury', 'sit him vs that defense?'. Also for bare follow-ups with no names: load this skill first, then ask for what is missing, rather than answering from memory. Ranks the user's own players for one week from the market's per-player projection (prop medians turned into fantasy points), role and injury status. Asks for the league's scoring if unstated. Not for waiver pickups (see nfl-fantasy-waivers), DFS salary lineups (see nfl-dfs-lineup), trade offers (see nfl-fantasy-trade-value) or prop research (see nfl-player-props-research)."
metadata:
  version: 1.0.0
  author: SportsDataHQ
---

# NFL fantasy start/sit

A start/sit call is a ranking of your own players for one week. The best single input is the market's projection of each player's yards, receptions and touchdowns, because it already prices the matchup, the game script and the injury news. This skill turns those medians into fantasy points with a stated formula, checks role and health, and ranks.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server. `season` is the year the season starts; `week` is the NFL week. Player ids are GSIS ids from `search_players`; snap counts use the PFR id that `search_players` also returns.
- The league's scoring (PPR, half PPR, standard, 6-point or 4-point passing TDs) comes from the user; if unstated, use full PPR and say so.
- Props are posted close to game day. Earlier in the week some players have no line yet.

## Default path (2 to 4 players, about 25 to 60 credits)
1. **Players and games.** `search_players(name)` (2 cr each) for ids and teams; `get_schedule(week)` (2 cr) for the game ids, the total and the spread.
2. **Cost check.** `get_player_props(game_id, estimate_only: true)` (free) before pulling lines; each game is 10 cr.
3. **The market's projection.** `get_prop_probabilities(game_id, player_id)` (10 cr per game) for yards, receptions and anytime TD fair probabilities by line; `get_player_props(game_id)` (10 cr) when you need the line numbers themselves.
4. **Health and role.** `get_injuries(team, week)` (2 cr per team) and `get_snap_counts(team, season, limit: 1000)` (5 cr per team) for snap share over the last 3 weeks.
5. **Recent production (required).** `get_player_stats(player_id, season)` (5 cr) for every ranked player: weekly lines for the last 5 games, used for the floor and ceiling and the snap check. Do not skip it, and do not claim a snap share or floor you did not pull.

## Projection method
For each player, per game:
- fantasy points = 0.04 x pass yards + 4 x pass TD - 2 x interceptions + 0.1 x rush yards + 0.1 x receiving yards + 1 x receptions (PPR; 0.5 for half PPR, 0 for standard) + 6 x expected rushing or receiving TDs.
- Rush and receiving yards, receptions and pass yards use the prop line (the median outcome). expected TDs = -ln(1 - p), where p is the fair (no-margin) anytime-TD probability from `get_prop_probabilities` (it is the chance of at least one TD, so this converts it to an expected count).
- Quarterbacks: add pass TDs and interceptions lines if posted; if interceptions are not posted, subtract the player's season average per game and say so.
- Range: take the lowest and highest of the last 5 weekly fantasy scores (`get_player_stats`) as the floor and ceiling of recent results, with n shown. Do not invent a distribution.
- Prop medians sit slightly below the mean for skewed stats like yards, so the projection runs a little low for every player alike. Compare players on the same basis and do not quote the number as a forecast of the exact score.
- Tiebreaks when projections are within about 1.5 points: higher snap share trend, then red-zone role, then the better game total (`get_schedule`). Say it is a coin flip when nothing separates them.
- Injury: a player listed Doubtful or Out is not ranked; Questionable gets a note and the user is told to check inactives before kickoff.

## Output template
```
Week <w> start/sit (<scoring>)
1. <player> (<team> vs <opp>): <proj> pts | rec <line> yds <line> TD prob <p> | snap share <%> over <n> wks | <injury status>
2. <player>: ...
Call: start <player> over <player>; margin <x> pts (<clear / close / coin flip>)
Floor / ceiling of the last <n> games: <player: lo-hi>
Data: get_schedule, get_prop_probabilities, get_injuries, get_snap_counts, get_player_stats | season <yyyy>, week <w>
```

## When there is no game or no line
If a player has no prop line yet, project from usage: trailing 3-week average of targets, carries and receptions from `get_player_stats`, times the player's season yards per target or per carry, plus 6 x (red-zone touches per game x 0.15 as a rough TD rate; say it is rough). Mark those rows 'usage-based, no market line' and do not rank them above a priced player on a thin gap. On a bye week or no game, say so and rank for the next game. If the injury report is not out yet, use last week's and say so.

## Rules
- Fantasy rankings and advice only: no betting language. A prop line is used here only as the market's median projection of a player's stat.
- The user's scoring and roster settings come from the user; if unstated, say which default you used and show the formula so they can swap it.

- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size and season convention.
- Do not add names, roles, injuries or storylines from memory; if a tool did not return it, leave it out.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Sequences (last 5, recent form, a run of results) are quoted in the order the tool returns them, with dates; never retell them from memory. Lists in the answer must match the tool rows.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.
