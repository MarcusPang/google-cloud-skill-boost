## Create a new ML dataset with these skillful features
Query got cut off, here's the fix
```sql
-- create training dataset:
-- create a row for the winning team
CREATE OR REPLACE TABLE `bracketology.training_new_features` AS
WITH outcomes AS (
SELECT
  -- features
  season, -- 1994
  'win' AS label, -- our label
  win_seed AS seed, -- ranking # this time without seed even
  win_school_ncaa AS school_ncaa,
  lose_seed AS opponent_seed, -- ranking
  lose_school_ncaa AS opponent_school_ncaa
FROM `bigquery-public-data.ncaa_basketball.mbb_historical_tournament_games` t
WHERE season >= 2014
UNION ALL
-- create a separate row for the losing team
SELECT
-- features
  season, -- 1994
  'loss' AS label, -- our label
  lose_seed AS seed, -- ranking
  lose_school_ncaa AS school_ncaa,
  win_seed AS opponent_seed, -- ranking
  win_school_ncaa AS opponent_school_ncaa
FROM
`bigquery-public-data.ncaa_basketball.mbb_historical_tournament_games` t
WHERE season >= 2014
UNION ALL
-- add in 2018 tournament game results not part of the public dataset:
SELECT
  season,
  label,
  seed,
  school_ncaa,
  opponent_seed,
  opponent_school_ncaa
FROM
  `data-to-insights.ncaa.2018_tournament_results`
)
SELECT
o.season,
label,
-- our team
  seed,
  school_ncaa,
  -- new pace metrics (basketball possession)
  team.pace_rank,
  team.poss_40min,
  team.pace_rating,
  -- new efficiency metrics (scoring over time)
  team.efficiency_rank,
  team.pts_100poss,
  team.efficiency_rating,
-- opposing team
  opponent_seed,
  opponent_school_ncaa,
  -- new pace metrics (basketball possession)
  opp.pace_rank AS opp_pace_rank,
  opp.poss_40min AS opp_poss_40min,
  opp.pace_rating AS opp_pace_rating,
  opp.efficiency_rank AS opp_efficiency_rank,
  opp.pts_100poss AS opp_pts_100poss,
  opp.efficiency_rating AS opp_efficiency_rating,

  opp.pace_rank - team.pace_rank AS pace_rank_diff,
  opp.poss_40min - team.poss_40min AS pace_stat_diff,
  opp.pace_rating - team.pace_rating AS pace_rating_diff,
  opp.efficiency_rank - team.efficiency_rank AS eff_rank_diff,
  opp.pts_100poss - team.pts_100poss AS eff_stat_diff,
  opp.efficiency_rating - team.efficiency_rating AS eff_rating_diff

FROM outcomes AS o
LEFT JOIN `data-to-insights.ncaa.feature_engineering` AS team 
ON o.school_ncaa = team.team AND o.season = team.season
LEFT JOIN `data-to-insights.ncaa.feature_engineering` AS opp 
ON o.opponent_school_ncaa = opp.team AND o.season = opp.season
```

## Make predictions
Same for this
```sql
CREATE OR REPLACE TABLE `bracketology.ncaa_2019_tournament` AS
WITH team_seeds_all_possible_games AS (
  SELECT
    NULL AS label,
    team.school_ncaa AS school_ncaa,
    team.seed AS seed,
    opp.school_ncaa AS opponent_school_ncaa,
    opp.seed AS opponent_seed
  FROM `data-to-insights.ncaa.2019_tournament_seeds` AS team
  CROSS JOIN `data-to-insights.ncaa.2019_tournament_seeds` AS opp
  WHERE team.school_ncaa <> opp.school_ncaa
)
, add_in_2018_season_stats AS (
    SELECT 
        team_seeds_all_possible_games.*,
        (SELECT AS STRUCT * FROM `data-to-insights.ncaa.feature_engineering` WHERE school_ncaa = team AND season = 2018) AS team,
        (SELECT AS STRUCT * FROM `data-to-insights.ncaa.feature_engineering` WHERE opponent_school_ncaa = team AND season = 2018) AS opp
    FROM team_seeds_all_possible_games
)
SELECT 
    label,
    2019 AS season, 
    seed,
    school_ncaa,
    team.pace_rank,
    team.poss_40min,
    team.pace_rating,
    team.efficiency_rank,
    team.pts_100poss,
    team.efficiency_rating,
    opponent_seed,
    opponent_school_ncaa,
    opp.pace_rank AS opp_pace_rank,
    opp.pace_rating AS opp_pace_rating,
    opp.efficiency_rank AS opp_efficiency_rank,
    opp.pts_100poss AS opp_pts_100poss,
    opp.efficiency_rating AS opp_efficiency_rating,
      opp.pace_rank - team.pace_rank AS pace_rank_diff,
  opp.poss_40min - team.poss_40min AS pace_stat_diff,
  opp.pace_rating - team.pace_rating AS pace_rating_diff,
  opp.efficiency_rank - team.efficiency_rank AS eff_rank_diff,
  opp.pts_100poss - team.pts_100poss AS eff_stat_diff,
  opp.efficiency_rating - team.efficiency_rating AS eff_rating_diff

FROM add_in_2018_season_stats
```