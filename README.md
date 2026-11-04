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
*Please see the attached `.sql` files in this repository for the complete code used to extract these insights.*
