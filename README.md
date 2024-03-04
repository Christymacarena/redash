# Redash Portfolio Project 
Here is the dashboard we built, [watch and enjoy](http://localhost:5000/public/dashboards/c5ojCwJIpz885OpA5ALjdUjjhpkzqV6DLoSoT1c2?org_slug=default).

For those who have some time, here how it was..

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

<img src="https://github.com/Christymacarena/redash/assets/110884096/68f2e93c-412e-40c7-9475-9c3b6da1a9ee" alt="Connect to Clickhouse" width="40%">

## Crafting the Dashboard

With all systems connected and automated, I set out to leverage this powerful toolkit to construct the company's dashboard.

<img src="https://github.com/Christymacarena/redash/assets/110884096/6852009b-b107-4dc6-9a01-b9696c2c6b34" alt="Dashboard Preview" width="50%">

One of the standout features of Redash is its multifilter functionality, allowing users to extract precise insights from their datasets effortlessly.

<img src="https://github.com/Christymacarena/redash/assets/110884096/e4752e47-bcdd-4ef8-82bb-621118fd5e37" alt="Multifilter Visualization" width="50%">

## Exploring Data Insights

I delved into SQL queries to uncover insights from the data, crafting queries that offered valuable perspectives on business metrics.

<details>
  <summary>Click to expand SQL Code</summary> 
\`\`\`sql
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
\`\`\`
</details>


## Visualizing Insights

Armed with aggregated values and diverse timelines, I embarked on a journey of visualization, unlocking a wealth of insights.

1. Time Series Analysis:

-Line charts showing trends in Gross Profit and Total Quantity over last year

<details>
  <summary>Click to expand SQL Code</summary>
```sql
  
SELECT
    toStartOfMonth(toDate(Date)) AS Month,
    SUM(Quantity) AS Total_Quantity,
    Country,
    State,
    `Product Category` AS PCategory,
    `Sub Category` AS SubCategory,
    SUM(Revenue - Cost) AS Gross_Profit, -- Calculate total gross profit
    SUM(
        CASE WHEN `Customer Gender` = 'F' THEN 1 ELSE 0 END
    ) AS Female_Count,
    AVG(
        CASE WHEN `Customer Gender` = 'F' THEN `Customer Age` END
    ) AS Avg_Female_Age,
    SUM(
        CASE WHEN `Customer Gender` = 'M' THEN 1 ELSE 0 END
    ) AS Male_Count,
    AVG(
        CASE WHEN `Customer Gender` = 'M' THEN `Customer Age` END
    ) AS Avg_Male_Age
FROM
    default.salesforcourse_4fe2kehu
WHERE 
    toStartOfMonth(toDate(Date)) BETWEEN '{{ dates.start }}' AND '{{ dates.end }}'
GROUP BY
    Month,
    Country,
    State,
    PCategory, 
    SubCategory
ORDER BY
    Gross_Profit DESC;
</details>



-Stacked bar charts comparing Total Quantity and Gross Profit between the last month and previus month

<details>
  <summary>Click to expand SQL Code</summary> 
```sql
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
</details>


2.Geospatial Analysis:

-Geographical heatmaps showing Average Revenue per Transaction by Country (sorry, server was too slow to work woth these data)
-Even can be a fancy Bubble map visualizing the distribution of Female Count and Male Count by geographical regions, but we don't hace lat and long:) so next time.

3.Product Category Analysis:

-Horizontal bar charts displaying Total Quantity and Gross Profit by Product Category.
-Treemaps representing the hierarchy of Product Categories and Subcategories based on Gross Profit.
Buuuuuuuuuut. i dint have Treemap in my version of Redash so i provided sunburns sequence

<details>
  <summary>Click to expand SQL Code</summary>
```sql
WITH SequenceCTE AS (
  SELECT
    "Product Category" AS stage1,
    "Sub Category" AS stage2,
    NULL AS stage3,
    NULL AS stage4,
    NULL AS stage5,
    COUNT(*) AS value
  FROM
    default.salesforcourse_4fe2kehu
  WHERE 
    toStartOfMonth(toDate("Date")) BETWEEN '{{ dates.start }}' AND '{{ dates.end }}'
  GROUP BY
    "Product Category",
    "Sub Category"
)
SELECT
  stage1,
  stage2,
  stage3,
  stage4,
  stage5,
  value
FROM
  SequenceCTE
ORDER BY
  value DESC;
</details>


3.Customer Demographics Analysis:

-Pie charts illustrating the distribution of Female and Male customers.
-Box plots showcasing the distribution of Average Female Age and Average Male Age across different product categories.

<details>
  <summary>Click to expand SQL Code</summary>
```sql
WITH GenderCounts AS (
    SELECT
        SUM(
            CASE WHEN `Customer Gender` = 'F' THEN 1 ELSE 0 END
        ) AS Female_Count,
        SUM(
            CASE WHEN `Customer Gender` = 'M' THEN 1 ELSE 0 END
        ) AS Male_Count,
        AVG(
            CASE WHEN `Customer Gender` = 'F' THEN "Revenue" - "Cost" ELSE NULL END
        ) AS Avg_Female_Gross_Profit,
        AVG(
            CASE WHEN `Customer Gender` = 'M' THEN "Revenue" - "Cost" ELSE NULL END
        ) AS Avg_Male_Gross_Profit,
        SUM(
            CASE WHEN `Customer Gender` = 'F' THEN "Quantity" ELSE 0 END
        ) AS Total_Female_Quantity,
        SUM(
            CASE WHEN `Customer Gender` = 'M' THEN "Quantity" ELSE 0 END
        ) AS Total_Male_Quantity
    FROM
        default.salesforcourse_4fe2kehu
    WHERE 
        toStartOfMonth(toDate(Date)) BETWEEN '{{ dates.start }}' AND '{{ dates.end }}'
)
SELECT
    'Female' AS Gender,
    Female_Count AS Count,
    Avg_Female_Gross_Profit AS Avg_Gross_Profit,
    Total_Female_Quantity AS Total_Quantity
FROM
    GenderCounts
UNION ALL
SELECT
    'Male' AS Gender,
    Male_Count AS Count,
    Avg_Male_Gross_Profit AS Avg_Gross_Profit,
    Total_Male_Quantity AS Total_Quantity
FROM
    GenderCounts;
</details>


4. Comparative Analysis:

*By juxtaposing metrics from different periods, our Comparative Analysis provides a comprehensive view of performance evolution, empowering stakeholders to make data-driven decisions and optimize business strategies
-Dual-axis line charts comparing Total Quantity and Gross Profit between the Current Period and Comparative Period.
-Side-by-side bar charts displaying Total Quantity and Gross Profit for the Current Period and Comparative Period.

<details>
  <summary>Click to expand SQL Code</summary>
```sql
WITH CohortCTE AS (
    SELECT
        toStartOfMonth(Date) AS Cohort_Month,
        "Product Category" AS Product_Category,
        AVG("Unit Price" - "Unit Cost") AS Avg_Profit_Margin,
        SUM(Revenue - Cost) AS Total_Profit
    FROM
        default.salesforcourse_4fe2kehu
    GROUP BY
        Cohort_Month,
        Product_Category
    ORDER BY
        Cohort_Month
)
SELECT
    Cohort_Month,
    Product_Category,
    AVG(Avg_Profit_Margin) AS Avg_Profit_Margin,
    SUM(Total_Profit) AS Total_Profit
FROM
    CohortCTE
GROUP BY
    Cohort_Month,
    Product_Category
ORDER BY
    Cohort_Month;
</details>


5. Overall Performance Metrics:

-KPI widgets showing aggregated metrics such as Total Quantity, Gross Profit, Female Count, and Male Count.
Gauges representing the percentage change in Total Quantity and Gross Profit compared to the previous period.

6. Customer Segmentation:

-Scatter plots visualizing the relationship between Gross Profit and Customer Age, segmented by gender.
-Radar charts comparing the average Gross Profit and Total Quantity for different customer segments.

## So let's go straight to conclusions:


Customer Demographics:
Majority of customers fall within the age range of 32-45 years, indicating a mature target audience.
Gender distribution is nearly equal, with approximately 47-53 split between male and female customers.
The average age of customers across different product categories ranges from 34 to 38 years, with the youngest customers observed in the road bikes category.

Product Performance:
Accessories emerge as the largest category of goods sold, with notable popularity in subcategories such as tires and tubes, and bottles and cages.
Assessing metrics like Total Quantity, Revenue, and Gross Profit helps identify top-performing products or categories, offering insights into factors driving their success.
Understanding the relationship between Unit Cost, Unit Price, and Gross Profit enables optimization of pricing strategies to enhance profitability.

Geospatial Analysis:
The US leads in terms of quantity of goods sold across all categories, while Germany boasts higher average revenue per transaction, suggesting lucrative market potential.
Geographical heatmaps visualize revenue or sales distribution across different countries or regions, facilitating targeted marketing efforts.
Time Series Analysis:

The business witnessed significant improvement from 2015 to 2016, with Gross Profit turning consistently positive over time.
Tracking trends in Total Quantity, Revenue, and Gross Profit helps identify seasonal patterns and long-term growth trends, enabling proactive business decisions.Customer Segmentation:
Despite limited data, the gender distribution analysis provides insights into customer demographics, aiding in personalized marketing campaigns.
Analyzing customer retention rates and lifetime value guides prioritization of customer acquisition and retention strategies.

Cost Analysis:
Evaluating cost distribution across product categories helps identify areas for cost optimization and potential cost-saving opportunities.
Products with high margins and low production costs should be prioritized in marketing efforts to maximize profitability and cost-effectiveness.
In summary, leveraging insights from customer demographics, product performance, geospatial analysis, time series analysis, customer segmentation, and cost analysis enables data-driven decision-making and strategic planning for business growth and profitability.
