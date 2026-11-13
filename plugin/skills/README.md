# EndzoneAPI skills

Sixteen Claude skills for NFL, FBS and FCS football analytics. Each folder holds one `SKILL.md`. Tool names, parameters and credit costs match the live EndzoneAPI catalog. Start with `dispatch` if you are not sure which to load, and `endzoneapi-tool` for the full tool reference.

Free tier: 500 credits every month, no card: https://endzoneapi.com/signup

| Skill | Category | What it does |
|-------|----------|--------------|
| [`game-lookup`](game-lookup/SKILL.md) | Data Exploration | Games, schedules, scores, standings and playoff seeding |
| [`player-scouting`](player-scouting/SKILL.md) | Data Exploration | Player search, weekly stats, leaders, snaps and usage |
| [`odds-explorer`](odds-explorer/SKILL.md) | Data Exploration | Multi-book lines, line movement, alternate ladders, key numbers |
| [`game-preview`](game-preview/SKILL.md) | Data Exploration | Pre-game brief: form, efficiency, injuries, rest, weather, lines |
| [`epa-analysis`](epa-analysis/SKILL.md) | Football Analytics | EPA per play, success rate, situational splits, team and unit rankings |
| [`next-gen-stats`](next-gen-stats/SKILL.md) | Football Analytics | Time to throw, CPOE, rush yards over expected, separation, pressure |
| [`college-football`](college-football/SKILL.md) | Football Analytics | FBS and FCS: ranks, Elo, plays, lines, and what differs from the NFL |
| [`feature-engineering`](feature-engineering/SKILL.md) | Methodology | Leak-free rolling EPA, opponent adjustment, QB and rest features, Elo |
| [`walk-forward-validation`](walk-forward-validation/SKILL.md) | Methodology | Time-ordered model testing against the closing line baseline |
| [`probability-calibration`](probability-calibration/SKILL.md) | Methodology | Reliability tables, Brier, log loss, Platt scaling with small samples |
| [`odds-analysis`](odds-analysis/SKILL.md) | Methodology | American odds, devigging methods, hold, EV and break-even rates |
| [`edge-detection`](edge-detection/SKILL.md) | Betting | Model vs devigged market, EV, shrinkage and fractional Kelly |
| [`backtesting`](backtesting/SKILL.md) | Betting | Honest strategy tests on closing lines, ROI intervals, CLV vs Pinnacle |
| [`prop-modeling`](prop-modeling/SKILL.md) | Betting | Player props: usage, game script, distributions, prop history over rates |
| [`dispatch`](dispatch/SKILL.md) | Workflow | Routes a football request to the right 2-3 skills |
| [`endzoneapi-tool`](endzoneapi-tool/SKILL.md) | Infrastructure | Every tool, credit cost, coverage by season, REST and MCP setup |

College football and FCS use the `/v1/ncaaf/` and `/v1/fcs/` routes. See `college-football` for what those leagues do and do not carry (no EPA, tracking data, injuries or depth charts).

Betting skills frame edges as estimates. Nothing here is a guarantee, closing-line value and sample size matter more than any one result, and past results do not predict future returns.
