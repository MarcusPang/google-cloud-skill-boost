# Query 1

```sql
SELECT SUM(cumulative_confirmed) AS total_cases_worldwide
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date = "2020-04-15"
```

# Query 2

```sql
SELECT COUNT(*) AS count_of_states
FROM (
    SELECT subregion1_name AS state,
      SUM(cumulative_deceased) AS death_count
    FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
    WHERE country_name = "United States of America"
      AND date = '2020-04-10'
      AND subregion1_name IS NOT NULL
    HAVING SUM(death_count) > 100
    GROUP BY subregion1_name
  )
```

# Query 3

```sql
SELECT subregion1_name as state,
  SUM(cumulative_confirmed) as total_confirmed_cases
FROM bigquery - public - data.covid19_open_data.covid19_open_data
WHERE date = "2020-04-10"
  AND country_code = "US"
  AND subregion1_name IS NOT NULL
GROUP BY subregion1_name
HAVING total_confirmed_cases > 1000
ORDER BY total_confirmed_cases DESC
```

# Query 4

```sql
SELECT SUM(cumulative_confirmed) as total_confirmed_cases,
  SUM(cumulative_deceased) as total_deaths,
  (
    SUM(cumulative_deceased) / SUM(cumulative_confirmed)
  ) * 100 as case_fatality_ratio
FROM bigquery - public - data.covid19_open_data.covid19_open_data
WHERE date BETWEEN "2020-04-01" AND "2020-04-30"
  AND country_name = "Italy"
```

# Query 5

```sql
SELECT date,
  FROM bigquery - public - data.covid19_open_data.covid19_open_data
WHERE country_name = "Italy"
  AND cumulative_deceased > 10000
ORDER BY date
LIMIT 1
```

# Query 6

```sql
WITH india_cases_by_date AS (
  SELECT date,
    SUM(cumulative_confirmed) AS cases
  FROM bigquery - public - data.covid19_open_data.covid19_open_data
  WHERE country_name = "India"
    AND date between '2020-02-21' and '2020-03-15'
  GROUP BY date
  ORDER BY date ASC
),
india_previous_day_comparison AS (
  SELECT date,
    cases,
    LAG(cases) OVER(
      ORDER BY date
    ) AS previous_day,
    cases - LAG(cases) OVER(
      ORDER BY date
    ) AS net_new_cases
  FROM india_cases_by_date
)
SELECT COUNT(date)
FROM india_previous_day_comparison
WHERE net_new_cases = 0
```

# Query 7

```sql
WITH cases_by_date AS (
  SELECT date,
    SUM(cumulative_confirmed) AS cases
  FROM bigquery - public - data.covid19_open_data.covid19_open_data
  WHERE country_name = "United States of America"
    AND date between '2020-03-22' and '2020-04-20'
  GROUP BY date
  ORDER BY date ASC
),
previous_day_comparison AS (
  SELECT date,
    cases,
    LAG(cases) OVER(
      ORDER BY date
    ) AS previous_day,
    cases - LAG(cases) OVER(
      ORDER BY date
    ) AS net_new_cases,
    (
      cases - LAG(cases) OVER(
        ORDER BY date
      )
    ) * 100 /(
      LAG(cases) OVER(
        ORDER BY date
      )
    ) AS percentage_increase
  FROM cases_by_date
)
SELECT date as Date,
  cases as Confirmed_Cases_On_Day,
  previous_day as Confirmed_Cases_Previous_Day,
  percentage_increase as Percentage_Increase_In_Cases
FROM previous_day_comparison
WHERE percentage_increase > 10
```

# Query 8

```sql
SELECT country_name as country,
  SUM(cumulative_recovered) as recovered_cases,
  SUM(cumulative_confirmed) as confirmed_cases,
  SUM(cumulative_recovered) / SUM(cumulative_confirmed) * 100 as recovery_rate
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date = '2020-05-10'
GROUP BY country_name
HAVING confirmed_cases > 50000
ORDER BY recovery_rate DESC
LIMIT 10

/* Data Studio */
SELECT date,
  SUM(cumulative_confirmed) AS country_cases,
  SUM(cumulative_deceased) AS country_deaths
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date BETWEEN '2020-03-15' AND '2020-04-30'
  AND country_name = 'United States of America'
GROUP BY date
```
