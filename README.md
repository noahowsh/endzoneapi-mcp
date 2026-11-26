# EndzoneAPI MCP server



**The football data API.** Play-by-play with EPA and win probability, player and team stats, schedules, injuries, depth charts and closing lines for every NFL game since 1999. REST API and MCP server. 500 free credits every month, no card.

Website: https://endzoneapi.com?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp · Docs: https://endzoneapi.com/docs?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp · Get a key: https://endzoneapi.com/signup?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp

```
You:    "What was the closing spread for every Chiefs game last season?"
Claude: [calls get_games -> one call, every game with spread, total and moneyline]

You:    "Which offenses had the best EPA per play on third down in 2025?"
Claude: [calls get_plays with third_down filter -> EPA already attached]
```

## Try it now, no key

- **A free season of football data:** the last completed season as CSV (games, standings, player season totals and our ratings). [Download the zip](https://github.com/sports-data-hq/endzoneapi-mcp/releases/download/season-pack/endzoneapi-season-pack.zip) or browse [what's in it](data/README.md).
- **Open it in Colab:** [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/sports-data-hq/endzoneapi-mcp/blob/main/notebooks/endzoneapi-season-pack.ipynb) loads the pack and answers three questions in a minute.
- **Ask Claude without a key:** install the skills below; `season-pack-offline` answers from the pack until you connect.

**With a free key it does far more:** every earlier season, today's games, play-by-play, lineups, odds and player props with their history and movement, and every skill below running on live data. [Get a free key](https://endzoneapi.com/signup?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-try-now): 500 credits a month, no card.

## Skills

7 Claude skills for football ship in this repo, plus 15 analytics skills for models, backtests, calibration and comparing a model with the market. Every number in an answer comes from a live EndzoneAPI tool call, and odds are information only: no stakes, no picks.

Claude Code:

```
/plugin marketplace add sports-data-hq/endzoneapi-mcp
/plugin install nfl-data@endzoneapi
/plugin install sports-analytics-core@endzoneapi
```

Codex: `codex plugin marketplace add sports-data-hq/endzoneapi-mcp`. Any agent that reads skills: `npx skills add sports-data-hq/endzoneapi-mcp`.

| Skill | What it does |
|---|---|
| `nfl-epa-leaderboards` | EPA leaderboards with the filters stated and sample sizes, excluding kneels, spikes and penalties |
| `nfl-fourth-down-review` | Fourth-down decisions: what teams did in each situation and what the numbers say |
| `nfl-game-preview` | One-page preview: records, EPA and success rate, injuries, the line and what it implies |
| `nfl-injury-impact` | What an injury changes: depth chart, snap counts and with-and-without splits from play-by-play |
| `nfl-player-props-research` | Prop lines by sportsbook, the player's record against his line, movement and no-margin probability |
| `nfl-qb-efficiency` | Quarterback efficiency: EPA per dropback, success rate, completion over expected, air yards |
| `nfl-weekly-usage-report` | Weekly usage: snaps, targets, carries and red-zone work, and who absorbs a vacated role |

The analytics skills: `backtesting`, `data-pipeline`, `elo-engineering`, `feature-engineering`, `market-vs-model`, `model-building`, `nl-to-query`, `odds-market-analysis`, `prediction-tracker`, `probability-calibration`, `season-pack-offline`, `sports-data-preflight`, `sports-skill-router`, `visualization`, `walk-forward-validation`.

All nine sports' skills live in [sports-data-hq/sportsdatahq-skills](https://github.com/sports-data-hq/sportsdatahq-skills). The copies here are synced from there, so please open issues and pull requests there.

## Useful pages

- Odds this week: https://endzoneapi.com/odds?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp
- Player props this week (best price per player across books): https://endzoneapi.com/props?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp
- Props guides: [receiving yards history](https://endzoneapi.com/guides/nfl-receiving-yards-prop-history-python?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp), [anytime touchdown history](https://endzoneapi.com/guides/nfl-anytime-touchdown-prop-history-python?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp), [passing yards history](https://endzoneapi.com/guides/nfl-passing-yards-prop-history-python?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp)
- Guides: https://endzoneapi.com/guides?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp
- Docs: https://endzoneapi.com/docs?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp
- Pricing: https://endzoneapi.com/pricing?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp
- Data coverage: https://endzoneapi.com/data?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp
- College football (FBS): https://endzoneapi.com/college-football?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp
- FCS football: https://endzoneapi.com/fcs-football?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp

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

## Use it in

- **Claude, ChatGPT, Cursor, VS Code, Gemini CLI, Grok, Perplexity, Le Chat:** add `https://mcp.endzoneapi.com/mcp` as a remote MCP server. Step-by-step for each: https://endzoneapi.com/mcp?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp
- **Gemini CLI:** `gemini extensions install https://github.com/sports-data-hq/endzoneapi-mcp`
- **Cursor:** the [SportsDataHQ plugins for Cursor](https://github.com/sports-data-hq/sportsdatahq-cursor-plugins)
- **Google Sheets:** `=SDHQ("endzoneapi","list_teams")` fills a live table: https://sportsdatahq.com/sheets?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp
- **REST:** `POST https://mcp.endzoneapi.com/v1/<tool>` with an `x-api-key` header: https://endzoneapi.com/docs?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp

## Tools (26)

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
| `get_plays` | Play-by-play with EPA, win probability, situation filters |
| `get_odds` | Odds by game with bookmaker filters |
| `get_game_detail` | Full game detail with team stats, starting QBs, odds, weather, and officials |
| `get_head_to_head` | Matchup history with win/loss records and margins |
| `get_line_movement` | Time-series odds grouped by bookmaker |
| `get_player_props` | Player prop lines by sportsbook, opening and closing |
| `get_alt_lines` | Alternate spreads and totals by sportsbook: every point on the ladder, opening and closing, from the 2023 season to the current one |
| `get_prop_history` | A player's prop lines against the result, game by game |
| `get_prop_movement` | How one player's prop line moved before a game, every capture by sportsbook |
| `get_alt_player_props` | Alternate player prop ladders by sportsbook: every passing, rushing and receiving line a book posted for a player (history from the 2023 season) |
| `get_market_odds` | Opening and closing lines by sportsbook for the half and quarter markets (history from the 2023 season) |
| `get_game_probabilities` | What the market thinks of a game: average implied and fair probability for the moneyline, spreads and totals |
| `get_prop_probabilities` | What the market thinks of a game's player props: average implied and fair probability for each player and line |
| `get_prop_backtest` | How often a kind of player prop hit against what the market implied, by your filters, since 2023 |

## Data

- Play-by-play for every game since 1999 with EPA, win probability and model columns
- Closing spread, total and moneyline for every game since 1999; per-book odds from 2020 (up to 8 books, hourly in season from 2026)
- Player prop lines from 2023, captured live before every game
- Weekly player and team stats, injuries (2009+), depth charts (2001+), snap counts (2012+), Next Gen Stats, FTN charting (2022+)

Coverage by season, read live from the database: https://endzoneapi.com/data/historical-odds?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp, https://endzoneapi.com/data/player-props?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp and https://endzoneapi.com/data/play-by-play?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp

## Used by

[NoPunt](https://www.nopunt.com/), our NFL predictions site, runs on this API.

- [nfl-props-backtest](https://github.com/sports-data-hq/nfl-props-backtest): a season of NFL player props against what happened (calibration, over rates, line moves, anytime TD prices), in Python and R.

## Pricing

500 free credits every month, no card. Plans: Starter $19/mo (10,000 credits), Pro $49/mo (30,000), Scale $149/mo (125,000), Business $299/mo (300,000). Wallet top-ups from $5, kept while your account is open. https://endzoneapi.com/pricing?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp

## Attribution

Stats, schedules and play-by-play are from [nflverse](https://github.com/nflverse) (CC BY 4.0), modified and combined with our own models. Not affiliated with the NFL or any team.

## License

MIT for the contents of this repository. Data terms as above.

## Use with Codex

The SportsDataHQ skills marketplace also works in Codex (it reads `.agents/plugins/marketplace.json`):

```bash
codex plugin marketplace add sports-data-hq/sportsdatahq-skills
```

Install `nfl-data`, which bundles the football skills and the EndzoneAPI MCP server.

## Use with Gemini CLI

```bash
export ENDZONEAPI_API_KEY=your_key
gemini extensions install https://github.com/sports-data-hq/endzoneapi-mcp
```

The extension adds the EndzoneAPI MCP server (https://mcp.endzoneapi.com/mcp) and sends the key in the `x-api-key` header. Get a free key (500 credits a month) at https://endzoneapi.com/signup?utm_source=github&utm_medium=readme&utm_campaign=endzoneapi-mcp.
