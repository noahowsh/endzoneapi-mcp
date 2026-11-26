---
name: nfl-injury-impact
description: "Use when the user asks what an NFL injury or absence changes for a team, in fan or analyst wording: 'how bad is losing their LT', 'Chiefs without Kelce', 'does the D fall off with no Micah', 'who steps in at center', 'how do they play when he sits', 'is the injury report a big deal', 'record without their QB'. Use this whenever a missing player and a team's results or lineup are discussed. Load it for short fragments and follow-ups too, even with no name given, instead of answering from memory or going straight to the raw tools. Combines the injury report, depth chart, snap counts and with-and-without splits from play-by-play, with sample sizes. Not for fantasy waiver or usage (see nfl-weekly-usage-report), prop lines (see nfl-player-props-research), a game preview (see nfl-game-preview) or other sports' injuries."
metadata:
  version: 2.1.0
  author: SportsDataHQ
---

# NFL injury impact

An injury matters through the player's role and who replaces him. This skill finds the status, the replacement and what the team's efficiency looked like with and without the player, and says plainly when the sample cannot answer.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server (`https://mcp.endzoneapi.com/mcp`). Call them; never fill in a number from memory.
- `season` is the year the regular season starts (2025 = the 2025-26 season). Team codes are standard abbreviations (LA, WAS, LV, LAC, JAX); `list_teams` (1 cr) if unsure.
- The injury report gives game status (Out, Doubtful, Questionable) and practice participation by team and week; it defaults to the latest reported week. It is not a forecast of who plays.
- With-and-without splits need games the player missed. If he missed fewer than 3 games, report the games and say the split is anecdotal.
- Snap counts use the `pfr_id` that `search_players` returns; plays and advanced stats use GSIS ids.

## Default path (about 20 to 30 credits)
1. **Status.** `get_injuries(team, week)` (2 cr), filter `status` if needed. Report status, injury and practice participation.
2. **Replacement.** `get_depth_chart(team, week)` (2 cr): who sits behind him.
3. **Role.** `get_snap_counts(team, season)` (5 cr) or by player: snap share of the injured player and the backup.
4. **Games without him.** Find the weeks he missed from the snap counts, then `get_plays(season, posteam, week)` (10 cr per week) for those weeks and for a comparison set; report EPA per play and success rate from the query summaries. For a defender, use `defteam`.
5. **Team context (optional).** `get_team_stats(team, season)` (5 cr) for weekly lines to see points scored and allowed in the missed games.

## How to frame the split
Compare like with like: same team, same season, games he played against games he missed, with the opponents named. Opponent strength and game script move EPA per play more than most single players. Report the difference and its sample (games and plays on each side) and call it a description, not an estimate of the player's value. For quarterbacks, a mid-game replacement is a different sample from a full game started by the backup.

## Output template
```
<Player, position, team> | status: <Out/Doubtful/Questionable>, practice <DNP/limited/full>, <injury> | week <n>, <season>
Replacement: <name> (depth chart), snap share <%> vs <%> for the starter
With him: <games>, EPA per play <v>, success rate <%> (<plays> plays)
Without him: <games>, EPA per play <v>, success rate <%> (<plays> plays), opponents <names>
Read: <what changed, what the sample cannot separate>
Data: get_injuries, get_depth_chart, get_snap_counts, get_plays | season <yyyy>, weeks <list>
```

## Rules
- Status is the report's, not a prediction; say when the report is more than a few days old.
- Fewer than 3 missed games, or fewer than 150 plays on the without side: say "not enough data" for the split and show the games only.
- Never call a player's absence the cause of a result the sample cannot isolate.
- Every number names its tool (the Data line, plus inline where it could be doubted) and every rate shows its sample size.
- If a tool returns too little, say "not enough data" and stop; do not estimate or borrow another sample.
- Odds and prices are information about the market only: no picks, leans, stakes, payouts or sportsbook links.
- Sequences (last 5, recent form, a run of results) are quoted in the order the tool returns them, with dates; never retell them from memory. Lists in the answer must match the tool rows.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.

## Example prompts
- "How much does losing the left tackle matter for the Eagles?"
- "What does the offense look like without Joe Burrow?"
- "Who plays if the Lions RB1 is out?"
