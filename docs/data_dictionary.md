# Data Dictionary

## Silver Layer — `silver/nyc_taxi`

| Column | Type | Source | Description |
|---|---|---|---|
| `tpep_pickup_datetime` | Timestamp | Raw | Trip start datetime |
| `tpep_dropoff_datetime` | Timestamp | Raw | Trip end datetime |
| `passenger_count` | Integer | Raw | Number of passengers (1–6) |
| `trip_distance` | Double | Raw | Distance in miles |
| `fare_amount` | Double | Raw | Base fare charged |
| `total_amount` | Double | Raw | Total including tips and fees |
| `tip_amount` | Double | Raw | Tip amount |
| `PULocationID` | Integer | Raw | TLC pickup zone ID |
| `DOLocationID` | Integer | Raw | TLC dropoff zone ID |
| `payment_type` | Integer | Raw | Numeric payment code |
| `trip_duration_mins` | Double | Derived | Dropoff minus pickup in minutes |
| `hour_of_day` | Integer | Derived | Pickup hour (0–23) |
| `day_of_week` | Integer | Derived | 1=Sunday, 7=Saturday |
| `is_weekend` | Boolean | Derived | True if Saturday or Sunday |
| `time_of_day` | String | Derived | morning / afternoon / evening / night |
| `tip_percentage` | Double | Derived | Tip as % of fare |
| `speed_mph` | Double | Derived | Average speed for the trip |
| `payment_type_desc` | String | Derived | Human-readable payment label |
| `year` | String | Derived | Partition column |
| `month` | String | Derived | Partition column |

## Gold Layer — `gold/daily_revenue`

| Column | Type | Description |
|---|---|---|
| `trip_date` | Date | Calendar date |
| `total_trips` | Long | Count of trips on that date |
| `total_fare` | Double | Sum of fare_amount |
| `total_revenue` | Double | Sum of total_amount |
| `avg_fare` | Double | Average fare per trip |
| `avg_tip_pct` | Double | Average tip percentage |
| `avg_duration_mins` | Double | Average trip duration |
| `avg_distance_miles` | Double | Average trip distance |
