
# Telecommunications Site Outages Analysis and Reporting

![telco-image](assets/images/powerbi_dashboard.png)
[Interactive Report](https://app.powerbi.com/view?r=eyJrIjoiOTk0NzY4ODItZjMzOC00NDhiLWI4NGQtZDI3OThhM2E4MmJlIiwidCI6ImRmODY3OWNkLWE4MGUtNDVkOC05OWFjLWM4M2VkN2ZmOTVhMCJ9)  

## Table of contents

- [Objective](#objective)
- [Data Collection and Preparation](#data-collection-and-preparation)
  - [Dataset Acquisition](#dataset-acquisition)
  - [Data Cleaning](#data-cleaning)
- [Data Integration and Modeling in Power BI](#data-integration-and-modeling-in-power-bi)
  - [Loading Data into Power BI](#loading-data-into-power-bi)
  - [Creating a Calendar Table](#creating-a-calendar-table)
  - [Data Modeling](#data-modeling)
- [Measure Creation](#measure-creation)
  - [Matrix Table Measures](#matrix-table-measures)
  - [Performance Table Measures](#performance-table-measures)
  - [Outages Table Measures](#outages-table-measures)
- [Data Visualization](#data-visualization)
  - [Performance Over Time](#performance-over-time)
  - [Regional Performance](#regional-performance)
  - [Interactive Filtering](#interactive-filtering)
- [Conclusion](#conclusion)

## Objective

The goal of this project is to analyze and visualize the performance and availability of telecommunications sites over a specified period. By integrating and modeling outage data with site information and creating detailed reports, we aim to provide actionable insights into site performance and identify trends and patterns in outages.

## Data Collection and Preparation

### Dataset Acquisition

- A dataset was downloaded containing records of telecommunications site outages, including the availability of each site, measured as a percentage of each day. [Datasets](assets/datasets)

### Data Cleaning

- The raw outage data was loaded into a Pandas DataFrame for cleaning, which involved handling missing values, correcting data types, and removing any inconsistencies. [Notebook Link](assets/performance%20and%20outage%20EDA%20and%20Cleaning.ipynb)
- The cleaned data was then exported as a CSV file for further analysis.


## Data Integration and Modeling in Power BI

### Loading Data into Power BI

- The cleaned outage data CSV file was imported into Power BI.
- An additional database containing comprehensive information about the telecommunications sites was also loaded.

### Creating a Calendar Table

  A calendar table was generated in Power BI to facilitate time-based analysis of the site's performance. This table includes all the necessary dates over the period of analysis. Table is also dynamic, so that end date is always "today"
  The Year, Quater, Month columns are also created

```DAX
calender = CALENDAR(DATE(2022,01,01),TODAY())

Year = 
YEAR(calender[Date]
)

Quarter = 
"Qtr" & QUARTER(calender[Date])

Month Name = 
FORMAT(
    DATE(
        1,
        'calender'[Month#],
        1
    ),
    "mmmm"
)

Month# = 
FORMAT(MONTH([Date]), "00")

Month = CONCATENATE([Month#],[Month Name])
```

### Data Modeling

- The four tables (cleaned outage and performance data, site information, and calendar table) were modeled and joined on relevant keys:
  - The outage and performance data was joined to the site information on the "Site ID".
  - The outage and performance data was joined to the calendar table on the "Date".
  - Only "One to Many (1:*)" cardinality is used 
![model-image](assets/images/data_model.png)

### Measure/Column Creations on Cleaned Data
  This section outlines the measures and columns created on the cleaned data for analysis. The calculations use DAX (Data Analysis Expressions) to derive insights from the data.

#### Performance Table

- **Average Performance:**
  This measure calculates the average performance across all selected filters, providing a quick snapshot of overall performance levels.
  ```DAX
  average performance = 
  AVERAGE(daily_availability[Performance]
        )
  ```

#### Outage Table

- **Total Outages:**
  This measure counts the total number of outage entries in the outage_report table, giving a summary of all recorded outages.
  ```DAX
  Total Outages_atc = 
  COUNTROWS(outage_report)
  ```

- **Checking for Repeated Outages:**
  This measure identifies repeated outages by checking for multiple occurrences of the same outage_code on the same day. It helps in analyzing the frequency of repeated failures due to the same root cause.
  ```DAX
  Repeated Outages_atc = 
  VAR CurrentRowDate = [Alarm Start Time]
  VAR CurrentRepeatedOutage = [outage_code]
  VAR SameDayOccurrences =
      CALCULATE(
          COUNTROWS('outage_report'),
          FILTER(
              'outage_report',
              YEAR([Alarm Start Time]) = YEAR(CurrentRowDate) &&
              MONTH([Alarm Start Time]) = MONTH(CurrentRowDate) &&
              DAY([Alarm Start Time]) = DAY(CurrentRowDate) &&
              [outage_code] = CurrentRepeatedOutage
          )
      )
  RETURN
      IF(SameDayOccurrences > 1, 1, 0)
  ```

- **Total Repeated Outages:**
  This measure sums up all the repeated outages, providing a total count of how many outages were repeated based on the previous calculation.
  ```DAX
  Total Repeated Outages atc = 
  SUM(outage_report[Repeated Outages_atc])
  ```

- **Average Downtime:**
  This measure calculates the average downtime for each outage, providing an average time to repair (MTTR) metric. It converts the total duration from minutes into a more readable format of hours and minutes.
  ```DAX
  AverageDowntime_atc = 
  VAR TotalMinutes = SUM(outage_report[DURATION]) / [Total Outages_atc]
  VAR RoundedMinutes = ROUND(TotalMinutes, 0)
  VAR Hours = INT(RoundedMinutes / 60)
  VAR Minutes = MOD(RoundedMinutes, 60)
  RETURN
  IF(
      Minutes < 10, 
      Hours & ":0" & Minutes, 
      Hours & ":" & Minutes
  )
  ```

#### Escalation Matrix

- **Site Count:**
  This measure counts the total number of sites managed, as listed in the escalation_matrix table, providing a count of all monitored sites.
  ```DAX
  Site Count_atc = 
  COUNTROWS(escalation_matrix)
  ```

### Additional notes
- Average Performance: Helps quickly gauge the overall performance based on selected filters, allowing for immediate insights into performance trends.
- Total Outages: Useful for understanding the frequency and volume of outages, essential for operational and performance analysis.
- Checking for Repeated Outages: This measure is crucial for identifying systemic issues or recurring problems that may need attention.
- Total Repeated Outages: Summarizes repeated outages to help quantify the impact of recurring issues.
- Average Downtime: Provides a clear view of the average time to repair, which is essential for assessing maintenance efficiency and responsiveness.
- Site Count: Offers a straightforward count of the sites being managed, which can be useful for capacity planning and resource allocation.


## Data Visualization on Power BI

### Performance Over Time

- A line chart was used to visualize site performance over time, showing trends in availability, A constant line is also included to show target
- A clustered bar chart is used to show outages in compparison to repeated outages over same period

### Regional Performance

- A bar chart was employed to display performance metrics per regional manager and clusters, allowing for a clear comparison of regional performance.

### Interactive Filtering

- Slicers were created to enable interactive filtering of the data, allowing users to focus on specific regions, site types, or time periods as needed.

## Conclusion

By integrating outage data with site information and utilizing Power BI's robust modeling and visualization capabilities, this project provides a comprehensive view of telecommunications site performance. The insights gained can be used to improve site reliability, allocate resources more effectively, and address recurring issues in a timely manner. [Click here for Interactive Power BI Report](https://app.powerbi.com/view?r=eyJrIjoiOTk0NzY4ODItZjMzOC00NDhiLWI4NGQtZDI3OThhM2E4MmJlIiwidCI6ImRmODY3OWNkLWE4MGUtNDVkOC05OWFjLWM4M2VkN2ZmOTVhMCJ9)
