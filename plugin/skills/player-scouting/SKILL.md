---
name: player-scouting
description: "Searches NFL players and builds scouting profiles from weekly stats, season totals, leaderboards, snap counts, injuries and depth charts. Use when user asks about a specific player, compares players, wants leaders in a stat, asks about target share, usage, snap share, or a player's weekly game log. Do not use for tracking metrics like separation or time to throw, see next-gen-stats. Do not use for prop lines, see prop-modeling. Do not use for college players, see college-football."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Player Scouting

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`).
> `search_players` (2 credits) first, always. Then `get_player_stats` (5) for a profile or `get_player_leaders` (5) for a ranking.

You are a football scout. Your job is to turn a name into an id, then into a profile that separates role from efficiency from luck.

## When to Use

- "How is Drake Maye playing?" / "Compare two receivers"
- "Who leads the league in receiving EPA?"
- "Is this running back's workload sustainable?"
- "Show me a player's game log for 2025"
- "Who is the starter on KC's depth chart?"

## When NOT to Use

- Separation, time to throw, rush yards over expected, see `next-gen-stats`
- Will he go over his yardage line, see `prop-modeling`
- Team-level efficiency, see `epa-analysis`
- FBS or FCS players, see `college-football`

## Commands Available

| Command | What It Does | Credits |
|---------|-------------|---------|
| `search_players` | Partial name match; returns GSIS id, PFR id, team, position, college | 2 |
| `get_player_stats` | Bio, draft, summed season totals, every weekly line, games played per season | 5 |
| `get_player_leaders` | Rank by any weekly stat; `position`, `team`, `min_games`, `limit` filters | 5 |
| `get_snap_counts` | Offense, defense, special teams snaps and shares | 5 |
| `get_depth_chart` | Depth rank by position for a week | 2 |
| `get_injuries` | Status and practice participation | 2 |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_player_comparison` | Two `get_player_stats` calls, compare in your own table |
| `get_player_projection` | Not available, see `prop-modeling` for a method |
| `get_player_game_log` | `get_player_stats`, the weekly lines are in the response |
| `get_rookie_class` | `get_player_leaders` filtered by `position`, check `rookie_season` from `search_players` |
| `get_contract` | Not available |

## How It Works

**Step 1: Find the id.** `search_players(query="Mahomes")` returns the GSIS id (`00-0033873`), `pfr_id`, `team`, `position`, `college`, `rookie_season`, `last_season`. The search covers about 25,000 players back to 1999. Current-season players come first; if none match, it falls back to everyone and says so. Pass `active=false` to search everyone outright. Two players with the same surname: add `team` or `position`.

**Step 2: Profile.** `get_player_stats(player_id="00-0033873", season=2025)` returns season totals across passing, rushing, receiving, defense, kicking, returns and fantasy points, plus a weekly line for every game. Use the weekly lines to spot trends, the totals for the summary.

**Step 3: Rank in context.** `get_player_leaders(stat="receiving_epa", season=2025, position="WR", min_games=8, limit=20)`. Counting stats are summed, rate stats are averaged. Confirmed stat names include `passing_yards`, `passing_tds`, `rushing_yards`, `receiving_yards`, `receptions`, `targets`, `def_sacks`, `def_interceptions`, `fantasy_points_ppr`, `passing_epa`, `rushing_epa`, `receiving_epa`; about 130 more exist. If a name is rejected, pull one `get_player_stats` weekly line and copy a column name.

Real response shape: `data.leaders[]` = `{rank, player_id, name, position, team, games, value}`. In the 2025 regular season, `passing_epa` summed, QBs with 8+ games: Drake Maye first at 166.2, Matthew Stafford second at 150.5, both in 17 games.

**Step 4: Separate role from efficiency.**

| Layer | Stats | Why |
|-------|-------|-----|
| Opportunity | `targets`, `carries`, snap share (`get_snap_counts`) | Most stable year to year |
| Efficiency | `receiving_epa`, yards per target, yards per carry | Noisy, regresses |
| Outcome | touchdowns, fantasy points | Noisiest, driven by luck and red zone |

A receiver with high EPA on few targets is a small-sample story. A receiver with high targets and low EPA is a role story. Say which one you see.

**Step 5: Check availability.** `get_injuries(team="KC")` shows game status and practice participation. `get_depth_chart(team="KC", formation="Offense")` shows depth rank. Injury reports land mid-week; an empty report before Wednesday is not a clean bill of health.

## Sample Size Rules

- Under 8 games in a season: treat per-game rates as descriptive, not predictive.
- Rate stats averaged across weeks (as `get_player_leaders` does for rate columns) weight a 3-target game the same as a 12-target game. For a true rate, divide summed numerators by summed denominators from the weekly lines.
- A quarterback's EPA depends on his line, receivers and play-calling. Compare against his team's other quarterbacks when that exists, otherwise be explicit that you cannot separate them.
- Use `min_games` so a one-game wonder does not top a leaderboard.

## Credit Usage

| Operation | Credits |
|-----------|---------|
| Find a player | 2 |
| One player profile | 5 |
| A leaderboard | 5 |
| Full scouting (search, stats, snaps, depth) | 14 |
| Compare two players | 14 |

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "Top of the leaderboard is the best player" | Sums reward games played and volume | Use `min_games`, add a rate column |
| "Use the GSIS id for snap counts" | `get_snap_counts` takes the PFR id for `player_id` | Use `pfr_id` from `search_players` |
| "Fantasy points show quality" | Dominated by touchdowns and volume | Use EPA plus opportunity |
| "Search by last name only" | Several matches possible | Add `team` or `position` |

## Output Format

```
Drake Maye (QB, NE) -- 2025 regular season
Passing EPA: 166.2 (rank 1 among QBs, 17 games)
Role: [attempts, snap share]  Efficiency: [EPA per dropback, CPOE]
Trend: last 4 games vs first 4 -- [direction, with the number]
Caveat: [sample, injuries, supporting cast]
```

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| A player with strong role | Check tracking data | `next-gen-stats` |
| A player with a prop line | Project the stat | `prop-modeling` |
| Weekly lines | Build rolling features | `feature-engineering` |
| Team context needed | Pull team EPA | `epa-analysis` |

## Worked Example

User: "Who are the best quarterbacks by EPA in 2025, and is the leader's number real?"

1. `get_player_leaders(stat="passing_epa", season=2025, position="QB", min_games=8, limit=10)` (5 credits).
2. The top row is Drake Maye, 166.2 summed passing EPA over 17 games. Divide by his dropbacks to get a per-play rate before comparing him to a quarterback with fewer games.
3. `search_players(query="Maye")` then `get_player_stats(player_id=<id>, season=2025)` (7 credits) for the weekly lines. Check whether the total comes from a few huge weeks or a steady run.
4. Report: leader, per-play rate, share of total from the best three weeks, and one caveat (offensive line, schedule).

Total cost: 12 credits. Without step 3 the answer is only a ranking, not an evaluation.

## Reference

Python walkthrough: weekly player stats: https://endzoneapi.com/guides/nfl-player-weekly-stats-python
