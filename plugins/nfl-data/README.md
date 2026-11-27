# EndzoneAPI: football skills

Football analysis skills on EndzoneAPI: game previews, player and team research, and the community's standard metrics, with every number pulled live from the EndzoneAPI MCP server.

## Skills

- `nfl-dfs-lineup`: Use when someone has an NFL daily fantasy slate and wants lineups built.
- `nfl-epa-leaderboards`: Use when the user wants NFL rankings built on expected points added.
- `nfl-fantasy-draft-board`: Use when someone is preparing a fantasy football draft.
- `nfl-fantasy-start-sit`: Use when a fantasy football manager asks who to start.
- `nfl-fantasy-trade-value`: Use when a fantasy football manager asks about a trade.
- `nfl-fantasy-waivers`: Use when a fantasy football manager is looking for pickups.
- `nfl-fourth-down-review`: Use when the user wants to review fourth-down decisions in the NFL.
- `nfl-game-preview`: Use when the user wants a preview or breakdown of one NFL game.
- `nfl-injury-impact`: Use when the user asks what an NFL injury or absence changes for a team, in fan or analyst wording.
- `nfl-line-movement`: Use when the user asks how an NFL spread, total or moneyline moved during the week, in fan wording.
- `nfl-player-props-research`: Use when the user asks about NFL player prop lines or how a player did against his line, in fan or analyst wording.
- `nfl-qb-efficiency`: Use when the user wants an efficiency profile of an NFL quarterback or a comparison of two.
- `nfl-weekly-usage-report`: Use when the user has weekly NFL fantasy or usage questions, in fan or analyst wording.

## Data it sends

The skills call the EndzoneAPI MCP server at `mcp.endzoneapi.com`. Each call sends the tool name and its inputs (teams, players, dates, seasons) with your EndzoneAPI API key, and returns data. Nothing from your files or conversation is sent beyond what a tool call needs. Calls use credits on your EndzoneAPI account; a free key at https://endzoneapi.com/signup includes monthly credits.

Privacy policy: https://endzoneapi.com/privacy

## Install

```
/plugin marketplace add sports-data-hq/sportsdatahq-skills
/plugin install nfl-data@sportsdatahq
```

More: https://endzoneapi.com/skills. Licence: MIT (see LICENSE).
