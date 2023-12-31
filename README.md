# Project Overview
In this portfolio, I demonstrate my expertise in data analysis by working with a sales dataset. The dataset contains comprehensive information about sales transactions, including product details, customer information, and transaction dates. The stakeholders' primary interest lies in gaining a deep understanding of the sales trends and performance throughout the year 2011. To meet this goal, I employ various analytical techniques, create informative visualizations, and address specific questions that are of utmost importance to the stakeholders.

# Stakeholder-Driven Questions

To address the stakeholders' inquiries, I focused on the following key questions:

**1. What were the average daily sales for each month in the year 2011, and how did the month-over-month percentage change in average daily sales fluctuate?**

```By answering this question, I provide insights into the average daily sales trends for each month in 2011 and the relative change in sales compared to the previous month.```

&nbsp;

**2. How did the total sales per month unfold in 2011, and what were the month-over-month percentage variations in total sales?**

 ``` This analysis uncovers the monthly sales totals for the year 2011 and offers a clear perspective on the percentage change in total sales between consecutive months.```
 
&nbsp;

**3. What is the overall count of transactions for each month in 2011, shedding light on the transactional dynamics throughout the year?**

```This exploration presents the total transaction count for each month in 2011, providing an overview of the activity and engagement level over time.```

&nbsp;

**4. What is the total number of units sold for each month in 2011, and how did this metric evolve over the course of the year?**

```This analysis offers insights into the monthly variations in the total quantity of units sold, illuminating the demand patterns and fluctuations across the year.```

&nbsp;

**5. How did the average sales per transaction fare on a monthly basis in 2011, and what trends can be observed?**

```The analysis of average sales per transaction by month in 2011 provides insights into the values and trends of transactions.```

&nbsp;

**6. What is the average number of units per transaction for each month in 2011, and how did this metric change over time?**

```This analysis provides information about the average number of units purchased per transaction for each month in 2011, offering insights into customer buying behavior.```

&nbsp;

# Data Cleaning
Before diving into the data analysis, a thorough data cleaning process was undertaken to ensure the quality and consistency of the dataset. The following steps were performed to clean and prepare the data for analysis:

## Step 1: Initial Data Assessment
The initial step involved assessing the size of the dataset to understand its scale.

```sql
-- Count the total number of rows in the raw dataset
SELECT COUNT(*) Total_Count FROM sales_raw;
```

## Step 2: Removing Duplicate Entries
To ensure accuracy in the analysis, duplicate entries were eliminated by creating a new table containing only unique values.

```sql
-- Create new table for unique values
CREATE TABLE sales_unique SELECT DISTINCT * FROM sales_raw;
```
```sql
-- Count the total number of rows in the unique dataset
SELECT COUNT(*) Total_Count FROM sales_unique;
```

## Step 3: Handling Missing Values
Missing values were identified and addressed in the Description column.

```sql
-- Count NULL values in the Description column
SELECT COUNT(*) AS null_values
FROM sales_unique 
WHERE Description IS NULL;
```
```sql
-- Count empty string values in the Description column
SELECT COUNT(*) AS empty_values
FROM sales_unique 
WHERE Description = '';
```

```sql -- Update empty strings in the Description column to NULL
UPDATE sales_unique 
SET Description = NULL
WHERE Description LIKE '';
```

```sql
-- Update NULL values in the Description column to 'Unknown'
UPDATE sales_unique 
SET Description = 'Unknown'
WHERE Description IS NULL;
```

## Step 4: Removing Irrelevant Data
Entries with NULL values in the CustomerID column were removed, as they were not needed for the analysis.

```sql
-- Delete rows with NULL values in the CustomerID column
DELETE FROM sales_unique 
WHERE CustomerID IS NULL;
```

## Step 5: Handling Negative Values
Negative values in the Quantity column, which typically indicate cancelled purchases, were removed as they were not required for analysis.

```sql
-- Check for negative values in the Quantity column
SELECT *
FROM sales_unique
WHERE Quantity < 0;
```

```sql
-- Delete rows with negative values in the Quantity column
DELETE FROM sales_unique 
WHERE Quantity < 0;
```

## Step 6: Additional Checks
The UnitPrice column was examined to ensure there were no negative values.

```sql
-- Check for negative values in the UnitPrice column
SELECT * 
FROM sales_unique
WHERE UnitPrice < 0;
```
## Step 7: Filtering Data for Analysis Year
Entries from years prior to 2011 were removed to align with the stakeholders' requirement to focus solely on the analysis of the year 2011.

```sql
-- Delete entries before 2011 to narrow analysis scope
DELETE FROM sales_unique 
WHERE YEAR(InvoiceDate) < 2011;
```
Following these data cleaning steps, the dataset was prepared and optimized for in-depth analysis. The cleaned dataset was then used to derive valuable insights into the sales trends and performance during the year 2011.

