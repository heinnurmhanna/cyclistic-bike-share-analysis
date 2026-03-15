# CYCLISTIC BIKE SHARE ANALYSIS #
Data analysis project exploring how casual riders and members use Cyclistic bikes differently.

Business task
Cyclistic wants to **increase the number of annual memberships**. 
The goal of this analysis is to **understand how casual riders and annual members use Cyclistic bikes differently.**

Stakeholders
- Lily Moreno (Marketing Director)
- Cyclistic Marketing Team
- Cyclistic Executive Team

## 1. ANALYSE ##

Three questions will guide the future marketing program:
1. How do annual members and casual riders use Cyclistic bikes differently?
2. Why would casual riders buy Cyclistic annual memberships?
3. How can Cyclistic use digital media to influence casual riders to become members?

Lily Moreno has assigned me the first question to answer: **How do annual members and casual riders use Cyclistic bikes differently?**

## 2. PREPARE ##

In this stage, the data required for the analysis was collected, organized, and evaluated for credibility. The dataset used in this project is the Cyclistic historical bike trip data, made available by Motivate International Inc. The data represents trip information for Cyclistic bike-share users and contains details about ride duration, start and end stations, bike types, and user types (annual members and casual riders). This dataset enables the analysis of usage patterns between different customer groups.

The data was downloaded as multiple CSV files covering the previous 12 months of trip data. Each file represents trips recorded during a specific month. All files were stored locally in a structured project folder to ensure proper organization and reproducibility of the analysis.

The dataset includes the following key variables used in this analysis:
- _ride_id_ - unique identifier for each ride
- _rideable_type_ - tyoe of bike used
- _started_at_ - timestamp when the ride started
- _ended_at_ – timestamp when the ride ended
- _start_station_name_ – name of the starting station
- _end_station_name_ – name of the ending station
- _member_casual_ – user type (annual member or casual rider)

Before proceeding to data processing, the dataset was reviewed to ensure it met the **ROCCC criteria (Reliable, Original, Comprehensive, Current, and Cited).** The data is considered reliable because it originates from the official Cyclistic bike-share system and is publicly available for analytical purposes.

After verifying the dataset structure and credibility, the data was ready to be cleaned and transformed in the next stage of the analysis.

## 3. PROCESS ##

### STEP 1. COMBINE THE DATASETS ###

All monthly trip datasets were combined into a single table in Google BigQuery to simplify analysis and ensure the full dataset can be queried efficiently. This unified table contains **5,552,092 rows**, representing all recorded bike trips for the selected period.

### STEP 2. INITIAL DATA EXPLORATION ###

The first step was to explore the dataset and verify that the main fields contain expected values.

**Total number of rows**

```sql
SELECT COUNT(*) AS total_rows
FROM `cyclistic-bike-share-heinnurm.bike_share_data.trips_2025`
```

Result: total_rows = 5,552,092

**Verify rider types**

Cyclistic differentiates between casual riders and annual members, which is central to the business question.

```sql
SELECT DISTINCT member_casual
FROM `cyclistic-bike-share-heinnurm.bike_share_data.trips_2025`
```

Result: member, casual

This confirms that the dataset contains only the two expected rider types.

**Verify bike types**

The available bike types were also checked.

```sql
SELECT DISTINCT rideable_type
FROM `cyclistic-bike-share-heinnurm.bike_share_data.trips_2025`
```

Result: electric_bike, classic_bike

**Verify date range**

Trip timestamps were examined to ensure that the data falls within the expected time range.

```sql
SELECT 
  MIN(started_at) AS min_start,
  MAX(started_at) AS max_start,
  MIN(ended_at) AS min_end,
  MAX(ended_at) AS max_end
FROM `cyclistic-bike-share-heinnurm.bike_share_data.trips_2025`
```

This check ensures there are no obvious timestamp anomalies.

### STEP 3. DUPLICATE CHECK ###

To verify data integrity, duplicate ride IDs were checked.

```sql
SELECT 
  COUNT(*) AS total_rows,
  COUNT(DISTINCT ride_id) AS distinct_ride_ids
FROM `cyclistic-bike-share-heinnurm.bike_share_data.trips_2025`
```

**Result:**
- total_rows = 5,552,092
- distinct_ride_ids = 5,552,092

Since both values match, the dataset contains no duplicate ride records.

### STEP 4. IDENTIFY MISSING VALUES ###

Next, the dataset was checked for NULL values in key columns.

```sql
SELECT
  COUNTIF(start_station_name IS NULL) AS null_start_station,
  COUNTIF(end_station_name IS NULL) AS null_end_station,
  COUNTIF(started_at IS NULL) AS null_started_at,
  COUNTIF(ended_at IS NULL) AS null_ended_at,
  COUNTIF(member_casual IS NULL) AS null_member_type
FROM `cyclistic-bike-share-heinnurm.bike_share_data.trips_2025`
```

