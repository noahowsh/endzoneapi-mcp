# EndzoneAPI MCP server

[![Smithery](https://img.shields.io/badge/Smithery-listed-ff5601?style=flat-square)](https://smithery.ai/servers/noahowsh/EndzoneAPI)

**The football data API.** Play-by-play with EPA and win probability, player and team stats, schedules, injuries, depth charts and closing lines for every NFL game since 1999. REST API and MCP server. 500 free credits every month, no card.

Website: https://endzoneapi.com · Docs: https://endzoneapi.com/docs · Get a key: https://endzoneapi.com/signup

```
You:    "What was the closing spread for every Chiefs game last season?"
Claude: [calls get_games -> one call, every game with spread, total and moneyline]

You:    "Which offenses had the best EPA per play on third down in 2025?"
Claude: [calls get_plays with third_down filter -> EPA already attached]
```

## Skills

16 free Claude skills turn Claude into a football analyst wired to these tools: EPA per play and success rate, Next Gen Stats, weekly player stats, odds and devigging, prop modeling, college football and FCS, and honest backtesting. They are plain markdown folders.

```bash
git clone https://github.com/noahowsh/endzoneapi-mcp.git
```

Or install them as a Claude Code plugin:

```
/plugin marketplace add noahowsh/endzoneapi-mcp
/plugin install endzoneapi-skills@endzoneapi
```

Or with the skills CLI: `npx skills add noahowsh/endzoneapi-mcp`.

Point Claude (Claude Code, or any client that loads skills) at the `skills/` folder, and connect the MCP server above so the skills can call live data. The `dispatch` skill routes a request to the right two or three skills. Full list: [skills/README.md](skills/README.md)

```
You:    "Which offenses were best on third down in 2025, and how many plays is that based on?"
Claude: [loads epa-analysis + endzoneapi-tool, calls get_plays with third_down and a limit that
         holds the split, reports EPA per play, success rate and the play count]
```

## Useful pages

- Odds this week: https://endzoneapi.com/odds
- Player props this week (best price per player across books): https://endzoneapi.com/props
- Props guides: [receiving yards history](https://endzoneapi.com/guides/nfl-receiving-yards-prop-history-python), [anytime touchdown history](https://endzoneapi.com/guides/nfl-anytime-touchdown-prop-history-python), [passing yards history](https://endzoneapi.com/guides/nfl-passing-yards-prop-history-python)
- Guides: https://endzoneapi.com/guides
- Docs: https://endzoneapi.com/docs
- Pricing: https://endzoneapi.com/pricing
- Data coverage: https://endzoneapi.com/data
- College football (FBS): https://endzoneapi.com/college-football
- FCS football: https://endzoneapi.com/fcs-football
- Compare: [ESPN API alternative](https://endzoneapi.com/compare/espn-api-alternative), [nflfastR API](https://endzoneapi.com/compare/nflfastr-api), [SportsDataIO alternative](https://endzoneapi.com/compare/sportsdataio-alternative)

## Python

The official Python client is on PyPI: [endzoneapi](https://pypi.org/project/endzoneapi/).

```bash
pip install endzoneapi
export ENDZONEAPI_API_KEY=sk_live_...
```

```python
from endzoneapi import Client, to_dataframe

client = Client()

print(client.get_player_props(game_id="2025_01_KC_LAC", estimate_only=True))

props = client.get_player_props(
    game_id="2025_01_KC_LAC",
    market="player_pass_yds",
    bookmaker="draftkings",
)
df = to_dataframe(props, key="props")
print(df[["player_name", "line", "over_odds", "under_odds"]])
```

Typed errors for a bad key, no credits left, not found and rate limits, automatic retries, and `to_dataframe` for pandas.

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

## Tools (18)

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
| `get_player_props` | Player prop lines by sportsbook, opening and closing: passing, rushing and receiving yards, TDs, receptions, anytime TD |
| `get_prop_history` | One player against his prop line over time for one market, with hit rate |
| `get_alt_lines` | Alternate spreads and totals by sportsbook, every point on each ladder with its price |

## Data

- Play-by-play for every game since 1999 with EPA, win probability and the nflfastR model columns
- Closing spread, total and moneyline for every game since 1999; per-book odds from 2020 (up to 8 books, hourly in season from 2026)
- Player prop lines from 2023, captured live before every game
- Weekly player and team stats, injuries (2009+), depth charts (2001+), snap counts (2012+), Next Gen Stats, FTN charting (2022+)

Coverage by season, read live from the database: https://endzoneapi.com/data/historical-odds, https://endzoneapi.com/data/player-props and https://endzoneapi.com/data/play-by-play

## Used by

[NoPunt](https://www.nopunt.com/), our NFL predictions site, runs on this API.

## Pricing

500 free credits every month, no card. Plans: Starter $19/mo (10,000 credits), Pro $49/mo (30,000), Scale $149/mo (125,000). Wallet top-ups from $5, kept while your account is open. https://endzoneapi.com/pricing

## Attribution

Stats, schedules and play-by-play are from [nflverse](https://github.com/nflverse) (CC BY 4.0), modified and combined with our own models. Not affiliated with the NFL or any team.

## License

MIT for the contents of this repository. Data terms as above.

## Use with Codex

Add this repo as a Codex plugin marketplace, then install the plugin:

```bash
codex plugin marketplace add noahowsh/endzoneapi-mcp
```

Then open the plugin directory in Codex and install `endzoneapi-skills` from the `endzoneapi` marketplace. The plugin bundles the skills in `skills/` and the EndzoneAPI MCP server (https://mcp.endzoneapi.com/mcp). Set your key first (free at https://endzoneapi.com/signup):

```bash
export ENDZONEAPI_API_KEY=your_key
```

Codex sends it as a bearer token.

## Use with Gemini CLI

```bash
export ENDZONEAPI_API_KEY=your_key
gemini extensions install https://github.com/noahowsh/endzoneapi-mcp
```

The extension adds the EndzoneAPI MCP server (https://mcp.endzoneapi.com/mcp) and sends the key in the `x-api-key` header. Get a free key (500 credits a month) at https://endzoneapi.com/signup.