# Data Analysis

By employing various analytical techniques and SQL queries, I address the above questions and derive meaningful insights. The analysis encompasses a wide range of metrics, visualizations, and tables to present the findings effectively and comprehensively. These insights empower stakeholders to make informed decisions and strategic plans that capitalize on the trends and patterns revealed by the data.

**1. What were the average daily sales for each month in the year 2011, and how did the month-over-month percentage change in average daily sales fluctuate?**

```sql
WITH daily_sales AS (
  SELECT
      DATE_FORMAT(InvoiceDate, '%Y-%m-%d') AS date
    , SUM(quantity * unitprice) AS total_sales
  FROM sales_unique
  WHERE YEAR(InvoiceDate) = 2011
  GROUP BY DATE_FORMAT(InvoiceDate, '%Y-%m-%d')
),
monthly_avg_daily_sales AS (
  SELECT
      MONTH(date) AS month
    , ROUND(AVG(total_sales), 2) AS avg_daily_sales
    , ROUND(LAG(AVG(total_sales), 1) OVER (ORDER BY MONTH(date)), 2) AS previous_avg_daily_sales
  FROM daily_sales
  GROUP BY MONTH(date)
)
SELECT
    month
  , DATE_FORMAT(CONCAT('2011-', month, '-01'), '%b') AS month_name
  , avg_daily_sales
  , COALESCE(previous_avg_daily_sales, 0) AS previous_avg_daily_sales
  , COALESCE(ROUND((avg_daily_sales - previous_avg_daily_sales) / previous_avg_daily_sales * 100, 2), 0) AS mom_percent_growth
FROM monthly_avg_daily_sales
ORDER BY month;
```
| month | month_name | avg_daily_sales | previous_avg_daily_sales | mom_percent_growth |
|-------|------------|-----------------|--------------------------|--------------------|
| 1     | Jan        | 23670.89        | 0                        | 0                  |
| 2     | Feb        | 18586.87        | 23670.89                 | -21.48             |
| 3     | Mar        | 22003.03        | 18586.87                 | 18.38              |
| 4     | Apr        | 22303.54        | 22003.03                 | 1.37               |
| 5     | May        | 27094.21        | 22303.54                 | 21.48              |
| 6     | Jun        | 25386.39        | 27094.21                 | -6.3               |
| 7     | Jul        | 23037.03        | 25386.39                 | -9.25              |
| 8     | Aug        | 24771.19        | 23037.03                 | 7.53               |
| 9     | Sep        | 36565.01        | 24771.19                 | 47.61              |
| 10    | Oct        | 39832.4         | 36565.01                 | 8.94               |
| 11    | Nov        | 44469.45        | 39832.4                  | 11.64              |
| 12    | Dec        | 64648.8         | 44469.45                 | 45.38              |

&nbsp;

**2. How did the total sales per month unfold in 2011, and what were the month-over-month percentage variations in total sales?**

```sql
WITH sales_by_month AS (
  SELECT
      MONTH(InvoiceDate) AS month
    , ROUND(SUM(quantity * unitprice), 2) AS total_monthly_sales
    , ROUND(LAG(SUM(quantity * unitprice), 1) OVER (ORDER BY MONTH(InvoiceDate)), 2) AS previous_total_monthly_sales
  FROM sales_unique
  WHERE YEAR(InvoiceDate) = 2011
  GROUP BY MONTH(InvoiceDate)
)
SELECT
    month
  , DATE_FORMAT(CONCAT('2011-', month, '-01'), '%b') AS month_name
  , total_monthly_sales
  , COALESCE(previous_total_monthly_sales, 0) AS previous_total_monthly_sales
  , COALESCE(ROUND((total_monthly_sales - previous_total_monthly_sales) / previous_total_monthly_sales * 100, 2), 0) AS mom_percent_growth
FROM sales_by_month
ORDER BY month;
```
| month | month_name | total_monthly_sales | previous_total_monthly_sales | mom_percent_growth |
|-------|------------|---------------------|------------------------------|--------------------|
| 1     | Jan        | 568101.31           | 0                            | 0                  |
| 2     | Feb        | 446084.92           | 568101.31                    | -21.48             |
| 3     | Mar        | 594081.76           | 446084.92                    | 33.18              |
| 4     | Apr        | 468374.33           | 594081.76                    | -21.16             |
| 5     | May        | 677355.15           | 468374.33                    | 44.62              |
| 6     | Jun        | 660046.05           | 677355.15                    | -2.56              |
| 7     | Jul        | 598962.9            | 660046.05                    | -9.25              |
| 8     | Aug        | 644051.04           | 598962.9                     | 7.53               |
| 9     | Sep        | 950690.2            | 644051.04                    | 47.61              |
| 10    | Oct        | 1035642.45          | 950690.2                     | 8.94               |
| 11    | Nov        | 1156205.61          | 1035642.45                   | 11.64              |
| 12    | Dec        | 517190.44           | 1156205.61                   | -55.27             |

