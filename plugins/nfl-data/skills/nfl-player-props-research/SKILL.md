---
name: nfl-player-props-research
description: "Use when the user asks about NFL player prop lines as information: 'what's Josh Allen's passing yards line', 'how often has Saquon gone over his rushing line', 'compare receiving yards lines across books', 'how did his line move', 'what does the market imply for his TD'. Shows lines by sportsbook, a player's record against his line with sample size, line movement and no-margin probability. Never says which side to take. Not for game-level previews (see nfl-game-preview) or fantasy start/sit (see nfl-weekly-usage-report)."
metadata:
  version: 2.0.0
  author: SportsDataHQ
---

# NFL player props research

Prop lines are information about what the market expects a player to do. This skill shows that information clearly and checks it against what the player actually did. It never recommends a side, a stake or a bet.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server. Call them; never quote a line from memory (lines move hourly).
- Prop history starts with the 2023 season. Player ids are GSIS ids (`00-0033873`); get them from `search_players`.
- Markets: `player_pass_yds`, `player_pass_tds`, `player_rush_yds`, `player_reception_yds`, `player_receptions`, anytime TD and more (see the tool's `market` list).

## Default path
1. **Resolve the player.** `search_players(query)` (2 cr) for the GSIS id; confirm team and position.
2. **Current lines.** `get_player_props(game_id or player_id, market)` (10 cr): line and prices per book, opening and closing.
3. **What the market implies.** `get_prop_probabilities(game_id, market, player_id)` (10 cr): average implied and no-margin probability of the over and under.
4. **Track record against the line.** `get_prop_history(player_id, market)` (10 cr): each game's line, result and over/under/push.
5. **Movement (optional).** `get_prop_movement(game_id, player_id, market)` (10 cr) when the user asks how the line moved.
6. **Context (optional).** `get_snap_counts` (5 cr) and `get_injuries` (2 cr) when usage or health explains the history.

## How to report a track record
- Hit rate = overs / (overs + unders). Pushes are excluded and counted separately.
- Always show n: "over in 9 of 14 (64%), 1 push, 2023-2025". Below 10 games, say the sample is too small to mean much.
- Show the trend in the line itself (lines rise for players who keep going over).
- A past hit rate is not a forecast. Do not present it as one.

## Output template
```
<Player> (<team>, <pos>) | <market> | <game, date>
Lines now: <book: line (over/under prices)> ... | consensus <line>
Market implies: over <x%> / under <y%> (no-margin, average of books)
Against his line: over <a> of <a+b> (<%>), <p> pushes, since <season> | last 5: <O/U sequence>
Movement: opened <line> -> now <line> (<books>)
Context: snaps/targets or injury note if relevant
Data: search_players, get_player_props, get_prop_probabilities, get_prop_history | season <yyyy>
```

## Rules
- Odds are information only: no picks, no "lean", no "value", no stake or payout math, no sportsbook links.
- Name the tool behind every number and the sample size behind every rate.
- If a market has no line for this game yet, say so; do not substitute another market.
