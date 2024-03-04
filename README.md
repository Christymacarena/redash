# Redash Portfolio Project 

## Introduction

In my quest to find a meaningful project to channel my energy and passion, I decided to enhance my business analytics portfolio. My search led me to discover Redash, an open-source dashboard creator. Intrigued by its capabilities, I began crafting the blueprint for a comprehensive company dashboard to showcase my skills in data analysis and visualization.

## Setting Up the Environment

I kicked off by setting up a local server using Docker and seamlessly integrated it with my chosen data sources. Although I didn't have a dataset at hand initially, I was confident in my abilities to bring the project to life.

<img src="https://github.com/Christymacarena/redash/assets/110884096/a22c9b9e-ae6a-4a43-9c15-f14d21271947" alt="Run Local Server" width="50%">

With the local server up and running, I dove into the world of Redash, exploring its vast potential at http://localhost:5000/.

## Connecting Data Sources

My journey into data visualization began with PostgreSQL, my trusted companion in data management. Despite encountering some initial hiccups in connecting to the 5432 port, I persevered and successfully dismantled the connection. 

<img src="https://github.com/Christymacarena/redash/assets/110884096/eb07d941-ac43-4d1b-a7ca-9b8198d76213" alt="Connect to PostgreSQL" width="50%">

Determined to explore the full spectrum of possibilities, I ventured into Clickhouse. Overcoming challenges with URL and CURL configurations, I triumphed in establishing a seamless connection.

<img src="https://github.com/Christymacarena/redash/assets/110884096/68f2e93c-412e-40c7-9475-9c3b6da1a9ee" alt="Connect to Clickhouse" width="50%">

## Crafting the Dashboard

With all systems connected and automated, I set out to leverage this powerful toolkit to construct the company's dashboard.

<img src="https://github.com/Christymacarena/redash/assets/110884096/6852009b-b107-4dc6-9a01-b9696c2c6b34" alt="Dashboard Preview" width="50%">

One of the standout features of Redash is its multifilter functionality, allowing users to extract precise insights from their datasets effortlessly.

<img src="https://github.com/Christymacarena/redash/assets/110884096/e4752e47-bcdd-4ef8-82bb-621118fd5e37" alt="Multifilter Visualization" width="50%">

## Exploring Data Insights

I delved into SQL queries to uncover insights from the data, crafting queries that offered valuable perspectives on business metrics.

<details>
  <summary>Click to expand SQL Code</summary>
  
```sql
-- Your SQL code goes here
SELECT
    toStartOfMonth(toDate(Date)) AS Month,
    multiIf(
        '{{ cmp }}' = 'No', 
            if(
                toStartOfMonth(toDate(Date)) BETWEEN '{{ dates.start }}' AND '{{ dates.end }}', 
                'Current period', 
                'Comparative period'
            ),
        '{{ cmp }}' = 'With last month', 
            if(
                (toDate(addMonths(Date, -1)) BETWEEN '{{ dates.start }}' AND '{{ dates.end }}') 
                OR 
                ( toStartOfMonth(toDate(Date)) BETWEEN '{{ dates.start }}' AND '{{ dates.end }}'), 
                'Current period', 
                'Comparative period'
            ),
        NULL
    ) AS Comparing,
    SUM(Quantity) AS Total_Quantity,
    Country,
    State,
    `Product Category` AS PCategory,
    `Sub Category` AS SubCategory,
    SUM(Revenue) - SUM(Cost) AS Gross_Profit,
    SUM(
        CASE
            WHEN `Customer Gender` = 'F' THEN 1
            ELSE 0
        END
    ) AS Female_Count,
    AVG(
        CASE
            WHEN `Customer Gender` = 'F' THEN `Customer Age`
        END
    ) AS Avg_Female_Age,
    SUM(
        CASE
            WHEN `Customer Gender` = 'M' THEN 1
            ELSE 0
        END
    ) AS Male_Count,
    AVG(
        CASE
            WHEN `Customer Gender` = 'M' THEN `Customer Age`
        END
    ) AS Avg_Male_Age
FROM
    default.salesforcourse_4fe2kehu
WHERE 
    (
        '{{ cmp }}' = 'No' AND 
        toStartOfMonth(toDate(Date)) BETWEEN '{{ dates.start }}' AND '{{ dates.end }}'
    )
    OR
    (
        '{{ cmp }}' = 'With last month' AND 
        (
            (
                toStartOfMonth(toDate('{{ dates.end }}')) = toStartOfMonth(toDate(addMonths(Date, -1))) 
                AND 
                toStartOfMonth(toDate('{{ dates.start }}')) = toStartOfMonth(toDate(Date))
            ) 
             OR
            (
        '{{ cmp }}' = 'With last month' AND 
        (
            (toDate(addMonths(Date, 1)) BETWEEN '{{ dates.start }}' AND '{{ dates.end }}') 
            OR 
            (toDate(Date) BETWEEN '{{ dates.start }}' AND '{{ dates.end }}')
        )
            )
        )
    )
GROUP BY
    Month,
    Comparing,
    Country,
    State,
    PCategory, 
    SubCategory
ORDER BY
    Gross_Profit DESC;

## Visualizing Insights

Armed with aggregated values and diverse timelines, I embarked on a journey of visualization, unlocking a wealth of insights.

1. Time Series Analysis:

-Line charts showing trends in Gross Profit and Total Quantity over last year - newplot png
-Stacked bar charts comparing Total Quantity and Gross Profit between the last month and previus month newplot (1)

<details>
  <summary>Click to expand SQL Code</summary>
  
```sql
-- Your SQL code goes here
SELECT * FROM your_table;

2.Geospatial Analysis:

-Geographical heatmaps showing Average Revenue per Transaction by Country
-Even can be a fancy Bubble map visualizing the distribution of Female Count and Male Count by geographical regions, but we don't hace lat and long:) so next time.

3.Product Category Analysis:

-Horizontal bar charts displaying Total Quantity and Gross Profit by Product Category.
-Treemaps representing the hierarchy of Product Categories and Subcategories based on Gross Profit.
Buuuuuuuuuut. i dint have Treemap in my version of Redash so i will do suburns sequence

3.Customer Demographics Analysis:

-Pie charts illustrating the distribution of Female and Male customers.
-Box plots showcasing the distribution of Average Female Age and Average Male Age across different product categories.

4. Comparative Analysis:

Dual-axis line charts comparing Total Quantity and Gross Profit between the Current Period and Comparative Period.
Side-by-side bar charts displaying Total Quantity and Gross Profit for the Current Period and Comparative Period.

5. Overall Performance Metrics:

-KPI widgets showing aggregated metrics such as Total Quantity, Gross Profit, Female Count, and Male Count.
Gauges representing the percentage change in Total Quantity and Gross Profit compared to the previous period.

6. Customer Segmentation:

-Scatter plots visualizing the relationship between Gross Profit and Customer Age, segmented by gender.
Radar charts comparing the average Gross Profit and Total Quantity for different customer segments.
 
