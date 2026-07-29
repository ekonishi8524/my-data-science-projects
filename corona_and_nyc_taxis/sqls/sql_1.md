## query_1_1

```sql
SELECT
  DATE(pickup_datetime) AS ride_date,
  COUNT(*) AS total_trips,
  ROUND(AVG(fare_amount), 2) AS avg_fare,
  ROUND(AVG(tip_amount), 2) AS avg_tip,
  MAX(fare_amount) AS max_fare
FROM
  `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2020`
WHERE
  fare_amount > 0
  AND pickup_datetime BETWEEN '2020-01-01' AND '2020-12-31'
GROUP BY
  ride_date
ORDER BY
  ride_date ASC;
```

## query_1_2

```sql
SELECT
  DATE(pickup_datetime) AS ride_date,
  COUNT(*) AS total_trips,
  ROUND(AVG(fare_amount), 2) AS avg_fare,
  ROUND(AVG(tip_amount), 2) AS avg_tip,
  ROUND(AVG(trip_distance), 2) AS avg_distance,
  ROUND(AVG(TIMESTAMP_DIFF(dropoff_datetime, pickup_datetime, SECOND) / 60.0), 1)\
   AS avg_duration_minutes
FROM
  `bigquery-public-data.new_york_taxi_trips.tlc_yellow_trips_2020`
WHERE
  fare_amount > 0
  AND trip_distance > 0
  AND dropoff_datetime > pickup_datetime
  AND TIMESTAMP_DIFF(dropoff_datetime, pickup_datetime, HOUR) < 24
  AND pickup_datetime BETWEEN '2020-01-01' AND '2020-12-31'
GROUP BY
  ride_date
ORDER BY
  ride_date ASC;
```
