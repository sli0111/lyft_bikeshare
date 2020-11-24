## Increasing Ridership for Lyft Bikeshare using Big Query

June 2020

### Table of Contents

- Files
- Data Source
- Sample BigQuery through GCP, CLI, and Jupyter Notebook
- Report in Jupyter Notebook
- References
- Bonus

### Introduction
In the this project, BigQuery (BQ) is used to explore the Bay Area Bikeshare dataset from Lyft to provide insights on the top 5 "commuter trips" and make recommendations to increase bikeshare membership.  The BigQueries were performed using the Google Cloud Platform (GCP) through the UI and in Jupyter Notebook.  This README contains samples of the queries used for the insights and recommendations.  The complete report is in Jupyter Notebook which contexualizes the results into one place. The Jupyter Notedbook will contain data tables in the form of Pandas dataframes and data visualizations using Seaborn / Matplotlib.  


### Files 
The following two files will be used for Jupyter Notebook report.  
  * [Jupyter Notebook Project-1](https://github.com/mids-w205-de-sola/project-1-sli0111/blob/assignment/Project_1.ipynb)
  * [df_sf_map.jpeg](https://github.com/mids-w205-de-sola/project-1-sli0111/blob/assignment/df_sf_map.jpg)

### Data Source
The Bay Bike Share dataset is located on GCP and the tables used in this project are.
  * bigquery-public-data.san_francisco.bikeshare_stations
  * bigquery-public-data.san_francisco.bikeshare_status
  * bigquery-public-data.san_francisco.bikeshare_trips

The dataset can be queried using the following scripts from the UI on GCP or Jupyter Notebook.

* Using BQ from GCP has the general query structure:
```
SELECT count(*)
FROM
   `bigquery-public-data.san_francisco.bikeshare_trips`'
```

* Using BQ in Jupyter Notebook has the general query structure:
```
%%bigquery dataframe_name

SELECT count(*)
FROM
   `bigquery-public-data.san_francisco.bikeshare_trips`'
```

### Sample Queries

The following three parts contain sample queries used to understand specific question in this analysis.  Queries listed in Part 1 are for use in UI and Part 2 for use in Jupyter Notebook.  Part 1 covers general queries for basic understanding of the dataset and Part 2 asks deeper questions.

### Part 1 - Data Query with BigQuery

The queries below are samples for use in GCP BigQuery UI.

- Question 1: What's the size of this dataset? (i.e., how many trips)
  * Answer: 983,649 unique trips
```sql
SELECT COUNT(DISTINCT trip_id)
FROM `bigquery-public-data.san_francisco.bikeshare_trips`;
```
| Row | f0_     |   
|-----|---------|
| 1   | 983648  | 

- Question 2: What is the earliest start date and time and latest end date and time for a trip?
  * Answer: Dates range from 2013 to 2016
```sql
SELECT MIN(DISTINCT start_date), MAX(DISTINCT end_date)
FROM `bigquery-public-data.san_francisco.bikeshare_trips`;
```
| Row | f0_                     | f1_                     |   
|-----|-------------------------|-------------------------|
| 1   | 2013-08-29 09:08:00 UTC | 2016-08-31 23:48:00 UTC |  
 
- Question 3: How many bikes are there?
  * Answer: 700
```sql
SELECT COUNT(DISTINCT bike_number)
FROM `bigquery-public-data.san_francisco.bikeshare_trips`;
```
| Row | f0_  |   
|-----|------|
| 1   | 700  | 

- Question 4: How many trips are in the morning vs in the afternoon?
  * Answer: Given the morning is from 4am-12pm and afternoon is 12pm-4pm, then morning = 453,267 trips and afternoon = 264,897 trips.
```sql
  bq query --use_legacy_sql=false '
      SELECT count(trip_id)
      FROM 'bigquery-public-data.san_francisco.bikeshare_trips`'
      WHERE extract(hour from start_date) between 4 and 12 ;
  ```
|  f0_   |
|--------|
| 453267 |


```sql
 bq query --use_legacy_sql=false '
      SELECT count(trip_id)
      FROM 'bigquery-public-data.san_francisco.bikeshare_trips`'
      WHERE extract(hour from start_date) between 12 and 16 ;
```
|  f0_   |
|--------|
| 264897 |

- Question 4: What is the percentage of trips for each month?
  * Answer:  December has the least trips and August has the most trips at 9.7%.
```sql
SELECT 
    EXTRACT(month from start_date) as months,
    COUNT(trip_id) as trips_started,
    ROUND(COUNT(trip_id) / 983648 *100, 2) as percent_of_total_rides
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
GROUP BY months
ORDER BY percent_of_total_rides DESC;
```
| Row | months | trips_started | percent_of_total_rides |
|-----|--------|---------------|------------------------|
| 1   |      8 |         95576 |                   9.72 |
| 2   |     10 |         94378 |                   9.59 |
| 3   |      6 |         91672 |                   9.32 |
| 4   |      7 |         89539 |                    9.1 |
| 5   |      9 |         87321 |                   8.88 |
| 6   |      5 |         86364 |                   8.78 |
| 7   |      4 |         84196 |                   8.56 |
| 8   |      3 |         81777 |                   8.31 |
| 9   |     11 |         73091 |                   7.43 |
| 10  |      1 |         71788 |                    7.3 |
| 11  |      2 |         69985 |                   7.11 |
| 12  |     12 |         57961 |                   5.89 |

- Question 5: What is the percentage of trips for each day of a week?
  * Answer:  Sunday has the least trips and Tuesday has the most trips at 18.8%.
```sql
SELECT 
    EXTRACT(dayofweek FROM start_date) as days,
    COUNT(trip_id) as trips_started,
    ROUND(COUNT(trip_id) / 983648 *100, 2) as percent_of_total_rides
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
GROUP BY days
ORDER BY percent_of_total_rides DESC;
```
| Row | days | trips_started | percent_of_total_rides |
|-----|------|---------------|------------------------|
| 1   |    3 |        184405 |                  18.75 |
| 2   |    4 |        180767 |                  18.38 |
| 3   |    5 |        176908 |                  17.98 |
| 4   |    2 |        169937 |                  17.28 |
| 5   |    6 |        159977 |                  16.26 |
| 6   |    7 |         60279 |                   6.13 |
| 7   |    1 |         51375 |                   5.22 |

- Question 6: What is the percentage of trips for each hour of a day?
  * Answer: 8am has the most trips at 13.5%.
```sql
SELECT 
    EXTRACT(hour FROM start_date) as hours,
    COUNT(trip_id) as trips_started,
    ROUND(COUNT(trip_id) / 983648 *100, 2) as percent_of_total_rides
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
GROUP BY hours
ORDER BY percent_of_total_rides DESC;
```
| Row | hours | trips_started | percent_of_total_rides |
|-----|-------|---------------|------------------------|
| 1   |     8 |        132464 |                  13.47 |
| 2   |    17 |        126302 |                  12.84 |
| 3   |     9 |         96118 |                   9.77 |
| 4   |    16 |         88755 |                   9.02 |
| 5   |    18 |         84569 |                    8.6 |
| 6   |     7 |         67531 |                   6.87 |
| 7   |    15 |         47626 |                   4.84 |
| 8   |    12 |         46950 |                   4.77 |
| 9   |    13 |         43714 |                   4.44 |
| 10  |    10 |         42782 |                   4.35 |
| 11  |    19 |         41071 |                   4.18 |
| 12  |    11 |         40407 |                   4.11 |
| 13  |    14 |         37852 |                   3.85 |
| 14  |    20 |         22747 |                   2.31 |
| 15  |     6 |         20519 |                   2.09 |
| 16  |    21 |         15258 |                   1.55 |
| 17  |    22 |         10270 |                   1.04 |
| 18  |    23 |          6195 |                   0.63 |
| 19  |     5 |          5098 |                   0.52 |
| 20  |     0 |          2929 |                    0.3 |
| 21  |     1 |          1611 |                   0.16 |
| 22  |     4 |          1398 |                   0.14 |
| 23  |     2 |           877 |                   0.09 |
| 24  |     3 |           605 |                   0.06 |


### Part 2 - Data query with Jupyter Notebook

The queries below are samples for use in Jupyter Notebook with Python 3 kernel.  The queries were also used for recommendations for bikeshare memberships in the report.

- Question 1: Are there still options in coverting customers to subscribers during commuting hours?
  * Answer: For the most part, the most popular stations have >95% subscriber membership. However, we can focus some efforts to improve membership at the Ferry Building and the Embarcadero at Sansome which have only about 90% subscriber membership.

```python
%%bigquery df_commuting_subscriber

SELECT 
  start_station_name, 
  count(*) as Total_Trips,
  sum(case when subscriber_type = 'Subscriber' then 1 else 0 end) as Subscriber,
  sum(case when subscriber_type = 'Customer' then 1 else 0 end) as Customer
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE 
    extract(hour from start_date) in (7, 8, 9, 16, 17, 18) and
    extract(dayofweek from start_date) in (2, 3, 4, 5, 6)
GROUP BY start_station_name 
ORDER BY Total_Trips DESC LIMIT 10
;
```
|    | start_station_name                            |   Total_Trips |   Subscriber |   Customer |
|---:|:----------------------------------------------|--------------:|-------------:|-----------:|
|  0 | San Francisco Caltrain (Townsend at 4th)      |         52441 |        50832 |       1609 |
|  1 | San Francisco Caltrain 2 (330 Townsend)       |         40264 |        39319 |        945 |
|  2 | Temporary Transbay Terminal (Howard at Beale) |         28880 |        28448 |        432 |
|  3 | Harry Bridges Plaza (Ferry Building)          |         27298 |        25002 |       2296 |
|  4 | Steuart at Market                             |         25313 |        24173 |       1140 |
|  5 | 2nd at Townsend                               |         24103 |        23183 |        920 |
|  6 | Townsend at 7th                               |         20323 |        19669 |        654 |
|  7 | Market at Sansome                             |         18880 |        18037 |        843 |
|  8 | Embarcadero at Sansome                        |         18587 |        15782 |       2805 |
|  9 | Market at 10th                                |         17273 |        16457 |        816 |

- Question 2: Which stations have the least amount of start trips in San Francisco?
  * Answer: These areas are in the Tenderloin and between Union Square and Chinatown.   

```python
%%bigquery df_least

SELECT
    start_station_id,
    landmark,
    start_station_name,
    count(trip_id) as trips_started,
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
INNER JOIN `bigquery-public-data.san_francisco.bikeshare_stations`
ON start_station_id = station_id
WHERE landmark = 'San Francisco'
GROUP BY start_station_id, start_station_name, landmark
ORDER BY trips_started ASC LIMIT 10
;
```
|    |   start_station_id | landmark      | start_station_name                            |   trips_started |
|---:|-------------------:|:--------------|:----------------------------------------------|----------------:|
|  0 |                 91 | San Francisco | Cyril Magnin St at Ellis St                   |              69 |
|  1 |                 90 | San Francisco | 5th St at Folsom St                           |             173 |
|  2 |                 46 | San Francisco | Washington at Kearney                         |            1472 |
|  3 |                 47 | San Francisco | Post at Kearney                               |            2503 |
|  4 |                 58 | San Francisco | San Francisco City Hall                       |            6730 |
|  5 |                 46 | San Francisco | Washington at Kearny                          |            7136 |
|  6 |                 59 | San Francisco | Golden Gate at Polk                           |           10651 |
|  7 |                 47 | San Francisco | Post at Kearny                                |           11308 |
|  8 |                 41 | San Francisco | Clay at Battery                               |           14351 |
|  9 |                 68 | San Francisco | Yerba Buena Center of the Arts (3rd @ Howard) |           14711 |

- Question 3: How is the ratio of customers to subscribers changing throughout the year?
  * Answer: The % of customer actually reaches a peak in August. 

```python
%%bigquery df_month_winter

SELECT 
    EXTRACT(month from start_date) as months,
    COUNT(trip_id) as trips_started,
    ROUND(COUNT(trip_id) / 983648 *100, 2) as percent_of_total_rides,
    sum(case when subscriber_type = 'Subscriber' then 1 else 0 end) as Subscriber,
    sum(case when subscriber_type = 'Customer' then 1 else 0 end) as Customer
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
GROUP BY months
ORDER BY percent_of_total_rides DESC;
```
|    |   months |   trips_started |   percent_of_total_rides |   Subscriber |   Customer |
|---:|---------:|----------------:|-------------------------:|-------------:|-----------:|
|  0 |        8 |           95576 |                     9.72 |        80033 |      15543 |
|  1 |       10 |           94378 |                     9.59 |        80340 |      14038 |
|  2 |        6 |           91672 |                     9.32 |        79525 |      12147 |
|  3 |        7 |           89539 |                     9.1  |        76387 |      13152 |
|  4 |        9 |           87321 |                     8.88 |        70011 |      17310 |
|  5 |        5 |           86364 |                     8.78 |        73623 |      12741 |
|  6 |        4 |           84196 |                     8.56 |        74218 |       9978 |
|  7 |        3 |           81777 |                     8.31 |        71619 |      10158 |
|  8 |       11 |           73091 |                     7.43 |        63720 |       9371 |
|  9 |        1 |           71788 |                     7.3  |        64075 |       7713 |
| 10 |        2 |           69985 |                     7.11 |        62123 |       7862 |
| 11 |       12 |           57961 |                     5.89 |        51165 |       6796 |

- Question 4: How long are customers riding?
  * Answer: The top 10 popular stations for customers are in San Francisco and they take rides that are less than 25 minutes long. 
  
```python
%%bigquery df_stations_customers

SELECT
    start_station_name,
    count(trip_id) as trips_started,
    ROUND(AVG(duration_sec/60),2) as average_trip_length_minutes,
    sum(case when subscriber_type = 'Customer' then 1 else 0 end) as Customer
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
GROUP BY start_station_name
ORDER BY Customer DESC LIMIT 10
;
```
|    | start_station_name                       |   trips_started |   average_trip_length_minutes |   Customer |
|---:|:-----------------------------------------|----------------:|------------------------------:|-----------:|
|  0 | Embarcadero at Sansome                   |           41137 |                         22.13 |      13934 |
|  1 | Harry Bridges Plaza (Ferry Building)     |           49062 |                         22.74 |      12441 |
|  2 | Market at 4th                            |           27502 |                         20.65 |       5952 |
|  3 | Powell Street BART                       |           25204 |                         22.02 |       5214 |
|  4 | Embarcadero at Vallejo                   |           15302 |                         24.86 |       4945 |
|  5 | Powell at Post (Union Square)            |           16984 |                         25.11 |       4932 |
|  6 | Steuart at Market                        |           38531 |                         14.35 |       4469 |
|  7 | 2nd at Townsend                          |           39936 |                         12.48 |       4436 |
|  8 | San Francisco Caltrain (Townsend at 4th) |           72683 |                         13.41 |       4299 |
|  9 | Market at Sansome                        |           35142 |                         14.3  |       3874 |

### Jupyter Notebook Report: Project-1
Project-1 is a Jupyter Notebook that provides a report on the findings in this project.  To run the notebook with BigQuery, one needs to install the [Python client for Google BigQuery](https://googleapis.dev/python/bigquery/latest/index.html) or use the Jupyter Notebook on the Google Cloud VM.  

There are 7 sections in the notebook:
* 0. Import necessary libraries: Pandas, Matplotlib, Seaborn
* 1. Initial Data Exploration
* 2. Bikeshare trips vs Time
* 3. Bikeshare trips vs Location
* 4. Bikeshare trips vs Length of Ride
* 5. Top 5 Commuter Trips
* 6. Recommendation for New Memberships
* 7. Conclusion

To run the notebook properly, Section 0. must be run first to load the libraries.  Then subsqeuent sections need to be executed in the following cell block order.  The BG magic commands will generate a Pandas dataframe which will be used to display a table and the visualizations.
1. Bigquery magic command with sql script
2. Pandas DataFrame table with edits
3. Visualizations with Matplotlib and Seaborn

### Resources:
* SQL tutorial: https://www.w3schools.com/sql/default.asp
* Read: https://cloud.google.com/docs/overview/
* BigQuery: https://cloud.google.com/bigquery/
* Lyft Bay Wheels: https://www.lyft.com/bikes/bay-wheels
* Python client for Google BigQuery: https://googleapis.dev/python/bigquery/latest/index.html

### Bonus:
The following are bonus queries using the dynamic datasets:
* bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info
* bigquery-public-data.san_francisco_bikeshare.bikeshare_regions
* bigquery-public-data.san_francisco_bikeshare.bikeshare_trips

Question 1: Top 5 popular station pairs in each region
```sql
(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    start_station_name,
    end_station_name,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id = 3
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, start_station_name, end_station_name
ORDER BY total_trips DESC LIMIT 5)

UNION ALL

(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    start_station_name,
    end_station_name,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id = 5
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, start_station_name, end_station_name
ORDER BY total_trips DESC LIMIT 5)

UNION ALL

(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    start_station_name,
    end_station_name,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id = 12
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, start_station_name, end_station_name
ORDER BY total_trips DESC LIMIT 5)

UNION ALL

(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    start_station_name,
    end_station_name,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id = 13
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, start_station_name, end_station_name
ORDER BY total_trips DESC LIMIT 5)

UNION ALL

(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    start_station_name,
    end_station_name,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id = 14
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, start_station_name, end_station_name
ORDER BY total_trips DESC LIMIT 5)
```
| Row | name          | start_station_name                   | end_station_name                         | total_trips |
|-----|---------------|--------------------------------------|------------------------------------------|-------------|
| 1   | San Jose      | University and Emerson               | University and Emerson                   |        1184 |
| 2   | San Jose      | 5th St at Virginia St                | San Fernando at 7th St                   |         884 |
| 3   | San Jose      | San Fernando at 7th St               | 5th St at Virginia St                    |         630 |
| 4   | San Jose      | San Jose Diridon Station             | San Fernando St at 4th St                |         477 |
| 5   | San Jose      | San Fernando St at 4th St            | Ryland Park                              |         445 |
| 6   | Oakland       | 19th Street BART Station             | Grand Ave at Perkins St                  |        2372 |
| 7   | Oakland       | Grand Ave at Perkins St              | 19th Street BART Station                 |        2073 |
| 8   | Oakland       | Bay Pl at Vernon St                  | 19th Street BART Station                 |        1968 |
| 9   | Oakland       | 19th Street BART Station             | Bay Pl at Vernon St                      |        1536 |
| 10  | Oakland       | Lake Merritt BART Station            | 2nd Ave at E 18th St                     |        1179 |
| 11  | Berkeley      | Bancroft Way at College Ave          | Downtown Berkeley BART                   |        1341 |
| 12  | Berkeley      | Bancroft Way at College Ave          | Bancroft Way at Telegraph Ave            |        1083 |
| 13  | Berkeley      | Bancroft Way at Telegraph Ave        | Downtown Berkeley BART                   |         913 |
| 14  | Berkeley      | Bancroft Way at College Ave          | Fulton St at Bancroft Way                |         591 |
| 15  | Berkeley      | Downtown Berkeley BART               | Bancroft Way at College Ave              |         557 |
| 16  | Emeryville    | Adeline St at 40th St                | MacArthur BART Station                   |         314 |
| 17  | Emeryville    | Horton St at 40th St                 | Horton St at 40th St                     |         279 |
| 18  | Emeryville    | 65th St at Hollis St                 | 65th St at Hollis St                     |         174 |
| 19  | Emeryville    | Emeryville Town Hall                 | 53rd St at Hollis St                     |         121 |
| 20  | Emeryville    | Doyle St at 59th St                  | MacArthur BART Station                   |         113 |
| 21  | San Francisco | Harry Bridges Plaza (Ferry Building) | Embarcadero at Sansome                   |        9150 |
| 22  | San Francisco | 2nd at Townsend                      | Harry Bridges Plaza (Ferry Building)     |        7620 |
| 23  | San Francisco | Harry Bridges Plaza (Ferry Building) | 2nd at Townsend                          |        6888 |
| 24  | San Francisco | Embarcadero at Sansome               | Steuart at Market                        |        6874 |
| 25  | San Francisco | Embarcadero at Folsom                | San Francisco Caltrain (Townsend at 4th) |        6351 |

Question 2: Top 3 most popular regions(stations belong within 1 region)
```sql
SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name
ORDER BY total_trips DESC LIMIT 3
```
| Row | name          | total_trips |
|-----|---------------|-------------|
| 1   | San Francisco |     1450749 |
| 2   | Oakland       |      144522 |
| 3   | San Jose      |       43173 |

Total trips for each short station name in each region

```sql
SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name
ORDER BY total_trips DESC LIMIT 3
;

(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    short_name,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = 3
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, short_name
ORDER BY total_trips DESC) 

UNION ALL

(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    short_name,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = 5
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, short_name
ORDER BY total_trips DESC) 

UNION ALL

(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    short_name,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = 12
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, short_name
ORDER BY total_trips DESC) 

UNION ALL

(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    short_name,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = 13
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, short_name
ORDER BY total_trips DESC) 

UNION ALL

(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    short_name,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = 14
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, short_name
ORDER BY total_trips DESC) 
;
```

| Row | name     | short_name | total_trips |   |
|-----|----------|------------|-------------|---|
| 1   | Oakland  | OK-F4      |        8166 |   |
| 2   | Oakland  | OK-H9-1    |        2314 |   |
| 3   | Oakland  | OK-J5      |        3145 |   |
| 4   | Oakland  | OK-M7      |        5695 |   |
| 5   | Oakland  | OK-I9      |        6038 |   |
| 6   | Oakland  | OK-E5      |        1769 |   |
| 7   | Oakland  | OK-I6      |        5499 |   |
| 8   | Oakland  | OK-B3      |         507 |   |
| 9   | Oakland  | OK-C4      |         732 |   |
| 10  | Oakland  | OK-M1      |        3407 |   |
| 11  | Oakland  | OK-H4-3    |         691 |   |
| 12  | Oakland  | OK-I8      |        5513 |   |
| 13  | Oakland  | OK-L2      |        1431 |   |
| 14  | Oakland  | OK-J6-1    |        4882 |   |
| 15  | Oakland  | OK-D5      |         758 |   |
| 16  | Oakland  | OK-K2      |        1025 |   |
| 17  | Oakland  | OK-L7-2    |        3488 |   |
| 18  | Oakland  | OK-L8      |        2433 |   |
| 19  | Oakland  | OK-L5      |        8293 |   |
| 20  | Oakland  | OK-M5      |        1535 |   |
| 21  | Oakland  | OK-K9      |        5245 |   |
| 22  | Oakland  | OK-L10     |         421 |   |
| 23  | Oakland  | OK-L12     |        9132 |   |
| 24  | Oakland  | OK-I5      |        2066 |   |
| 25  | Oakland  | OK-G5      |        2076 |   |
| 26  | Oakland  | OK-J3      |        1023 |   |
| 27  | Oakland  | OK-B5      |        1483 |   |
| 28  | Oakland  | OK-C3      |         843 |   |
| 29  | Oakland  | OK-D4      |        2078 |   |
| 30  | Oakland  | OK-M3      |         273 |   |
| 31  | Oakland  | OK-L11     |         362 |   |
| 32  | Oakland  | OK-D2      |        1980 |   |
| 33  | Oakland  | OK-A2      |        1155 |   |
| 34  | Oakland  | OK-L9      |        1803 |   |
| 35  | Oakland  | OK-A4      |        1332 |   |
| 36  | Oakland  | OK-H5      |        2262 |   |
| 37  | Oakland  | OK-K5-1    |        3553 |   |
| 38  | Oakland  | OK-K4      |         814 |   |
| 39  | Oakland  | OK-D3-1    |        1137 |   |
| 40  | Oakland  | OK-J6-2    |        1935 |   |
| 41  | Oakland  | OK-F5      |        2942 |   |
| 42  | Oakland  | OK-M16     |         111 |   |
| 43  | Oakland  | OK-H3      |        1247 |   |
| 44  | Oakland  | OK-L16     |         452 |   |
| 45  | Oakland  | OK-H9-2    |        2348 |   |
| 46  | Oakland  | OK-E3      |         476 |   |
| 47  | Oakland  | OK-E4      |        1668 |   |
| 48  | Oakland  | OK-D3-2    |         608 |   |
| 49  | Oakland  | OK-G3      |         579 |   |
| 50  | Oakland  | OK-C2      |        1175 |   |
| 51  | Oakland  | OK-M6      |        2381 |   |
| 52  | Oakland  | OK-H2      |         806 |   |
| 53  | Oakland  | OK-B2      |         634 |   |
| 54  | Oakland  | OK-K3      |         318 |   |
| 55  | Oakland  | OK-L4      |         849 |   |
| 56  | Oakland  | OK-M2      |         561 |   |
| 57  | Oakland  | OK-L19     |          74 |   |
| 58  | Oakland  | OK-G4      |         986 |   |
| 59  | Oakland  | OK-F3      |         731 |   |
| 60  | Oakland  | OK-L3      |         321 |   |
| 61  | Oakland  | OK-G2      |         587 |   |
| 62  | Oakland  | OK-J4      |         632 |   |
| 63  | Oakland  | OK-L13     |          95 |   |
| 64  | Oakland  | OK-L6-     |         743 |   |
| 65  | Oakland  | OK-L7      |         431 |   |
| 66  | Oakland  | OK-K6-     |         650 |   |
| 67  | Oakland  | OK-N17     |         272 |   |
| 68  | Oakland  | OK-B4      |        1564 |   |
| 69  | Oakland  | OK-N7      |         464 |   |
| 70  | Oakland  | OK-N6      |         264 |   |
| 71  | Oakland  | OK-L17     |          90 |   |
| 72  | Oakland  | OK-I3      |         234 |   |
| 73  | Oakland  | OK-C5      |         286 |   |
| 74  | Oakland  | OK-E2      |          93 |   |
| 75  | Oakland  | OK-B1      |         103 |   |
| 76  | Oakland  | OK-L14     |          72 |   |
| 77  | Oakland  | OK-I4      |          78 |   |
| 78  | Oakland  | OK-L15     |          95 |   |
| 79  | Oakland  | OK-K5-2    |       10208 |   |
| 80  | Berkeley | BK-E10     |        6765 |   |
| 81  | Berkeley | BK-E2      |         328 |   |
| 82  | Berkeley | BK-E8      |        1492 |   |
| 83  | Berkeley | BK-H10     |        1157 |   |
| 84  | Berkeley | BK-C6      |        1136 |   |
| 85  | Berkeley | BK-A7      |         827 |   |
| 86  | Berkeley | BK-C8      |         689 |   |
| 87  | Berkeley | BK-H9      |        2209 |   |
| 88  | Berkeley | BK-D7-1    |        3327 |   |
| 89  | Berkeley | BK-D7-2    |        1619 |   |
| 90  | Berkeley | BK-G7      |        1094 |   |
| 91  | Berkeley | BK-H6      |        3048 |   |
| 92  | Berkeley | BK-E9-2    |        2828 |   |
| 93  | Berkeley | BK-E9-1    |        3875 |   |
| 94  | Berkeley | BK-F2      |         285 |   |
| 95  | Berkeley | BK-D1      |         918 |   |
| 96  | Berkeley | BK-F8      |        1771 |   |
| 97  | Berkeley | BK-G9      |         589 |   |
| 98  | Berkeley | BK-D5      |         521 |   |
| 99  | Berkeley | BK-C5      |         860 |   |
| 100 | Berkeley | BK-C7      |        1216 |   |
| Row | name          | short_name | total_trips |   |
|-----|---------------|------------|-------------|---|
| 101 | Berkeley      | BK-B7      |         109 |   |
| 102 | Berkeley      | BK-H8      |         125 |   |
| 103 | Berkeley      | BK-E7      |         223 |   |
| 104 | Berkeley      | BK-H3      |         567 |   |
| 105 | Berkeley      | BK-F7      |         140 |   |
| 106 | Berkeley      | BK-G8      |         298 |   |
| 107 | Berkeley      | BK-B9      |         786 |   |
| 108 | Berkeley      | BK-A3      |         620 |   |
| 109 | Berkeley      | BK-F10     |         964 |   |
| 110 | Berkeley      | BK-I6      |         186 |   |
| 111 | Berkeley      | BK-G4      |          47 |   |
| 112 | San Jose      | SJ-L9      |        1261 |   |
| 113 | San Jose      | SJ-L10     |         395 |   |
| 114 | San Jose      | SJ-K11     |         609 |   |
| 115 | San Jose      | SJ-M8      |         752 |   |
| 116 | San Jose      | SJ-N10-1   |        1342 |   |
| 117 | San Jose      | SJ-O11     |         567 |   |
| 118 | San Jose      | SJ-P10     |        2448 |   |
| 119 | San Jose      | SJ-M10-2   |        2894 |   |
| 120 | San Jose      | SJ-M9-2    |        1377 |   |
| 121 | San Jose      | SJ-P8      |         283 |   |
| 122 | San Jose      | SJ-M11-2   |        2429 |   |
| 123 | San Jose      | SJ-L7-2    |        1155 |   |
| 124 | San Jose      | SJ-I10     |         266 |   |
| 125 | San Jose      | SJ-M10-3   |        1772 |   |
| 126 | San Jose      | SJ-N9      |         689 |   |
| 127 | San Jose      | SJ-N11     |        1698 |   |
| 128 | San Jose      | SJ-K5      |        1524 |   |
| 129 | San Jose      | SJ-M7-1    |        4040 |   |
| 130 | San Jose      | SJ-J8      |         672 |   |
| 131 | San Jose      | SJ-K6      |         682 |   |
| 132 | San Jose      | SJ-N10-2   |        1089 |   |
| 133 | San Jose      | SJ-M10-1   |        1817 |   |
| 134 | San Jose      | SJ-M11-1   |         755 |   |
| 135 | San Jose      | SJ-M8-2    |         213 |   |
| 136 | San Jose      | SJ-M11-3   |         793 |   |
| 137 | San Jose      | SJ-M6      |        3052 |   |
| 138 | San Jose      | SJ-M9-1    |         755 |   |
| 139 | San Jose      | SJ-K9      |        2434 |   |
| 140 | San Jose      | SJ-L7-1    |         605 |   |
| 141 | San Jose      | SJ-N10-3   |         725 |   |
| 142 | San Jose      | SJ-O9      |         363 |   |
| 143 | San Jose      | SJ-J10     |        1034 |   |
| 144 | San Jose      | SJ-N12-1   |         712 |   |
| 145 | San Jose      | SJ-I9      |         257 |   |
| 146 | San Jose      | SJ-Q8      |          41 |   |
| 147 | San Jose      | SJ-P9      |          27 |   |
| 148 | San Jose      | SJ-O10     |         461 |   |
| 149 | San Jose      | SJ-M10-4   |         642 |   |
| 150 | San Jose      | SJ-Q11     |          91 |   |
| 151 | San Jose      | SJ-H10     |         249 |   |
| 152 | San Jose      | SJ-J9      |         112 |   |
| 153 | San Jose      | SJ-H9      |          39 |   |
| 154 | San Jose      | SJ-Q9      |          52 |   |
| 155 | San Francisco | SF-O22     |        4930 |   |
| 156 | San Francisco | SF-Q22-1   |        4011 |   |
| 157 | San Francisco | SF-M22-2   |        6819 |   |
| 158 | San Francisco | SF-N22-1A  |        2426 |   |
| 159 | San Francisco | SF-I25     |        9570 |   |
| 160 | San Francisco | SF-O30     |        5078 |   |
| 161 | San Francisco | SF-N24-2   |        4016 |   |
| 162 | San Francisco | SF-C28-1   |        9252 |   |
| 163 | San Francisco | SF-L20     |        3742 |   |
| 164 | San Francisco | SF-T22     |        2794 |   |
| 165 | San Francisco | SF-S24     |        2803 |   |
| 166 | San Francisco | SF-R24     |        2342 |   |
| 167 | San Francisco | SF-I24     |        6264 |   |
| 168 | San Francisco | SF-O24     |        3283 |   |
| 169 | San Francisco | SF-N28     |        3061 |   |
| 170 | San Francisco | SF-N27     |        5182 |   |
| 171 | San Francisco | SF-M26     |        1718 |   |
| 172 | San Francisco | SF-F27     |        8414 |   |
| 173 | San Francisco | SF-K25     |        3382 |   |
| 174 | San Francisco | SF-P30     |        3111 |   |
| 175 | San Francisco | SF-H28     |        6517 |   |
| 176 | San Francisco | SF-S21     |         831 |   |
| 177 | San Francisco | SF-M24     |        3182 |   |
| 178 | San Francisco | SF-M22-1   |        4209 |   |
| 179 | San Francisco | SF-O18     |        1341 |   |
| 180 | San Francisco | SF-O19     |        3756 |   |
| 181 | San Francisco | SF-F26     |        9134 |   |
| 182 | San Francisco | SF-F28-1   |        7769 |   |
| 183 | San Francisco | SF-O25-1   |        2930 |   |
| 184 | San Francisco | SF-K28     |        5721 |   |
| 185 | San Francisco | SF-D28     |        8219 |   |
| 186 | San Francisco | SF-K27     |        5665 |   |
| 187 | San Francisco | SF-T21     |        1229 |   |
| 188 | San Francisco | SF-N22-2   |        5297 |   |
| 189 | San Francisco | SF-S22-1   |        3411 |   |
| 190 | San Francisco | SF-H24     |        4076 |   |
| 191 | San Francisco | SF-M25     |        2906 |   |
| 192 | San Francisco | SF-H18     |        4794 |   |
| 193 | San Francisco | SF-Q22-2   |        7391 |   |
| 194 | San Francisco | SF-R19     |        2715 |   |
| 195 | San Francisco | SF-P24     |        3836 |   |
| 196 | San Francisco | SF-D27     |        6728 |   |
| 197 | San Francisco | SF-H27-1   |        5193 |   |
| 198 | San Francisco | SF-H25     |        5658 |   |
| 199 | San Francisco | SF-C26     |        7075 |   |
| 200 | San Francisco | SF-H20     |        3228 |   |
| Row | name          | short_name | total_trips |   |
|-----|---------------|------------|-------------|---|
| 201 | San Francisco | SF-L30-1   |        7866 |   |
| 202 | San Francisco | SF-M21     |        4201 |   |
| 203 | San Francisco | SF-O23     |        2905 |   |
| 204 | San Francisco | SF-N21     |        2977 |   |
| 205 | San Francisco | SF-Q21-2   |        2668 |   |
| 206 | San Francisco | SF-O21     |        4319 |   |
| 207 | San Francisco | SF-L21     |        5688 |   |
| 208 | San Francisco | SF-T20     |        2664 |   |
| 209 | San Francisco | SF-E27     |        7959 |   |
| 210 | San Francisco | SF-E28     |        8863 |   |
| 211 | San Francisco | SF-M20     |        3290 |   |
| 212 | San Francisco | SF-G28-1   |        9734 |   |
| 213 | San Francisco | SF-N22-1B  |        3512 |   |
| 214 | San Francisco | SF-N29     |        3685 |   |
| 215 | San Francisco | SF-F30-2   |        8253 |   |
| 216 | San Francisco | SF-N20-2   |        1961 |   |
| 217 | San Francisco | SF-I27     |        5257 |   |
| 218 | San Francisco | SF-R20     |        1476 |   |
| 219 | San Francisco | SF-L26     |        4670 |   |
| 220 | San Francisco | SF-J18     |        3880 |   |
| 221 | San Francisco | SF-N23     |        1990 |   |
| 222 | San Francisco | SF-G19     |        5256 |   |
| 223 | San Francisco | SF-G29-2   |        3794 |   |
| 224 | San Francisco | SF-P22     |        4190 |   |
| 225 | San Francisco | SF-N20-1   |        2699 |   |
| 226 | San Francisco | SF-G26     |        9350 |   |
| 227 | San Francisco | SF-L19     |        6542 |   |
| 228 | San Francisco | SF-C28-2   |        9688 |   |
| 229 | San Francisco | SF-O25-2   |        3182 |   |
| 230 | San Francisco | SF-J17     |        2988 |   |
| 231 | San Francisco | SF-L25     |        4027 |   |
| 232 | San Francisco | SF-M27     |        1895 |   |
| 233 | San Francisco | SF-N25     |        2211 |   |
| 234 | San Francisco | SF I29-1   |        1159 |   |
| 235 | San Francisco | SF-N26     |         483 |   |
| 236 | San Francisco | SF-M28     |          97 |   |
| 237 | San Francisco | SF-D29     |        1533 |   |
| 238 | San Francisco | SF-H30     |        1026 |   |
| 239 | San Francisco | SF-Q21-1   |          47 |   |
| 240 | San Francisco | SF-L28     |          69 |   |
| 241 | San Francisco | SF-G27     |       19773 |   |
| 242 | San Francisco | SF-H26     |       16111 |   |
| 243 | San Francisco | SF-A27     |       26110 |   |
| 244 | San Francisco | SF-E29-1   |       25096 |   |
| 245 | San Francisco | SF-E29-2   |       18195 |   |
| 246 | San Francisco | SF-E29-3   |       13051 |   |
| 247 | San Francisco | SF-F28-2   |       19192 |   |
| 248 | San Francisco | SF-F29     |       19066 |   |
| 249 | San Francisco | SF-F30-1   |       10415 |   |
| 250 | San Francisco | SF-G30-1   |       12816 |   |
| 251 | San Francisco | SF-G30-2   |       17044 |   |
| 252 | San Francisco | SF-J29     |       24079 |   |
| 253 | San Francisco | SF-H29     |       10866 |   |
| 254 | San Francisco | SF-I19     |       27084 |   |
| 255 | San Francisco | SF-I23-1   |       21350 |   |
| 256 | San Francisco | SF-I23-2   |       19828 |   |
| 257 | San Francisco | SF-I28     |       19089 |   |
| 258 | San Francisco | SF-I29-2   |       23334 |   |
| 259 | San Francisco | SF-I30     |       56570 |   |
| 260 | San Francisco | SF-J15     |       21874 |   |
| 261 | San Francisco | SF-J19     |       22525 |   |
| 262 | San Francisco | SF-J20     |       41647 |   |
| 263 | San Francisco | SF-K21     |       26319 |   |
| 264 | San Francisco | SF-J23-1   |       27599 |   |
| 265 | San Francisco | SF-J23-2   |       18403 |   |
| 266 | San Francisco | SF-J24     |       46286 |   |
| 267 | San Francisco | SF-J25     |       45166 |   |
| 268 | San Francisco | SF-J26     |       27572 |   |
| 269 | San Francisco | SF-J27     |       22588 |   |
| 270 | San Francisco | SF-J28     |       30195 |   |
| 271 | San Francisco | SF-J29-1   |       30006 |   |
| 272 | San Francisco | SF-K17     |       80370 |   |
| 273 | San Francisco | SF-K18     |       19962 |   |
| 274 | San Francisco | SF-K19     |       23902 |   |
| 275 | San Francisco | SF-K20     |       23825 |   |
| 276 | San Francisco | SF-J21     |       43779 |   |
| 277 | San Francisco | SF-K22-1   |       23010 |   |
| 278 | San Francisco | SF-K22-2   |       33035 |   |
| 279 | San Francisco | SF-K24     |       39243 |   |
| 280 | San Francisco | SF-K29-1   |       19975 |   |
| 281 | San Francisco | SF-L27     |       11336 |   |
| 282 | Emeryville    | EM-B2      |         478 |   |
| 283 | Emeryville    | EM-A2      |         615 |   |
| 284 | Emeryville    | EM-D2      |        1326 |   |
| 285 | Emeryville    | EM-C1      |        1192 |   |
| 286 | Emeryville    | EM-D3      |         884 |   |
| 287 | Emeryville    | EM-C2      |         264 |   |
| 288 | Emeryville    | EM-A1      |         701 |   |
| 289 | Emeryville    | EM-C3      |         528 |   |
| 290 | Emeryville    | EM-D4      |        1021 |   |
| 291 | Emeryville    | EM-B1      |         204 |   |
| 192 | San Francisco | SF-H18     |        4794 |   |
| 193 | San Francisco | SF-Q22-2   |        7391 |   |
| 194 | San Francisco | SF-R19     |        2715 |   |
| 195 | San Francisco | SF-P24     |        3836 |   |
| 196 | San Francisco | SF-D27     |        6728 |   |
| 197 | San Francisco | SF-H27-1   |        5193 |   |
| 198 | San Francisco | SF-H25     |        5658 |   |
| 199 | San Francisco | SF-C26     |        7075 |   |
| 200 | San Francisco | SF-H20     |        3228 |   |

Question 4: What are the top 10 used bikes in each of the top 3 region. these bikes could be in need of more frequent maintenance?

```sql
(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    bike_number,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = 3
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, bike_number
ORDER BY total_trips DESC LIMIT 10)

UNION ALL

(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    bike_number,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = 5
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, bike_number
ORDER BY total_trips DESC LIMIT 10)

UNION ALL

(SELECT distinct 
    `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name,
    bike_number,
    count(*) as total_trips
FROM `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips`
ON station_id = start_station_id
JOIN `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`
ON `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.region_id
WHERE `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`.region_id = 12
GROUP BY `bigquery-public-data.san_francisco_bikeshare.bikeshare_regions`.name, bike_number
ORDER BY total_trips DESC LIMIT 10)
;
```
| Row | name          | bike_number | total_trips |
|-----|---------------|-------------|-------------|
| 1   | San Jose      |        1599 |         175 |
| 2   | San Jose      |         429 |         166 |
| 3   | San Jose      |        1872 |         165 |
| 4   | San Jose      |        1707 |         161 |
| 5   | San Jose      |        1754 |         160 |
| 6   | San Jose      |        1909 |         159 |
| 7   | San Jose      |        1831 |         155 |
| 8   | San Jose      |        1875 |         154 |
| 9   | San Jose      |         465 |         154 |
| 10  | San Jose      |        1784 |         153 |
| 11  | Oakland       |         570 |         258 |
| 12  | Oakland       |        1234 |         253 |
| 13  | Oakland       |         547 |         238 |
| 14  | Oakland       |         407 |         229 |
| 15  | Oakland       |         399 |         221 |
| 16  | Oakland       |         238 |         215 |
| 17  | Oakland       |         328 |         211 |
| 18  | Oakland       |         535 |         210 |
| 19  | Oakland       |        1393 |         210 |
| 20  | Oakland       |         152 |         208 |
| 21  | San Francisco |         395 |        2698 |
| 22  | San Francisco |         389 |        2695 |
| 23  | San Francisco |         625 |        2655 |
| 24  | San Francisco |         558 |        2651 |
| 25  | San Francisco |         540 |        2627 |
| 26  | San Francisco |         592 |        2612 |
| 27  | San Francisco |         534 |        2603 |
| 28  | San Francisco |         370 |        2594 |
| 29  | San Francisco |         421 |        2588 |
| 30  | San Francisco |         267 |        2569 |

