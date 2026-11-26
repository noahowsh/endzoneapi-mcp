---
name: nfl-dfs-lineup
description: "Use when someone has an NFL daily fantasy slate and wants lineups built: 'build my DraftKings lineup', 'here is the salary file, make a lineup under the cap', 'FanDuel lineup for Sunday', 'optimize with a QB stack', 'cheap flex plays', 'value at RB', 'projections for the main slate', 'one lineup with a bring-back'. Builds lineups under the cap from the user's salary file using the market's per-player projection (prop medians as fantasy points). Also for bare follow-ups with no names: load this skill first, then ask for what is missing, rather than answering from memory. Not for entry fees, contest choice or how much to play, which it will not advise, and not for season-long start/sit (see nfl-fantasy-start-sit) or prop research (see nfl-player-props-research). Needs the salary file pasted."
metadata:
  version: 1.0.0
  author: SportsDataHQ
---

# NFL DFS lineup builder

DFS lineup building is a salary-cap knapsack: maximize projected points under the cap given the roster slots. Salaries are not in the data, so the user pastes the slate file. Projections come from the market's median for each player's stats, turned into fantasy points with a stated formula.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server. `season` is the year the season starts; `week` is the NFL week. Player ids are GSIS ids from `search_players`; snap counts use the PFR id that `search_players` also returns.
- Required from the user: the salary file (name, position, team, salary, and game if available), the cap, the roster slots and the scoring. DraftKings classic is usually QB, 2 RB, 3 WR, TE, FLEX, DST at 50,000 with full PPR and bonuses; FanDuel differs. Confirm what the user's site uses; never assume.
- Ownership is not in the data; use it only if pasted.

## Default path (a slate of 6 to 12 games is 60 to 240 credits; ask before pulling more than 100)
1. **Scope.** Parse the pasted file; list the games and teams. `get_schedule(week)` (2 cr) for game ids, totals and spreads.
2. **Cost check.** `get_player_props(game_id, estimate_only: true)` (free) for each game; tell the user the total and let them trim the slate.
3. **Projections.** `get_prop_probabilities(game_id)` (10 cr per game) for every player's yards, receptions and anytime-TD fair probabilities in one call; `get_player_props(game_id)` (10 cr) only for line numbers it does not give.
4. **Availability.** `get_injuries(team, week)` (2 cr per team): drop Out and Doubtful players from the pool; flag Questionable.
5. **Players without lines.** `get_player_stats(player_id, season)` (5 cr) for the trailing 5-game average, used only for cheap flex candidates.

## Projection method
Projection per player, each stat from the market's median:
- fantasy points = 0.04 x pass yards + 4 x pass TD - 2 x interceptions + 0.1 x rush yards + 0.1 x receiving yards + 1 x receptions (PPR; 0.5 for half PPR, 0 for standard) + 6 x expected rushing or receiving TDs. Apply the site's bonuses (for example 3 points at 100 rushing or receiving yards, 300 passing) only when the median line is at or past the threshold, and say so.
- expected TDs = -ln(1 - p), where p is the fair (no-margin) anytime-TD probability from `get_prop_probabilities` (it is the chance of at least one TD, so this converts it to an expected count).
- Defenses and kickers: no props. Use the game total and spread from `get_schedule` (a low team-total opponent and a large favorite help), and say the DST number is a rough guide, not a projection.
- Players with no line: trailing 5-game average, marked 'no market line'.
Lineup construction:
- Maximize the sum of projected points subject to the cap and the slots (solve exactly with a short script if code is available; otherwise greedy by points per $1,000 then pairwise swaps until no swap raises points under the cap, and say which you did).
- Show total salary, remaining salary and total projected points. Use at least 99% of the cap unless a swap loses points.
- If the user asks for a stack, fix the QB and one or two of his pass catchers and optionally one opposing player, then re-solve; say correlation is a structural choice, not something measured here.
- If asked for several lineups, change one or two players at a time and show the points cost of each change.

## Output template
```
Slate: <games>, cap <$>, scoring <site>
Lineup 1: salary <used>/<cap>, projected <pts>
QB <name> $<s> <pts> | RB ... | WR ... | TE ... | FLEX ... | DST ...
Value plays: <name: pts per $1,000>
Questionable / watch: <names and status>
No market line (trailing average): <names>
Data: get_schedule, get_prop_probabilities, get_injuries, get_player_stats | season <yyyy>, week <w>
```

## When there is no game or no line
If the user has no salary file, say the lineup cannot be built and offer a projection table for the players they name. If a game has no props yet (early in the week), project from the 5-game average and say the table is provisional; offer to refresh closer to kickoff. If the slate has fewer than 4 games, say the pool is thin and the projections are less separated.

## Rules
- The salary file, the cap, the roster slots and the scoring system come from the user. If any is missing, ask for it; never use remembered salaries or prices.
- Build lineups under the cap the user gives. Do not advise on entry fees, which contest to enter or how much to play, and do not call any player a lock. Projections are estimates, not promises.
- Odds and prices are information about the market only: no picks, leans, stakes, payouts or sportsbook links. A prop line is used here only as the market's median projection of a player's stat.
- Ownership is not in the data. Use it only if the user pastes it, and say it came from them.

- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size and season convention.
- Do not add names, roles, injuries or storylines from memory; if a tool did not return it, leave it out.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Sequences (last 5, recent form, a run of results) are quoted in the order the tool returns them, with dates; never retell them from memory. Lists in the answer must match the tool rows.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.
