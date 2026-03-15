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

### STEP 4. IDENTIFY MISSING VALUES###

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