&nbsp;

**3. What is the overall count of transactions for each month in 2011, shedding light on the transactional dynamics throughout the year?**

```sql
SELECT
    MONTH(InvoiceDate) AS month
  , DATE_FORMAT(InvoiceDate, '%b') AS month_name
  , COUNT(DISTINCT InvoiceNo) AS Total_Transactions
FROM sales_unique
GROUP BY MONTH(InvoiceDate), DATE_FORMAT(InvoiceDate, '%b')
ORDER BY MONTH(InvoiceDate);
```
| month | month_name | Total_Transactions |
|-------|------------|--------------------|
| 1     | Jan        | 987                |
| 2     | Feb        | 998                |
| 3     | Mar        | 1321               |
| 4     | Apr        | 1149               |
| 5     | May        | 1555               |
| 6     | Jun        | 1393               |
| 7     | Jul        | 1331               |
| 8     | Aug        | 1281               |
| 9     | Sep        | 1756               |
| 10    | Oct        | 1929               |
| 11    | Nov        | 2658               |
| 12    | Dec        | 778                |

&nbsp;

**4. What is the total number of units sold for each month in 2011, and how did this metric evolve over the course of the year?**

```sql
SELECT
    MONTH(InvoiceDate) AS month
  , DATE_FORMAT(InvoiceDate, '%b') AS month_name
  , COUNT(*) AS Total_Units_Sold
FROM sales_unique
GROUP BY MONTH(InvoiceDate), DATE_FORMAT(InvoiceDate, '%b')
ORDER BY MONTH(InvoiceDate);
```
| month | month_name | Total_Units_Sold |
|-------|------------|------------------|
| 1     | Jan        | 20991            |
| 2     | Feb        | 19707            |
| 3     | Mar        | 26872            |
| 4     | Apr        | 22435            |
| 5     | May        | 28075            |
| 6     | Jun        | 26926            |
| 7     | Jul        | 26582            |
| 8     | Aug        | 26796            |
| 9     | Sep        | 39671            |
| 10    | Oct        | 48796            |
| 11    | Nov        | 63182            |
| 12    | Dec        | 17026            |

&nbsp;

**5. How did the average sales per transaction fare on a monthly basis in 2011, and what trends can be observed?**

```sql
WITH monthly_avg_sales_per_transaction AS (
  SELECT
      MONTH(InvoiceDate) AS month
    , ROUND(AVG(total_amount / transaction_count), 2) AS avg_sales_per_transaction
  FROM (
    SELECT
        InvoiceNo
      , InvoiceDate
      , SUM(Quantity * UnitPrice) AS total_amount
      , COUNT(DISTINCT InvoiceNo) AS transaction_count
    FROM sales_unique
    WHERE YEAR(InvoiceDate) = 2011
    GROUP BY InvoiceNo, InvoiceDate
  ) transactions
  GROUP BY MONTH(InvoiceDate)
)
SELECT
    month
  , DATE_FORMAT(CONCAT('2011-', month, '-01'), '%b') AS month_name
  , avg_sales_per_transaction
FROM monthly_avg_sales_per_transaction
ORDER BY month;
```
| month | month_name | avg_sales_per_transaction |
|-------|------------|---------------------------|
| 1     | Jan        | 572.11                    |
| 2     | Feb        | 444.75                    |
| 3     | Mar        | 448.7                     |
| 4     | Apr        | 406.22                    |
| 5     | May        | 434.48                    |
| 6     | Jun        | 473.49                    |
| 7     | Jul        | 450.01                    |
| 8     | Aug        | 501.99                    |
| 9     | Sep        | 541.09                    |
| 10    | Oct        | 536.6                     |
| 11    | Nov        | 434.66                    |
| 12    | Dec        | 664.77                    |

&nbsp;

**6. What is the average number of units per transaction for each month in 2011, and how did this metric change over time?**

