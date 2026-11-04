# NYC-Traffic-Accidents-Analysis-
# NYC Traffic Accidents Data Analysis

## Project Overview
This project analyzes over 74,000 New York City traffic accident records to uncover spatial and temporal trends in collision frequency and severity. The goal of this analysis is to identify peak accident times, evaluate borough-specific trends, and clean a raw, real-world dataset for structured querying.

## Dataset
* **Timeframe:** January 2020 – August 2020
* **Volume:** 74,881 records
* **Key Features:** Crash Date/Time, Borough, Coordinates, Injury/Fatality Counts, and Contributing Factors.

## Tools & Technologies
* **SQL Dialect:** SQLite
* **Database Management:** DB Browser for SQLite
* **Concepts Applied:** Data Cleaning, Aggregations (`GROUP BY`, `SUM`, `COUNT`), `CASE` statements, Data Type Conversion, Null Handling (`COALESCE`), and Datetime Formatting (`strftime`).

## Key Business Questions
1. **Temporal Trends:** What is the percentage of total accidents by month?
2. **High-Risk Windows:** How does accident frequency break down by day of the week and hour of the day?
3. **Peak Danger Zones:** Based on historical data, when do accidents occur most frequently?

## Data Cleaning & Transformation
Before analyzing the data, several cleaning steps were required to ensure accurate querying:
* **Handling Missing Data:** Utilized the `COALESCE()` function to replace missing borough locations with an 'UNKNOWN' label, ensuring aggregate counts remained accurate without dropping valuable rows.
* **Data Type Integrity:** Managed coordinate fields (Latitude/Longitude) as text strings to prevent float truncation, allowing for cleaner geographic mapping.
* **Standardizing Timeframes:** Used SQLite's `strftime` and `CASE` statements to convert raw numerical days into readable text formats (e.g., 'Monday', 'Tuesday') for reporting.

## Key Insights
* **Most Dangerous Hour:** Accidents occur most frequently on Fridays at 4:00 PM (16:00), accounting for the highest single-hour volume in the dataset.
* **Peak Commute Danger:** The surrounding Friday afternoon hours (2:00 PM, 5:00 PM, and 6:00 PM) consistently rank as the highest-risk windows for collisions.
* **Monthly Distribution:** January recorded the highest volume of accidents (19.1% of the total dataset), closely followed by February (18.3%), indicating a higher collision frequency during winter months.

## SQL Queries

### 1. Percentage of Total Accidents by Month
```sql
SELECT 
    strftime('%m', "CRASH DATE") AS Month,
    COUNT(*) AS Accident_Count,
    ROUND((COUNT(*) * 100.0) / (SELECT COUNT(*) FROM NYC_Accidents), 2) AS Percentage_of_Total
FROM NYC_Accidents
WHERE "CRASH DATE" IS NOT NULL
GROUP BY Month
ORDER BY Month;
```

### 1. Monthly Percentage Analysis
This query calculates the proportion of total accidents that occurred in each month to identify seasonal trends. 
* **Dynamic Subqueries:** Instead of hardcoding the total number of rows as a denominator, a subquery `(SELECT COUNT(*) FROM NYC_Accidents)` is used to calculate the total dynamically. This ensures the query remains accurate and scalable even if new data is appended to the database later.
* **Formatting:** The `ROUND()` function is applied to keep the output clean and readable at two decimal places for reporting purposes.

```sql
SELECT 
    CASE cast(strftime('%w', "CRASH DATE") AS integer)
        WHEN 0 THEN 'Sunday'
        WHEN 1 THEN 'Monday'
        WHEN 2 THEN 'Tuesday'
        WHEN 3 THEN 'Wednesday'
        WHEN 4 THEN 'Thursday'
        WHEN 5 THEN 'Friday'
        WHEN 6 THEN 'Saturday'
    END AS Day_Of_Week,
    strftime('%H', "CRASH TIME") AS Hour_Of_Day,
    COUNT(*) AS Total_Accidents
FROM NYC_Accidents
WHERE "CRASH DATE" IS NOT NULL AND "CRASH TIME" IS NOT NULL
GROUP BY Day_Of_Week, Hour_Of_Day
ORDER BY Total_Accidents DESC;

### 2. Peak Danger Times (Day & Hour)
This query identifies the specific times when accidents are most likely to occur by breaking down the timestamp into granular components.
* **Data Transformation:** SQLite's `strftime('%w')` extracts the day of the week as an integer (0-6). A `CASE` statement is utilized to transform these integers into readable string text (Sunday - Saturday), ensuring the output is immediately ready for stakeholder presentations.
* **Multi-Level Aggregation:** The dataset is aggregated by both the day of the week and the hour of the day using `GROUP BY`, then sorted in descending order using `ORDER BY` to instantly surface the highest-risk windows at the top of the result set.
```
