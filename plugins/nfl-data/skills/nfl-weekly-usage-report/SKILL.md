---
name: nfl-weekly-usage-report
description: "Use when the user has weekly NFL fantasy or usage questions, in fan or analyst wording: 'who do I start at flex', 'RB2 or WR3 this week', 'who replaces Kupp', 'is his snap share rising', 'handcuff to grab', 'target hog on the Bengals', 'best waiver pickups', 'backfield split', 'who eats the targets if the top WR is out', 'which of my flex guys sees more volume', 'next man up if he sits'. Use this whenever fantasy lineups, waivers, snaps, targets or carries are the subject. Also use it for bare follow-ups with no name given, and load it instead of answering from memory or going straight to the raw tools. Builds a usage report from snaps, targets, carries, red-zone work and injuries, and names who absorbs a vacated role. Not for how an injury changes a whole team (see nfl-injury-impact), player prop lines (see nfl-player-props-research) or a game preview (see nfl-game-preview)."
metadata:
  version: 2.0.0
  author: SportsDataHQ
---

# NFL weekly usage report

Fantasy managers repeat the same weekly ritual: who is playing, how much, and who inherits the work when someone is out. Usage (snaps, routes, targets, carries, red-zone touches) predicts fantasy points better than last week's points. This skill builds that report from EndzoneAPI.

## Before you start
- For anything across games, seasons or players, apply the sports-data-preflight checks first (ids from search, the season convention, one league, enough data).
- Tools come from the EndzoneAPI MCP server. `season` is the year the season starts; `week` is the NFL week.
- Player ids are GSIS ids from `search_players`; snap counts may use a separate id, which `search_players` also returns.

## Default path (one team, about 24 credits)
1. **Injury report.** `get_injuries(team, week)` (2 cr): who is Out, Doubtful, Questionable, and practice participation.
2. **Snap shares.** `get_snap_counts(team, season, week)` (5 cr) for the last 3-4 weeks: offensive snap share per player.
3. **Opportunity.** `get_player_stats(player_id, season)` (5 cr) for the skill players in question: targets, carries, receptions, per week. Target share = player targets / team targets in the weeks played.
4. **Red zone (optional).** `get_plays(season, posteam, red_zone: true)` (10 cr): who gets inside-the-20 targets and carries.
5. **Depth chart.** `get_depth_chart(team, week)` (2 cr): who is next up at the injured player's spot.

## Vacated-role logic
When a player is out, his snaps and targets do not vanish; they move.
- List his average share over the last 3 healthy weeks.
- Look at games he missed earlier this season (if any) to see who absorbed the work: that is the best evidence.
- Otherwise use the depth chart and the snap pattern from late in games he left early.
- Say how confident you are: one prior missed game is thin evidence.

## Output template
```
<Team> usage, weeks <a>-<b> (<season>)
Injuries: <player: status (practice)>
Snap share: <player: %, trend>
Targets / carries per game and share: <player: tgt share %, carry share %>
Red zone (last 4 weeks): <player: RZ targets / carries>
If <injured player> sits: <who absorbs, with the evidence and its sample size>
Data: get_injuries, get_snap_counts, get_player_stats, get_depth_chart | season <yyyy>, weeks <a>-<b>
```

## Rules
- Shares need denominators: show the team totals or games played behind every percentage.
- Three weeks is the default window; say so, and widen it if the user asks for season-long.
- Fantasy advice is about usage and role; do not mix in betting language.
- If the injury report has not been released for the week yet, say so and use last week's.
- Out of credits: if a call is refused for credits, pass on the API's message as it comes (it says when free credits refill and how to add more), and stop; don't retry or switch tools to get around it.