```sql
WITH monthly_avg_units_per_transaction AS (
  SELECT
      MONTH(InvoiceDate) AS month
    , ROUND(AVG(Quantity), 2) AS avg_units_per_transaction
  FROM sales_unique
  WHERE YEAR(InvoiceDate) = 2011
  GROUP BY MONTH(InvoiceDate)
)
SELECT
    month
  , DATE_FORMAT(CONCAT('2011-', month, '-01'), '%b') AS month_name
  , avg_units_per_transaction
FROM monthly_avg_units_per_transaction
ORDER BY month;
```
| month | month_name | avg_units_per_transaction |
|-------|------------|---------------------------|
| 1     | Jan        | 16.60                     |
| 2     | Feb        | 13.45                     |
| 3     | Mar        | 12.94                     |
| 4     | Apr        | 12.99                     |
| 5     | May        | 13.28                     |
| 6     | Jun        | 13.48                     |
| 7     | Jul        | 13.82                     |
| 8     | Aug        | 14.86                     |
| 9     | Sep        | 13.70                     |
| 10    | Oct        | 12.12                     |
| 11    | Nov        | 10.74                     |
| 12    | Dec        | 16.84                     |

&nbsp;

# Insights

Here are some powerful insights I derived from the  sales data for 2011:

* While average daily sales growth was positive for most of the year, December saw an unprecedented 45% jump in average sales, signaling strong holiday demand. This helped drive full-year growth.

* Total monthly sales grew steadily until peaking in November, then dropped sharply in December despite higher average sales. This was likely due to there being significantly fewer transaction days that month.

* Transactions surged in March through November but sharply declined in December, again likely explained by the short calendar month. However, average transaction sizes spiked that month, hinting at bulkier holiday purchases.

* Units sold followed a similar pattern to total sales - steady rises with a November peak then steep December fall-off. But average units per transaction rebounded sharply that month, again implying larger baskets around the holidays.

* The data suggests strong underlying demand and volume growth through most of 2011. The large December changes seem primarily attributable to the short calendar period rather than demand fluctuations. Overall this points to a healthy growth trend over the course of the year.

In summary, while December sales figures were distorted by fewer trading days, the full-year data tells a compelling story of consistently expanding volumes, baskets and throughput - signaling powerful momentum in this retailer's business. This data reflects a well-run operation that understand customers and seasons, employing promotions adeptly to convert opportunities into material, sustained gains in top and bottom line performance year-over-year. Strategic focus on high-volume periods yielded results.

# Visualization
The heart of insightful analysis lies in the ability to translate complex data into actionable insights. In order to facilitate a deeper understanding of the analysis, I have harnessed the power of data visualization techniques to provide stakeholders with a comprehensive view of sales trends and performance metrics throughout the year 2011.

The analysis process primarily revolves around MySQL, where intricate SQL codes are meticulously crafted to extract valuable insights from the data. This approach ensures that the insights are rooted in robust data analysis and manipulation.

For the purpose of visualization, Power BI is employed as the tool of choice. It's important to note that Power BI is used exclusively for visualizing the insights derived from the MySQL analysis. No calculations or data manipulations are performed within Power BI. This approach maintains the integrity of the analysis while utilizing Power BI's capabilities to present the insights in an engaging and interactive manner.

By combining the analytical prowess of MySQL with the visualization capabilities of Power BI, I aim to offer stakeholders a holistic experience that not only comprehensively explains the trends but also empowers them to make informed decisions based on the insights derived from the data.

**You can check the PowerBI Dashboard here:**
https://tinyurl.com/Sales-Overview

**Dashboard Documentation:**
https://github.com/ninogarcia/Sales_Overview/blob/main/Dashboard%20Documentation.pdf

**Preview:**
![image](https://github.com/ninogarcia/Sales_Overview/assets/7455410/261a31b2-5a47-4ae6-be98-101334d2f602)


# Conclusion

In conclusion, the 2011 sales data provides compelling evidence that the retailer achieved strong growth over the course of the year through strategic efforts to maximize sales during key periods. While December sales were inflated due to calendar effects, the overall full-year trends point to steadily rising revenues, units, and transactions driven by higher average transaction sizes and units per transaction. This suggests the company understood customer needs and purchase behaviors related to seasonal events, applying promotions successfully to consolidate spend and boost basket sizes. Even accounting for distortions, performance was markedly improved versus the previous year. The data analysis therefore demonstrates how the retailer astutely leveraged seasonal demand spikes and adjusted its marketing approach to harvest greater value from these opportunities. As a result, the business was able to gain meaningful market share and increase profitability through better fulfillment of customer requirements. Overall, the insights reveal a well-managed operation that drove powerful momentum through adept promotion execution aligned with customer lifecycles.


# Credits: 

Special Thanks to [Atharva Arya](https://www.kaggle.com/atharvaarya25) for the amazing Dataset from Kaggle: https://www.kaggle.com/datasets/atharvaarya25/e-commerce-analysis-uk



# Contact Information

You can connect with me and explore more about my professional journey through the following platforms:

[LinkedIn](https://www.linkedin.com/in/ninogarci/)
&nbsp;

[UpWork](https://www.upwork.com/freelancers/~01dd78612ac234aadd)

Feel free to reach out and connect. I'm excited to engage in meaningful discussions and collaborations!
