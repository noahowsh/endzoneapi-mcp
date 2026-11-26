---
name: nfl-fantasy-trade-value
description: "Use when a fantasy football manager asks about a trade: 'is this trade fair', 'my Barkley for his Chase', 'should I trade for him', 'who wins this trade', 'sell high on him?', 'buy low candidates', '2 for 1 offer', 'what is he worth rest of season', 'trade deadline moves'. Compares the two sides by rest-of-season points over replacement from the market's projection, usage and the remaining schedule and byes, for the user's scoring and roster. Also for bare follow-ups with no names: load this skill first, then ask for what is missing, rather than answering from memory. Not for start/sit, waivers, draft boards, dynasty pick values or DFS. Does not use trade charts it cannot verify."
metadata:
  version: 1.0.0
  author: SportsDataHQ
---

# NFL fantasy trade check

A trade is fair when the points you add to your lineup exceed the points you lose. That depends on the rest of the schedule, the byes and the replacement player you would start instead. This skill prices both sides in rest-of-season points above replacement and says who gains.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server. `season` is the year the season starts; `week` is the NFL week. Player ids are GSIS ids from `search_players`; snap counts use the PFR id that `search_players` also returns.
- Ask for league scoring, starters, and the user's roster at the positions involved (names are enough). Without them, price the players alone and say the roster fit is unknown.
- Dynasty picks, contract years and rookie value are not in the data; say a dynasty trade cannot be fully priced.

## Default path (about 40 to 80 credits)
1. **Players.** `search_players(name)` (2 cr each), `get_player_stats(player_id, season)` (5 cr each): weekly lines, games played, targets, carries.
2. **Role and health.** `get_snap_counts(team, season)` (5 cr per team) for the 3-week snap trend; `get_injuries(team)` (2 cr) and `get_depth_chart(team)` (2 cr).
3. **Rest of schedule.** `get_schedule(week)` (2 cr per week for the next 3 to 4 weeks, the playoff weeks if the user names them) for opponents, byes and game totals.
4. **Market projection for the next game.** `get_prop_probabilities(game_id, player_id)` (10 cr) for the closest game of each key player.

## Projection method
- Per-game level for each player = average of (a) the market projection for the next game, using fantasy points = 0.04 x pass yards + 4 x pass TD - 2 x interceptions + 0.1 x rush yards + 0.1 x receiving yards + 1 x receptions (PPR; 0.5 for half PPR, 0 for standard) + 6 x expected rushing or receiving TDs, with expected TDs = -ln(1 - p), where p is the fair (no-margin) anytime-TD probability from `get_prop_probabilities` (it is the chance of at least one TD, so this converts it to an expected count), and (b) the trailing 5-game average fantasy score (n shown). When only (b) exists, use it alone and say so.
- Rest-of-season games = games left minus the player's bye, minus any games he is expected to miss from the injury report (state the assumption).
- Replacement level = the points per game of the best player the user could start at that position from their bench, or the user's stated free-agent level; if unknown, use the position's replacement from `get_player_leaders` (the Nth best, N = starters x teams) and say it is generic.
- Points above replacement (PAR) = (per-game level - replacement) x games left, for each side; for a 2-for-1, the side receiving two players only counts the one who would start, plus the bench player's PAR over the player he displaces; the other side gains the open roster spot (use the replacement level).
- Net = PAR received - PAR given. Within about 10% of the larger PAR, call it even.
- Schedule note: mention the opponents for the next 3 games and the playoff weeks, and say schedule strength is noisy over a few games.
- Sell-high / buy-low flags need evidence from the tools: touchdowns above what yards and usage support, a snap share that moved by 10 points or more over 3 games, or a returning or newly injured starter.

## Output template
```
Trade check (<scoring>): you give <players>, you get <players>
<player>: level <x> pts/gm (market <a>, last <n> gms <b>) | games left <g> | PAR <v>
...
Net for you: <+/- points over replacement> -> <favours you / favours them / even>
Roster fit: <who you would start, from your list> (or "unknown, roster not given")
Flags: <bye, injury, role change, TD-heavy production>
Data: get_player_stats, get_snap_counts, get_schedule, get_prop_probabilities | season <yyyy>, week <w>
```

## When there is no game or no line
If a player has no upcoming game (bye) or no line, use the recent-average level only and mark it. Preseason or no games played: there is no rest-of-season to price; use last season's level and say so. Never fill in a trade-chart number from memory.

## Rules
- Fantasy rankings and advice only: no betting language. A prop line is used here only as the market's median projection of a player's stat.
- The user's scoring and roster settings come from the user; if unstated, say which default you used and show the formula so they can swap it.

- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size and season convention.
- Do not add names, roles, injuries or storylines from memory; if a tool did not return it, leave it out.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Sequences (last 5, recent form, a run of results) are quoted in the order the tool returns them, with dates; never retell them from memory. Lists in the answer must match the tool rows.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.
