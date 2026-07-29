## sql_2_1

```sql
WITH taxi_base AS (
  SELECT
    EXTRACT(DAYOFWEEK FROM pickup_datetime) AS day_of_week,
    EXTRACT(HOUR FROM pickup_datetime) AS pickup_hour,
    trip_distance,
    fare_amount,
    tip_amount,
    ROW_NUMBER() OVER(
      PARTITION BY EXTRACT(DAYOFWEEK FROM pickup_datetime), \
      EXTRACT(HOUR FROM pickup_datetime)
      ORDER BY trip_distance DESC
    ) AS distance_rank
  FROM
    `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2020`
  WHERE
    pickup_datetime BETWEEN '2020-01-01' AND '2020-12-31'
    AND passenger_count > 0
    AND trip_distance > 0
    AND fare_amount > 0
)
SELECT
  day_of_week,
  pickup_hour,
  COUNT(*) AS total_trips,
  ROUND(AVG(trip_distance), 2) AS avg_distance,
  ROUND(AVG(SAFE_DIVIDE(tip_amount, fare_amount)) * 100, 2) AS avg_tip_rate
FROM
  taxi_base
WHERE
  TRUE
GROUP BY
  day_of_week,
  pickup_hour
ORDER BY
  day_of_week,
  pickup_hour;
```

## sql_2_2

```sql
SELECT
  EXTRACT(MONTH FROM pickup_datetime) AS pickup_month,
  EXTRACT(DAYOFWEEK FROM pickup_datetime) AS day_of_week,
  EXTRACT(HOUR FROM pickup_datetime) AS pickup_hour,
  COUNT(*) AS total_trips,
  ROUND(AVG(SAFE_DIVIDE(tip_amount, fare_amount)) * 100, 2) AS avg_tip_rate,
  ROUND(AVG(trip_distance), 2) AS avg_distance
FROM
  `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2020`
WHERE
  pickup_datetime BETWEEN '2020-01-01' AND '2020-12-31'
  AND passenger_count > 0
  AND trip_distance > 0
  AND fare_amount > 0
GROUP BY
  pickup_month,
  day_of_week,
  pickup_hour
ORDER BY
  pickup_month,
  day_of_week,
  pickup_hour;
```

## sql_2_3

```sql
SELECT
  EXTRACT(MONTH FROM pickup_datetime) AS pickup_month,
  EXTRACT(DAYOFWEEK FROM pickup_datetime) AS day_of_week,
  EXTRACT(HOUR FROM pickup_datetime) AS pickup_hour,
  COUNT(*) AS total_trips,
  ROUND(AVG(SAFE_DIVIDE(tip_amount, fare_amount)) * 100, 2) AS avg_tip_rate
FROM
  `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2020`
WHERE
  pickup_datetime BETWEEN '2020-01-01' AND '2020-12-31'
  AND passenger_count > 0
  AND trip_distance > 0
  AND tip_amount <= fare_amount
GROUP BY
  pickup_month, day_of_week, pickup_hour
ORDER BY
  pickup_month, day_of_week, pickup_hour;
```

## sql_2_4

```sql
WITH jfk_trips AS (
  SELECT
    pickup_datetime,
    LAG(pickup_datetime) OVER(
      ORDER BY pickup_datetime
    ) AS prev_pickup_datetime
  FROM
    `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2020`
  WHERE
    pickup_datetime BETWEEN '2020-01-01' AND '2020-12-31'
    AND pickup_location_id = '132'
    AND passenger_count > 0
),
calculated_intervals AS (
  SELECT
    EXTRACT(MONTH FROM pickup_datetime) AS pickup_month,
    TIMESTAMP_DIFF(pickup_datetime, prev_pickup_datetime, MINUTE) AS interval_minutes
  FROM
    jfk_trips
  WHERE
    prev_pickup_datetime IS NOT NULL
)
SELECT
  pickup_month,
  COUNT(*) AS total_trips,
  ROUND(AVG(interval_minutes), 1) AS avg_interval_minutes
FROM
  calculated_intervals
WHERE
  interval_minutes BETWEEN 0 AND 1440
GROUP BY
  pickup_month
ORDER BY
  pickup_month;
```
