---
name: dispatch
description: "Routes football analytics requests to the right 2-3 skills. Load this FIRST when an NFL, FBS or FCS data or betting-analytics request arrives and you are unsure which skills to activate. Prevents loading all skills when only a few are needed."
metadata:
  version: 1.0.0
  author: EndzoneAPI
---

# Skill Dispatch Guide

Match the request to a pattern. Load those skills. Do not load everything.

**Default data tool:** `endzoneapi-tool`. Load it alongside the analysis skill whenever a call is needed, for the exact names, parameters and credit costs.

Free tier: 500 credits every month at https://endzoneapi.com/signup.

## Where Does the Question Live?

| Signal | League route |
|--------|--------------|
| Team names like Chiefs, Bills; "NFL"; weeks 1 to 18 | NFL (default) |
| Ranked teams, bowls, "college football", FBS | `college-football` (route `/v1/ncaaf/`) |
| FCS, Montana State, Big Sky | `college-football` (route `/v1/fcs/`) |

## Persona Detection

| Signal | Persona | Start With |
|--------|---------|-----------|
| "Who plays this week?" / exploring | Explorer | `game-lookup` + `endzoneapi-tool` |
| "How good is this offense?" | Analyst | `epa-analysis` + `endzoneapi-tool` |
| "I want to build a model" | Builder | `feature-engineering` + `walk-forward-validation` |
| "Is there an edge on Sunday?" | Bettor | `odds-explorer` + `edge-detection` |
| "Is my model any good?" | Validator | `walk-forward-validation` + `probability-calibration` |
| "I am new to betting" | New bettor | `odds-analysis` + `odds-explorer` |

## Request Patterns to Skills

### "Who plays Sunday?" / "What was the score?" / "Standings" / "Playoff seeds"
1. `game-lookup`
2. `endzoneapi-tool` for `get_games`, `get_schedule`, `get_standings`

### "Preview Bills at Chiefs" / "Who has the edge in this game?"
1. `game-preview`
2. `epa-analysis` for the efficiency comparison
3. `odds-explorer` if the user wants book-level prices

### "Which offense is best?" / "Third-down EPA" / "Success rate" / "Play action"
1. `epa-analysis`
2. `endzoneapi-tool`

### "Tell me about [player]" / "Who leads in receiving yards?" / "Target share"
1. `player-scouting`
2. `next-gen-stats` if separation, time to throw, pressure or rush yards over expected come up

### "Does he get open?" / "Time to throw" / "Rush yards over expected" / "Drops"
1. `next-gen-stats`
2. `player-scouting` for counting stats

### "What is the line?" / "Best price" / "How did it move?" / "Alternate spread"
1. `odds-explorer`
2. `odds-analysis` for devig or implied probability

### "What does -150 mean?" / "Remove the vig" / "Fair price" / "Break-even rate"
1. `odds-analysis`

### "Is this bet +EV?" / "How much to stake?"
1. `edge-detection`
2. `odds-analysis`
3. `probability-calibration` if probabilities come from a model

### "Would this strategy have worked?" / "Backtest" / "CLV"
1. `backtesting`
2. `walk-forward-validation`
3. `odds-analysis`

### "Will he go over 250.5?" / "Anytime TD price" / "Prop history"
1. `prop-modeling`
2. `player-scouting` for role, `odds-analysis` for the price

### "Build me a model" / "What features?" / "Elo"
1. `feature-engineering`
2. `walk-forward-validation`
3. `probability-calibration` after predictions exist

### "Is my model overconfident?" / "Brier score"
1. `probability-calibration`
2. `walk-forward-validation`

### "College football" / "Ranked teams" / "Bowl lines" / "FCS"
1. `college-football`
2. `odds-explorer` for lines, same math applies
3. Warn early that college has no EPA, tracking, injuries or depth charts

### "How much will this cost?" / "Which tool?" / "REST or MCP?"
1. `endzoneapi-tool`

## Multi-Skill Workflows

**Weekly betting card:**
1. `game-lookup` for the slate (2 credits)
2. `game-preview` on the games of interest
3. `edge-detection` where a model disagrees with the market
4. `odds-explorer` to place the best price

**Build and validate a model:**
1. `feature-engineering`
2. `walk-forward-validation`
3. `probability-calibration`
4. `backtesting`

**Prop session:**
1. `player-scouting` for role
2. `prop-modeling` for the projection and price
3. `odds-analysis` to devig

## Credit Awareness

Before loading a skill that will make several calls, estimate the cost and say it. Expensive calls: `get_line_movement` (25), `get_odds` for a season (10 per game), `get_plays` (10 per page), `get_player_props` for many games (10 each). A season of consensus lines through `get_games` is 5. Use `estimate_only` on `get_odds` and `get_player_props` when a pull could be large.

## Rules the Whole Pack Follows

- Edges are estimates. No guarantees, no "locks".
- Beating the closing line over many bets is better evidence than winning a few.
- Small samples first: say n.
- Never invent a tool. If it is not in `endzoneapi-tool`, it does not exist.
