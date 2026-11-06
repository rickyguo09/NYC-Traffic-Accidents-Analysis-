# NYC Traffic Accidents Data Analysis

## Project Overview
This project analyzes over 74,000 New York City traffic accident records to uncover spatial and temporal trends in collision frequency and severity. The goal of this analysis is to identify peak accident times, evaluate borough-specific trends, and clean a raw, real-world dataset for structured querying.

## Dataset
* **Source:** [Kaggle - Global Traffic Accidents Dataset](https://www.kaggle.com/datasets/adilshamim8/global-traffic-accidents-dataset)
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

## SQL Queries & Analysis

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

<img width="283" height="230" alt="image" src="https://github.com/user-attachments/assets/0c057603-f50f-40a5-8b9e-a28c6e1c2823" />

**Query Methodology:**
* **Dynamic Subqueries:** Instead of hardcoding the total number of rows as a denominator, a subquery `(SELECT COUNT(*) FROM NYC_Accidents)` is used to calculate the total dynamically. This ensures the query remains accurate and scalable even if new data is appended to the database later.
* **Formatting:** The `ROUND()` function is applied to keep the output clean and readable at two decimal places for reporting purposes.

**Macro-Level Trends: Monthly Accident Volume**
The dataset (spanning January to August) demonstrates a distinct curve in accident frequency across the year:
* **Early Year Peak:** Accidents were highest at the start of the year. Month 01 (January) recorded the most accidents at 14,287, making up 19.08% of the total. Month 02 (February) closely followed with 13,684 accidents, or 18.27% of the total.
* **The Spring Drop:** There is a drastic reduction in accidents in Month 04 (April), plummeting to just 4,116 incidents, or 5.5% of the total. This steep drop is consistent with the massive reduction in traffic volume caused by the onset of the COVID-19 pandemic lockdowns in New York City during the spring of 2020.
* **Summer Rebound:** As the year progressed, accident volumes steadily climbed back up. By Month 07 (July), incidents reached 9,225 (12.32%) before slightly dipping to 8,747 (11.68%) in Month 08 (August).

---

### 2. Peak Danger Times (Day & Hour)

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
```

<img width="317" height="891" alt="image" src="https://github.com/user-attachments/assets/80e6270e-0903-4a1f-ad52-d5e2b8231272" />

**Query Methodology:**
* **Data Transformation:** SQLite's `strftime('%w')` extracts the day of the week as an integer (0-6). A `CASE` statement is utilized to transform these integers into readable string text (Sunday - Saturday), ensuring the output is immediately ready for stakeholder presentations.
* **Multi-Level Aggregation:** The dataset is aggregated by both the day of the week and the hour of the day using `GROUP BY`, then sorted in descending order using `ORDER BY` to instantly surface the highest-risk windows at the top of the result set.

**Micro-Level Trends: Day of Week and Hour**
Analyzing the daily and hourly data reveals clear patterns tied to commuting behavior:
* **The Most Dangerous Times:** The late afternoon and early evening hours on weekdays are the highest-risk windows. The top five most frequent times for accidents are all concentrated in this timeframe:
  1. Friday at 16:00 (4:00 PM) – 870 accidents
  2. Friday at 14:00 (2:00 PM) – 815 accidents
  3. Thursday at 16:00 (4:00 PM) – 814 accidents
  4. Friday at 17:00 (5:00 PM) – 804 accidents
  5. Friday at 18:00 (6:00 PM) – 800 accidents
* **Day of the Week Volumes:** Friday is the most dangerous day of the week overall (12,271 total accidents), followed by Thursday (11,244). Sunday is the safest day to drive, recording the lowest total volume (9,003). 
* **The Safest Times:** The early morning hours between 2:00 AM and 4:00 AM see the fewest accidents. For example, Tuesday at 4:00 AM and Thursday at 3:00 AM both recorded just 78 accidents.

### 3. Accident Severity by Borough

```sql
SELECT 
    COALESCE(BOROUGH, 'UNSPECIFIED') AS Borough,
    COUNT(*) AS Total_Accidents,
    SUM("NUMBER OF PERSONS INJURED") AS Total_Injuries,
    SUM("NUMBER OF PERSONS KILLED") AS Total_Deaths
FROM NYC_Accidents
GROUP BY Borough
ORDER BY Total_Accidents DESC;
```

<img width="404" height="176" alt="image" src="https://github.com/user-attachments/assets/779fc78f-15d8-4ad9-bfc3-4b65f31de6df" />

Data Quality Insight: Real-world data is often incomplete. The query reveals a significant flaw in the NYPD reporting pipeline, as the largest category by far is 'UNSPECIFIED' (25,741 accidents, 72 deaths). Of the properly categorized incidents, Brooklyn is the most dangerous borough with 16,907 accidents and 6,024 injuries, while Staten Island is the safest.

Macro-Level Trends: Geographic Danger Zones
The query reveals a significant flaw in the raw data collection pipeline, as the largest category by far is 'UNSPECIFIED' (25,741 accidents and 72 deaths), meaning officers or automated systems frequently leave this field blank. Of the properly categorized incidents, Brooklyn is the most dangerous borough, recording 16,907 accidents and 6,024 injuries. Queens follows closely with 14,017 accidents, while Staten Island is the safest with only 1,446 recorded incidents.

### 4. Primary Causes of Accidents

```sql
SELECT 
    "CONTRIBUTING FACTOR VEHICLE 1" AS Primary_Cause,
    COUNT(*) AS Incident_Count
FROM NYC_Accidents
WHERE "CONTRIBUTING FACTOR VEHICLE 1" IS NOT NULL 
  AND "CONTRIBUTING FACTOR VEHICLE 1" != 'Unspecified'
GROUP BY Primary_Cause
ORDER BY Incident_Count DESC
LIMIT 10;
```

<img width="381" height="291" alt="image" src="https://github.com/user-attachments/assets/a74a74b4-fdc1-451a-9974-fccb73e46a06" />

Micro-Level Trends: Driver Behavior
The overwhelming primary cause of traffic collisions in New York City is "Driver Inattention/Distraction," which accounts for 19,123 incidents. This is nearly four times higher than the second leading cause, "Following Too Closely" (5,202 incidents). The top three causes are rounded out by "Failure to Yield Right-of-Way" (4,815 incidents), indicating that human error, distraction, and aggressive driving behaviors are the predominant drivers of accidents rather than environmental or mechanical factors.

## Conclusion
This exploratory data analysis reveals highly predictable spatial, temporal, and behavioral patterns in New York City traffic accidents. The data indicates that collisions peak during the winter months and late Friday afternoon commutes, suggesting that fatigue and rush-hour volume heavily influence accident rates. Geographically, Brooklyn experiences the highest volume of localized incidents. Most importantly, the overwhelming prevalence of "Driver Inattention/Distraction" as a root cause highlights that human error—rather than environmental factors—drives the majority of these incidents. 

From a strategic standpoint, city planners, NYPD traffic enforcement, and emergency medical services could optimize their resource allocation by heavily targeting Friday evening commutes and launching targeted public awareness campaigns regarding distracted driving. Furthermore, the high volume of 'UNSPECIFIED' geographic data exposes a critical flaw in the NYPD's reporting pipeline that must be addressed to improve future data integrity.
