# redash
My experiment to open Redash to run powerful portfolio project:)

I decided to return for my grait research of the company i can put my energy and lovve to, so i decises to improve businecc analytica portpholio i had. I google open source dashboard creator and found Redash. Sounded nice, so i started to build the 'company' print to show my skills in the field. 
I started with opening a local server with Docker to merge the dashboard with Data source (i still didnt have any, but i trusted in myself).
So here we go: develope a new server ![run local server](https://github.com/Christymacarena/redash/assets/110884096/a22c9b9e-ae6a-4a43-9c15-f14d21271947)
Everything is working now so i can go do magic on  http://localhost:5000/ of Radish
Now time for uploading data: let start with PostgreSQL as a usual datamanager for me. But doesnt matter how many shaman dances i did - Redash could reach the 5432 port ( ![postgres](https://github.com/Christymacarena/redash/assets/110884096/eb07d941-ac43-4d1b-a7ca-9b8198d76213)
Thanks godness of internet we can connect almost all databases to this App, so i continued with Clickhouse.
It would be too easy if it worked from the first attemp. So after research about differences if URL and CURL of the link [click problems](https://github.com/Christymacarena/redash/assets/110884096/68f2e93c-412e-40c7-9475-9c3b6da1a9ee) i did the connection ![click](https://github.com/Christymacarena/redash/assets/110884096/d1edd5d3-8110-42c7-a171-1a352828cc18)

So finally we have everything connected and automatized for using this set of tools to build a company's dashboard
![finally](https://github.com/Christymacarena/redash/assets/110884096/6852009b-b107-4dc6-9a01-b9696c2c6b34)

The main differences i found about redish itself is those multifilers you can create to reach distinct info exactly from the edited set. which is interesting to try ![image](https://github.com/Christymacarena/redash/assets/110884096/e4752e47-bcdd-4ef8-82bb-621118fd5e37)

So i try to wake my sql skills to check what we can see from data 
"
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
    Gross_Profit DESC
"
Now we have some agregated values and different timelines to see a perspective of the business.
Lets try to see maximum of it:

1. Time Series Analysis:

-Line charts showing trends in Gross Profit and Total Quantity over last year - newplot png
-Stacked bar charts comparing Total Quantity and Gross Profit between the last month and previus month newplot (1)

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

Customer Segmentation:

Scatter plots visualizing the relationship between Gross Profit and Customer Age, segmented by gender.
Radar charts comparing the average Gross Profit and Total Quantity for different customer segments.
 
