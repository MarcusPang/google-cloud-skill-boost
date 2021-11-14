# Task 1
Dataset name = marcus

# Task 2
```sql
CREATE OR REPLACE MODEL marcus.location_model
OPTIONS
  (model_type='linear_reg', labels=['duration_minutes']) AS
SELECT
    start_station_name,
    EXTRACT(HOUR FROM start_time) AS start_hour,
    EXTRACT(DAYOFWEEK FROM start_time) AS day_of_week,
    duration_minutes,
    address as location
FROM
    `bigquery-public-data.marcus_bikeshare.bikeshare_trips` AS trips
JOIN
    `bigquery-public-data.marcus_bikeshare.bikeshare_stations` AS stations
ON
    trips.start_station_name = stations.name
WHERE
    EXTRACT(YEAR FROM start_time) = 2016
    AND duration_minutes > 0
```

# Task 3
```sql
CREATE OR REPLACE MODEL marcus.subscriber_model
OPTIONS
(model_type='linear_reg', labels=['duration_minutes']) AS
SELECT
    start_station_name,
    EXTRACT(HOUR FROM start_time) AS start_hour,
    subscriber_type,
    duration_minutes
FROM `bigquery-public-data.marcus_bikeshare.bikeshare_trips` AS trips
WHERE EXTRACT(YEAR FROM start_time) = 2016
```

# Task 4
```sql
-- first
SELECT
SQRT(mean_squared_error) AS rmse,
mean_absolute_error
FROM
ML.EVALUATE(MODEL marcus.location_model, (
SELECT
    start_station_name,
    EXTRACT(HOUR FROM start_time) AS start_hour,
    EXTRACT(DAYOFWEEK FROM start_time) AS day_of_week,
    duration_minutes,
    address as location
FROM
    `bigquery-public-data.marcus_bikeshare.bikeshare_trips` AS trips
JOIN
`bigquery-public-data.marcus_bikeshare.bikeshare_stations` AS stations
ON
    trips.start_station_name = stations.name
WHERE EXTRACT(YEAR FROM start_time) = 2019)
)

-- second
SELECT
SQRT(mean_squared_error) AS rmse,
mean_absolute_error
FROM
ML.EVALUATE(MODEL marcus.subscriber_model, (
SELECT
    start_station_name,
    EXTRACT(HOUR FROM start_time) AS start_hour,
    subscriber_type,
    duration_minutes
FROM
    `bigquery-public-data.marcus_bikeshare.bikeshare_trips` AS trips
WHERE
    EXTRACT(YEAR FROM start_time) = 2019)
)
```
# Task 5
```sql
-- first
SELECT
start_station_name,
COUNT(\*) AS trips
FROM
`bigquery-public-data.marcus_bikeshare.bikeshare_trips`
WHERE
EXTRACT(YEAR FROM start_time) = 2019
GROUP BY
start_station_name
ORDER BY
trips DESC

-- second
SELECT AVG(predicted_duration_minutes) AS average_predicted_trip_length
FROM ML.predict(MODEL marcus.subscriber_model, (
SELECT
    start_station_name,
    EXTRACT(HOUR FROM start_time) AS start_hour,
    subscriber_type,
    duration_minutes
FROM
`bigquery-public-data.marcus_bikeshare.bikeshare_trips`
WHERE
EXTRACT(YEAR FROM start_time) = 2019
AND subscriber_type = 'Single Trip'
AND start_station_name = '21st & Speedway @PCL'))
```