A relatively large number of records contain missing values for _start_station_name_ and _end_station_name_. This is common in bike-share datasets and may occur when rides begin or end outside official docking stations or when station metadata is unavailable.

However, the critical analytical fields are complete:
- _started_at_ – no missing values
- _ended_at_ – no missing values
- _member_casual_ – no missing values

Because the main objective of the analysis is to compare **casual riders and annual members, the missing station names do not significantly affect the core analysis.**

Therefore, these rows will be **retained in the dataset.**

## 4. ANALYSE ##

To answer the business question “How do annual members and casual riders use Cyclistic bikes differently?”, the trip dataset was analyzed using SQL in BigQuery.

The analysis focused on identifying behavioral differences between casual riders and annual members, including ride frequency, ride duration, and temporal usage patterns. Understanding these differences is essential for Cyclistic’s marketing team, whose goal is to convert casual riders into annual members. 

To uncover these insights, the dataset was first prepared by creating additional variables such as ride_length, day_of_week, and month. These variables enabled the analysis of ride duration, weekly usage patterns, and seasonal trends.

The analysis then examined several key metrics:
- total number of rides by rider type
- average ride duration
- ride duration distribution
- ride frequency by weekday
- average ride length by weekday

These metrics help reveal how each rider group interacts with the bike-share system and provide valuable insights for developing targeted marketing strategies aimed at increasing annual memberships.

### 4.1. Created the ride_length column ###

A new column called _ride_length_ was created by calculating the difference between _ended_at_ and _started_at_. This metric represents how long each bike ride lasted.

Why this is important: 
- Ride duration helps identify how riders use Cyclistic bikes
- It allows comparison between casual riders and annual members
- Differences in ride duration can indicate whether bikes are used for commuting or leisure activities

Additional variables were also created:
- _day_of_week_ – identifies which day a ride started, helping reveal weekly usage patterns
- _month_ – helps identify seasonal trends in bike usage

These variables allow deeper behavioral analysis over time.

### 4.2. Total number of rides by user type ###

The number of rides was calculated for each user group.

```sql
SELECT
  member_casual,
  COUNT(*) AS total_rides
FROM `cyclistic-bike-share-heinnurm.bike_share_data.cleaned_trips`
GROUP BY member_casual
```

| User type | Total rides |
|-----------|-------------:|
| Casual riders | 2,000,084 |
| Annual members | 3,551,979 |

This shows that annual members generate the majority of rides in the system.
This supports Cyclistic's business strategy, since annual members are the most profitable customer segment.

However, casual riders still represent a significant user group, meaning there is strong potential to convert them into members.

### 4.3. Average ride duration ###

The average ride duration was calculated for each rider type.

```sql
SELECT
  member_casual,
  AVG(ride_length_minutes) AS avg_ride_length
FROM `cyclistic-bike-share-heinnurm.bike_share_data.cleaned_trips`
GROUP BY member_casual
```

| User type | Average ride length (minutes) |
|-----------|-------------:|
| Casual riders | 22.14 min |
| Annual members | 11.92 min |

Why this is important? This reveals a clear behavioral difference:
- Casual riders take significantly longer rides
- Members take shorter rides

This suggests:
- casual riders likely use bikes for leisure activities
- members likely use bikes for commuting or short trips

### 4.4. Ride duration distribution ###

Ride duration extremes were also analyzed.

```sql
SELECT
  member_casual,
  AVG(ride_length_minutes) AS avg_ride,
  MAX(ride_length_minutes) AS max_ride,
  MIN(ride_length_minutes) AS min_ride
FROM `cyclistic-bike-share-heinnurm.bike_share_data.cleaned_trips`
GROUP BY member_casual
```

| Metric | Members | Casual | 
|-----------|-----------|-------------:|
| Maximum ride length | 1499 min | 1574 min |
| Maximum ride length | 0 min | 0 min |

These values help identify:
- extreme ride durations
- potential data anomalies or system limits

The similar maximum values suggest that both rider types occasionally take very long rides, but this behavior is not typical for most users.

### 4.5. Rides by weekday ###

Ride frequency was analyzed for each day of the week.

```sql
SELECT
  member_casual,
  day_of_week,
  COUNT(*) AS rides
FROM `cyclistic-bike-share-heinnurm.bike_share_data.cleaned_trips`
GROUP BY member_casual, day_of_week
ORDER BY day_of_week
```

Example values:

| Day      | Members | Casual  |
| -------- | ------- | ------- |
| Monday   | 504,352 | 228,824 |
| Thursday | 571,181 | 257,285 |
| Saturday | 449,038 | 413,955 |
| Sunday   | 382,132 | 332,460 |


Key observations:
- Members ride most frequently on weekdays, especially Tuesday, Wednesday, and Thursday
- Casual riders ride most on weekends, particularly Saturday and Sunday

