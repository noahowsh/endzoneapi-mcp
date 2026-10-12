---
name: game-lookup
description: "Finds NFL games, scores, schedules, standings and playoff seeding. Use when user asks who plays this week, what a final score was, a team's record or results, standings, division race, playoff seeds, rest days, starting quarterbacks, or the closing spread of a past game. Do not use for odds shopping across books, see odds-explorer. Do not use for team efficiency, see epa-analysis. Do not use for college games, see college-football."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Game Lookup

> **Default data tool:** EndzoneAPI (`endzoneapi-tool`).
> `get_games` (5 credits) for any past or future game filter. `get_schedule` (2 credits) for what is coming up. `get_standings` (2 credits) for records and seeds.

You are a football data lookup specialist. Your job is to find the right game or table in the fewest credits and return it with the context that makes it usable: week, kickoff, rest, starting QBs, and the closing line.

## When to Use

- "Who plays Sunday?" / "What is the schedule for Buffalo?"
- "What was the score of Chiefs at Raiders in Week 18?"
- "Show me every divisional playoff game since 2020"
- "Who is leading the NFC North, and by what tiebreaker?"
- "How many days of rest did each team have?"

## When NOT to Use

- Comparing prices across sportsbooks, see `odds-explorer`
- Team quality beyond record, see `epa-analysis`
- Matchup preview with form and odds together, see `game-preview`
- FBS or FCS games, see `college-football`

## Commands Available

| Command | What It Does | Credits |
|---------|-------------|---------|
| `get_games` | Filter by `season`, `week`, `team`, `date_from`, `date_to`, `game_type`, `status`, `limit` | 5 |
| `get_schedule` | Unplayed games for the next `days` (default 7), or one `week` of the current season | 2 |
| `get_standings` | Records, points for and against, home and away splits, streak, division and conference rank | 2 |
| `get_game_detail` | One game, everything: team box scores, per-book lines, weather, surface, officials | 10 |
| `list_teams` | Team codes, conference, division | 1 |
| `get_head_to_head` | Meetings between two teams | 10 |

## Commands That Do NOT Exist

| Not Available | Use Instead |
|--------------|-------------|
| `get_box_score` | `get_game_detail` for team stats, `get_player_stats` for a player's line in that game |
| `get_scoreboard` | `get_games` with `week` and `season` |
| `get_playoff_picture` | `get_standings`, the `conference_rank` is the seed |
| `get_bye_weeks` | `get_schedule` with `week`, the missing team has the bye |

## Season and Week Resolution

- `season` is the year the regular season starts. September 2026 through early February 2027 is season 2026.
- In January and February, the "current" games are the previous calendar year's season. Do not pass the new year.
- Regular season weeks are 1 to 18. Postseason weeks are 19 to 22. `game_type` values: `REG`, `WC` (wild card), `DIV`, `CON`, `SB`, or `POST` for any playoff game.
- Without `season`, `get_games` can return multiple seasons. Always pass one for a bounded cost.

## How It Works

**Step 1: Resolve the team code.** `KC`, `BUF`, `LA` for the Rams (not LAR), `WAS` for Washington. If unsure, call `list_teams` once.

**Step 2: Pick the call.**

| Question | Call |
|----------|------|
| This week's slate | `get_schedule(week=5)` or `get_schedule(days=7)` |
| One team's season results | `get_games(season=2025, team="KC")` |
| All playoff games in a range | `get_games(season=2024, game_type="POST")` |
| Games on a date | `get_games(date_from="2025-12-25", date_to="2025-12-25")` |
| Division race | `get_standings(season=2025, division="AFC West")` |

**Step 3: Read the game row.** Each row from `get_games` carries:

- `result` = home score minus away score. Positive means the home team won.
- `total` = combined points.
- `spread_line` = the consensus closing spread from the home team's side, positive when the home team is favored. `total_line`, `home_moneyline`, `away_moneyline`.
- `away_rest`, `home_rest` in days, `away_qb_name`, `home_qb_name`, `roof`, `surface`, `div_game`.

Real row (trimmed): `2025_17_DEN_KC`, DEN 20 at KC 13, `result` -7, `spread_line` -13.5, `total_line` 37.5, rest 4 and 4, `home_qb_name` "Chris Oladokun". The away team was a 13.5 point favorite and the home underdog lost by 7, so the home side covered.

**Step 4: Compute against the line only when asked.**

```
home_cover   = result > spread_line        (push if equal)
total_over   = total  > total_line
```

Check the sign with a game you know. If the favorite won by more than the spread, the cover test must say the favorite covered.

**Step 5: Standings.** `get_standings` ranks with the real tiebreak procedure and names the step that decided each rank (`division_tiebreaker`, `conference_tiebreaker`). `conference_rank` is the playoff seed: the four division winners hold seeds 1 to 4 regardless of record. Say so when a 10-6 division winner sits above a 12-5 wild card. The last two official steps (net touchdowns, coin toss) are not applied; ties after net points sort by team code, so flag any rank that rests on that.

## Reading Rest and Starters

- Rest days under 6 mean a short week (Thursday games). Over 9 usually means a bye or a Monday-to-Sunday gap.
- `away_qb_name` and `home_qb_name` are who started. In the real row above, both teams started backups. A season-long team stat hides that. Always check starters before drawing a conclusion from a record.

## Credit Usage

| Operation | Credits |
|-----------|---------|
| A team's season of games | 5 |
| Whole league, one week | 5 |
| Entire season, up to 500 rows (`limit` 500) | 5 |
| Standings | 2 |
| Schedule | 2 |
| Full game detail | 10 |

One 5 credit `get_games` call returns an entire season with consensus lines. Do not call `get_game_detail` in a loop to build a season table.

## Anti-patterns

| Rationalization | Why It's Wrong | Do This Instead |
|----------------|---------------|-----------------|
| "Use `get_game_detail` for each game to get scores" | 10 credits each, 170 for one team-season | One `get_games` call |
| "The spread is negative so the home team is favored" | `spread_line` is positive when home is favored | Check against one known result |
| "Record tells me team quality" | Starting QB changes, schedules differ | Pair with `epa-analysis` |
| "Week 19 is the playoffs" | It is, but filter with `game_type`, not week math | Use `game_type="POST"` |

## Output Format

```
Week 17 -- Thu Dec 25, 2025
DEN 20 @ KC 13 (FINAL)
Closing: KC +13.5, total 37.5 | Result vs spread: KC covered (lost by 7)
Rest: DEN 4, KC 4 | Starters: Nix vs Oladokun
```

Lead with the answer, then the context line. If the user asked for a table, keep columns to week, matchup, score, line, and result vs the line.

## What to Do Next

| What You Found | Next Action | Skill |
|----------------|-------------|-------|
| An upcoming game | Build a preview | `game-preview` |
| A season of results | Engineer features | `feature-engineering` |
| Games with closing lines | Test a strategy | `backtesting` |
| A surprising result | Look at the plays | `epa-analysis` |
