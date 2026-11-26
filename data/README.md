# EndzoneAPI season pack

The last completed season of football as CSV, free to use under the [EndzoneAPI Sample Data Licence](../DATA-LICENSE.md). Use it in anything you build, credit EndzoneAPI, and don't republish the files themselves.

**Download:** the latest `endzoneapi-season-pack.zip` from this repository's [season-pack release](https://github.com/sports-data-hq/endzoneapi-mcp/releases/tag/season-pack). `preview/` holds the first 100 rows of a few files.

Seasons: 2025. Season naming: the year the regular season starts, so 2025 is 2025-26. Competitions, by their API id: ncaaf, nfl. No odds, props or play-by-play: those, every earlier season and live data are on the API, free to start at https://endzoneapi.com/signup?utm_source=github&utm_medium=pack&utm_campaign=endzoneapi-pack.

## Files

| File | Rows |
|---|---|
| `endzoneapi-ncaaf-2025-games.csv` | 934 |
| `endzoneapi-ncaaf-2025-players.csv` | 12,024 |
| `endzoneapi-ncaaf-2025-ratings.csv` | 230 |
| `endzoneapi-ncaaf-2025-standings.csv` | 230 |
| `endzoneapi-nfl-2025-games.csv` | 285 |
| `endzoneapi-nfl-2025-players.csv` | 1,803 |
| `endzoneapi-nfl-2025-qb-epa.csv` | 58 |
| `endzoneapi-nfl-2025-standings.csv` | 32 |
| `endzoneapi-nfl-2025-team-epa.csv` | 32 |

## Columns

- **games**: `league`, `season`, `stage`, `week`, `date`, `home_team`, `away_team`, `home_score`, `away_score`, `home_rank`, `away_rank`, `periods`, `neutral_site`, `conference_game`, `note`, `venue`
- **players**: `league`, `season`, `player`, `team`, `position`, `games`, `pass_cmp`, `pass_att`, `pass_yds`, `pass_td`, `pass_int`, `rush_att`, `rush_yds`, `rush_td`, `rec`, `rec_yds`, `rec_td`, `tackles`, `sacks`, `tackles_for_loss`, `def_int`, `passes_defended`, `fg_made`, `fg_att`, `xp_made`, `xp_att`
- **qb-epa**: `league`, `season`, `qb`, `team`, `dropbacks`, `total_epa`, `epa_per_dropback`, `success_rate`, `cpoe`
- **ratings**: `league`, `season`, `team`, `conference`, `elo_end_of_season`
- **standings**: `league`, `season`, `conference`, `team`, `games`, `wins`, `losses`, `conf_wins`, `conf_losses`, `points_for`, `points_against`, `point_diff`
- **team-epa**: `league`, `season`, `team`, `offensive_plays`, `epa_per_play`, `pass_epa_per_play`, `rush_epa_per_play`, `success_rate`, `defense_epa_per_play_allowed`

Credit line for anything you publish: **Data: EndzoneAPI (endzoneapi.com)**