### 4.6. Average ride length by weekday ###

| Day of week | Member avg ride (min) | Casual avg ride (min) |
| ----------- | --------------------- | --------------------- |
| Monday      | ~12                   | ~19                   |
| Wednesday   | ~12                   | ~19                   |
| Friday      | ~13                   | ~22                   |
| Saturday    | ~15                   | ~28                   |
| Sunday      | ~16                   | ~30                   |

```sql
SELECT
  member_casual,
  day_of_week,
  AVG(ride_length_minutes) AS avg_ride
FROM `cyclistic-bike-share-heinnurm.bike_share_data.cleaned_trips`
GROUP BY member_casual, day_of_week
ORDER BY day_of_week
```

Data shows that: 
- Casual riders consistently have longer ride durations than members.
- The difference becomes especially noticeable during weekends.
- Weekend rides by casual riders can be almost twice as long as those by members.

Cyclistic members seem to rely on bikes as a transportation tool, while casual riders treat them more like a recreational service.

### 4.7. Rides by month ###

| Month    | Member rides | Casual rides |
| -------- | ------------ | ------------ |
| January  | Low          | Very low     |
| April    | Increasing   | Increasing   |
| June     | High         | Very high    |
| July     | High         | Peak         |
| August   | High         | Peak         |
| December | Moderate     | Very low     |

```sql
SELECT
  member_casual,
  month,
  COUNT(*) AS rides
FROM `cyclistic-bike-share-heinnurm.bike_share_data.cleaned_trips`
GROUP BY member_casual, month
ORDER BY month
```

Data shows that: 
- Both rider types increase during spring and summer.
- Casual riders show much stronger seasonality.
- Casual rides peak during summer months (June–August).
- Members maintain relatively consistent activity throughout the year.

Casual riders are primarily seasonal leisure users, while members are regular transportation users.

### 4.8. Bikes type usage ###

| Rider type | Classic bike | Electric bike | Docked bike      |
| ---------- | ------------ | ------------- | ---------------- |
| Member     | High usage   | High usage    | Very low         |
| Casual     | High usage   | Medium usage  | Noticeable usage |

```sql
SELECT
  member_casual,
  rideable_type,
  COUNT(*) AS rides
FROM `cyclistic-bike-share-heinnurm.bike_share_data.cleaned_trips`
GROUP BY member_casual, rideable_type
```

Data shows that: 
- Both rider groups frequently use classic bikes.
- Members show strong adoption of electric bikes, likely for commuting efficiency.
- Casual riders use docked bikes more often.

Casual riders appear to prioritize accessibility and flexibility, while members prioritize efficiency and reliability.

### 4.9. Summary ###

The following table summarizes the number of rides and average ride duration by weekday for casual riders and annual members.

```sql
CREATE OR REPLACE TABLE `cyclistic-bike-share-heinnurm.bike_share_data.analysis_summary` AS
SELECT
  member_casual,
  day_of_week,
  COUNT(*) AS rides,
  AVG(ride_length_minutes) AS avg_ride
FROM `cyclistic-bike-share-heinnurm.bike_share_data.cleaned_trips`
GROUP BY member_casual, day_of_week
```

| Day of Week | Casual Rides | Avg Casual Ride (min) | Member Rides | Avg Member Ride (min) |
| ----------- | ------------ | --------------------- | ------------ | --------------------- |
| Monday      | 228,824      | 21.84                 | 504,352      | 11.51                 |
| Tuesday     | 227,079      | 19.36                 | 569,167      | 11.51                 |
| Wednesday   | 220,516      | 18.23                 | 548,658      | 11.38                 |
| Thursday    | 257,285      | 19.40                 | 571,181      | 11.48                 |
| Friday      | 319,965      | 22.07                 | 527,451      | 11.91                 |
| Saturday    | 413,955      | 24.82                 | 449,038      | 13.08                 |
| Sunday      | 332,460      | 25.67                 | 382,132      | 13.17                 |

Key Insights: 
- The analysis shows clear behavioral differences between casual riders and annual members.
- Casual riders take longer trips. Their average ride duration ranges from about 18–26 minutes, while member rides average around 11–13 minutes.
- Members ride more frequently during weekdays, suggesting that they use Cyclistic bikes mainly for commuting or daily transportation.
- Casual riders are more active during weekends, with the highest number of rides occurring on Saturday and Sunday.
- Weekend rides by casual riders are also longer in duration, indicating that these trips are likely recreational or leisure-oriented.
- Overall, the data suggests that members primarily use bikes for routine transportation, while casual riders tend to use them for leisure activities, especially on weekends.

## 5. VISUALIZE ##

## Cyclistic Bike Usage by Day of Week

![Cyclistic Bike Usage](bike_usage_day_of_week.png)
