SELECT * FROM cs5.weekly_sales

-- 1. Data Cleansing Steps

-- Convert the week_date to a DATE format
ALTER TABLE cs5.weekly_sales ALTER COLUMN week_date TYPE DATE USING week_date::DATE;

-- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc

SELECT week_date,
 DATE_PART('week', week_date) AS week_number,
 DATE_PART('month', week_date) AS month_number,
 DATE_PART('year', week_date) AS calendar_year,
 region, platform, segment,
 (CASE
     WHEN RIGHT(segment, 1) = '1' THEN 'Young Adults'
     WHEN RIGHT(segment, 1) = '2' THEN 'Middle Aged'
     WHEN RIGHT(segment, 1) IN ('3', '4') THEN 'Retirees'
     ELSE 'Unknown' END) AS age_band, 
 (CASE
     WHEN LEFT(segment, 1) = 'C' THEN 'Couples'
     WHEN LEFT(segment, 1) = 'F' THEN 'Families'
     ELSE 'Unknown' END) AS demographic, customer_type, transactions,
	 sales, ROUND (sales/transactions, 2) AS avg_transaction FROM cs5.weekly_sales


-- 2. Data Exploration
-- 1. What day of the week is used for each week_date value?
SELECT DISTINCT TO_CHAR(week_date, 'Day') AS first_day FROM cs5.weekly_sales;

-- 2. What range of week numbers are missing from the dataset?

-- 3. How many total transactions were there for each year in the dataset?
SELECT DATE_PART('year', week_date) AS calendar_year , SUM(transactions) AS total_transactions FROM cs5.weekly_sales group  by calendar_year order by calendar_year

-- 4. What is the total sales for each region for each month?
SELECT  DATE_PART('month', week_date) AS month_number, region , SUM(sales) AS total_transactions FROM cs5.weekly_sales group by month_number, region order by month_number

-- 5. What is the total count of transactions for each platform?
SELECT platform, SUM(transactions) AS total_transactions FROM cs5.weekly_sales GROUP BY platform;

-- 6. What is the percentage of sales for Retail vs Shopify for each month?

SELECT  DATE_PART('year', week_date) AS year, TO_CHAR(week_date, 'Month') month_name, 
 ROUND (SUM(CASE WHEN platform = 'Retail' THEN sales ELSE 0 END) * 100.0 / SUM(sales) ,2) AS retail_per,  
 ROUND (SUM(CASE WHEN platform = 'Shopify' THEN sales ELSE 0 END) * 100.0 / SUM(sales) ,2) AS shopify_per
FROM cs5.weekly_sales GROUP BY
year, month_name ORDER BY year, month_name

-- 7. What is the percentage of sales by demographic for each year in the dataset?
WITH cte AS (
SELECT DATE_PART('year', week_date) AS year, region,
(CASE WHEN LEFT(segment, 1) = 'C' THEN 'Couples'
      WHEN LEFT(segment, 1) = 'F' THEN 'Families'
      ELSE 'Unknown' END) AS demographic, sales FROM  cs5.weekly_sales )

SELECT year, 
ROUND(SUM(CASE WHEN demographic = 'Couples' THEN sales ELSE 0 END) * 100.0 / SUM(sales),2) AS Couples_per,
ROUND(SUM(CASE WHEN demographic = 'Families' THEN sales ELSE 0 END) * 100.0 / SUM(sales),2) AS fam_per,
ROUND(SUM(CASE WHEN demographic = 'Unknown' THEN sales ELSE 0 END) * 100.0 / SUM(sales),2) AS unknown_per
from cte GROUP BY year order by year

-- 8 
-- 9
