---
name: nfl-fantasy-waivers
description: "Use when a fantasy football manager is looking for pickups: 'best waiver adds this week', 'who do I grab off the wire', 'handcuff worth stashing', 'breakout WR available', 'who is trending up in snaps', 'replace my injured RB', 'streaming TE', 'is he worth a FAAB bid', 'sleeper RBs with rising usage'. Also for bare follow-ups with no names: load this skill first, then ask for what is missing, rather than answering from memory. Ranks available players by usage trend (snap share, targets, carries, red-zone work) and the depth chart, with the user's roster-percentage list if pasted. Not for start/sit between players already owned (see nfl-fantasy-start-sit), who absorbs one named injured player's role (see nfl-weekly-usage-report), trade offers or DFS lineups. Never advises how much FAAB to spend."
metadata:
  version: 1.0.0
  author: SportsDataHQ
---

# NFL fantasy waiver and pickup report

Good pickups are role changes the roster percentage has not caught up with. The signal is usage rising over the last 2 to 3 weeks (snap share, targets, carries, red-zone touches) or a vacated role on the depth chart. Whether a player is available in your league is not in the data, so the user supplies the pool or the names.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server. `season` is the year the season starts; `week` is the NFL week. Player ids are GSIS ids from `search_players`; snap counts use the PFR id that `search_players` also returns.
- Roster percentage, FAAB budgets and who is available in the league are not in the data. Ask the user to paste their free-agent list or the names; otherwise report a league-wide usage-risers list and say availability is unchecked.
- Never recommend a FAAB amount; that depends on the user's budget and league.

## Default path (about 30 to 50 credits)
1. **Usage leaders by position.** `get_player_leaders(stat: targets, position: WR, season, limit: 40)` (5 cr), then `carries` for RB and `targets` for TE (5 cr each). These are season totals; for the trend use step 3.
2. **Candidates.** Keep players the user lists as available, or in the top of the list with low season totals relative to recent usage.
3. **Trend.** `get_snap_counts(team, season, limit: 1000)` (5 cr per team; the default 200 rows cuts a team's season short) for each candidate's team: offensive snap share by week; compare the last 2 weeks to the 3 weeks before.
4. **Role.** `get_depth_chart(team)` (2 cr) and `get_injuries(team)` (2 cr): is he next up, or did a starter go down?
5. **Opportunity detail (required for the top 3, and for the user's own players when they are named).** `get_player_stats(player_id, season)` (5 cr): weekly targets, carries and receptions; `get_schedule(days: 21)` (2 cr) for the next three opponents and games.

## Projection method
Rank candidates on one explicit score, shown per player:
- Opportunity per game = targets + carries per game over the last 3 games played (n shown).
- Trend = last 2 games minus the 3 games before, in opportunities and in snap share points.
- Role = depth chart rank at his position and whether a starter ahead is Out or Doubtful.
- Order: role gain first, then opportunity per game, then trend. A player with two weeks of rising work and a hurt starter ahead outranks a one-week spike.
- One game is not a trend: a single week above 60% snaps after a year below 30% is flagged as one game.
- Handcuffs: name the starter, his injury status and the backup's current snap share; say a handcuff has value only if the starter is out or at risk.
- For tight ends and quarterbacks, targets and pass attempts per game carry the role; say the TE pool is thin when few have 5 targets per game.

## Output template
```
Waiver report, week <w> (<scoring if stated>)
1. <player> (<team>, <pos>): opp/gm <x> over <n> gms | snap share <a>% -> <b>% | depth <rank> | next 3 opponents <list>
2. <player>: ...
Handcuffs: <starter (status)> -> <backup: snap %>
Availability: <checked against your list / unchecked>
Data: get_player_leaders, get_snap_counts, get_depth_chart, get_injuries, get_player_stats, get_schedule | season <yyyy>, weeks <a>-<b>
```

## When there is no game or no line
If it is early in the season (under 3 weeks played), use last season's final role plus the depth chart and say the sample is under 3 games. If the injury report is not out, use last week's. In a bye week or offseason, rank on last season's usage and the depth chart and say there is no current game to base a trend on.

## Rules
- Fantasy rankings and advice only: no betting language. A prop line is used here only as the market's median projection of a player's stat.
- The user's scoring and roster settings come from the user; if unstated, say which default you used and show the formula so they can swap it.

- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size and season convention.
- Do not add names, roles, injuries or storylines from memory; if a tool did not return it, leave it out.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Sequences (last 5, recent form, a run of results) are quoted in the order the tool returns them, with dates; never retell them from memory. Lists in the answer must match the tool rows.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.
