# EndzoneAPI MCP server

**The football data API.** Play-by-play with EPA and win probability, player and team stats, schedules, injuries, depth charts and closing lines for every NFL game since 1999. REST API and MCP server. 500 free credits, no card.

Website: https://endzoneapi.com · Docs: https://endzoneapi.com/docs · Get a key: https://endzoneapi.com/signup

```
You:    "What was the closing spread for every Chiefs game last season?"
Claude: [calls get_games -> one call, every game with spread, total and moneyline]

You:    "Which offenses had the best EPA per play on third down in 2025?"
Claude: [calls get_plays with third_down filter -> EPA already attached]
```

## Quick start

This is a hosted (remote) MCP server over Streamable HTTP. There is nothing to install.

**Claude Code**

```bash
claude mcp add --transport http endzoneapi "https://mcp.endzoneapi.com/mcp?key=YOUR_API_KEY"
```

**Claude Desktop, Cursor and other clients**

```json
{
  "mcpServers": {
    "endzoneapi": {
      "url": "https://mcp.endzoneapi.com/mcp?key=YOUR_API_KEY"
    }
  }
}
```

The key can also be sent as an `x-api-key` header or `Authorization: Bearer`. Connecting and listing tools works without a key; calling a tool needs one.

**REST**

```bash
curl -X POST https://mcp.endzoneapi.com/v1/list_teams -H "x-api-key: YOUR_API_KEY"
```

Every tool is `POST /v1/{tool}` with a JSON body.

## Tools (17)

| Tool | What it returns |
|------|-----------------|
| `list_teams` | All 32 teams with conference, division, colours, and logos |
| `get_standings` | Standings and playoff seeding by the NFL tiebreaking procedure, with the step that decided each rank |
| `get_schedule` | Upcoming games by week or for the next N days |
| `search_players` | Find players by name, team, or position |
| `get_injuries` | Injury report by team and week with practice and game status |
| `get_depth_chart` | Team depth chart by week |
| `get_games` | Query games by season, week, team, date, type, or status, with closing lines |
| `get_team_stats` | Team totals and per-week lines with EPA, success rate, and situational splits |
| `get_player_stats` | Weekly and season stat lines for one player: passing, rushing, receiving, defense, kicking, fantasy |
| `get_player_leaders` | Leaderboard for any stat, filtered by position, team, and season |
| `get_snap_counts` | Offensive, defensive, and special-teams snaps and shares by game or player |
| `get_advanced_stats` | Next Gen Stats tracking metrics and Pro Football Reference advanced stats |
| `get_plays` | Play-by-play with EPA, win probability, situation filters, and optional FTN charting |
| `get_odds` | Odds by game with bookmaker filters |
| `get_game_detail` | Full game detail with team stats, starting QBs, odds, weather, and officials |
| `get_head_to_head` | Matchup history with win/loss records and margins |
| `get_line_movement` | Time-series odds grouped by bookmaker |

## Data

- Play-by-play for every game since 1999 with EPA, win probability and the nflfastR model columns
- Closing spread, total and moneyline for every game since 1999; per-book odds and hourly line movement from September 2026
- Weekly player and team stats, injuries (2009+), depth charts (2001+), snap counts (2012+), Next Gen Stats, FTN charting (2022+)

Coverage by season, read live from the database: https://endzoneapi.com/data/historical-odds and https://endzoneapi.com/data/play-by-play

## Pricing

500 free credits on signup, no card. Plans: Starter $19/mo (10,000 credits), Pro $49/mo (30,000), Scale $149/mo (125,000). Wallet top-ups from $5, never expire. https://endzoneapi.com/pricing

## Attribution

Stats, schedules and play-by-play are from [nflverse](https://github.com/nflverse) (CC-BY 4.0; FTN charting CC-BY-SA 4.0). Not affiliated with the NFL or any team.

## License

MIT for the contents of this repository. Data terms as above.
