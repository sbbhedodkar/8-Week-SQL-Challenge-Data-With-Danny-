## Data Cleansing Steps

**In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:**

-  Convert the week_date to a DATE format

-  Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc

-  Add a month_number with the calendar month for each week_date value as the 3rd column

-  Add a calendar_year column as the 4th column containing either 2018, 2019, or 2020 values

-  Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value

## Age Bands

| Segment    | Age Band        |
|------------|------------------|
| 1          | Young Adults     |
| 2          | Middle Aged      |
| 3 or 4     | Retirees         |

## Age Bands and Demographics

| Segment    | Age Band        | Demographic   |
|------------|------------------|---------------|
| 1          | Young Adults     | Couples       |
| 2          | Middle Aged      | Families      |
| 3 or 4     | Retirees         | Families      |
| unknown    | unknown          | unknown       |

### Handling Null Values

- Any null or empty string values in the original `segment`, `age_band`, or `demographic` columns are replaced with the string "unknown".
- The `demographic` column is determined by the first letter of the `segment` value:
  - `C` for Couples
  - `F` for Families
  - If the `segment` value does not start with `C` or `F`, it is considered as `unknown`.


- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record

    ```sql
    ALTER TABLE cs5.weekly_sales
    ALTER COLUMN week_date TYPE DATE USING week_date :: DATE;
    ```

    Add a week_number as the second column for each week_date value. For example, any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2, etc.

    ```sql
    SELECT week_date,
           DATE_PART('week', week_date) AS week_number,
           DATE_PART('month', week_date) AS month_number,
           DATE_PART('year', week_date) AS calendar_year,
           region, platform, segment,
           (CASE
               WHEN RIGHT(segment, 1) = '1' THEN 'Young Adults'
               WHEN RIGHT(segment, 1) = '2' THEN 'Middle Aged'
               WHEN RIGHT(segment, 1) IN ('3', '4') THEN 'Retirees'
               ELSE 'Unknown'
           END) AS age_band,
           (CASE
               WHEN LEFT(segment, 1) = 'C' THEN 'Couples'
               WHEN LEFT(segment, 1) = 'F' THEN 'Families'
               ELSE 'Unknown'
           END) AS demographic,
           customer_type, transactions, sales,
          ROUND(sales::NUMERIC/transactions, 2) AS avg_transaction
    FROM cs5.weekly_sales;
    ```

## Data Exploration

1. **What day of the week is used for each week_date value?**

    ```sql
    SELECT DISTINCT TO_CHAR(week_date, 'Day') AS first_day
    FROM cs5.weekly_sales;
    ```

2. **What range of week numbers are missing from the dataset?**
 ```sql
SELECT * FROM GENERATE_SERIES(1,53) AS missing_week_numbers
WHERE missing_week_numbers NOT IN (
  SELECT DISTINCT cs5.week_number FROM cs5.weekly_sales
) 
ORDER BY missing_week_numbers;
 ```
3. **How many total transactions were there for each year in the dataset?**

    ```sql
    SELECT DATE_PART('year', week_date) AS calendar_year,
           SUM(transactions) AS total_transactions
    FROM cs5.weekly_sales
    GROUP BY calendar_year
    ORDER BY calendar_year;
    ```

4. **What is the total sales for each region for each month?**

    ```sql
    SELECT DATE_PART('month', week_date) AS month_number,
           region,
           SUM(sales) AS total_sales
    FROM cs5.weekly_sales
    GROUP BY month_number, region
    ORDER BY month_number;
    ```

5. **What is the total count of transactions for each platform?**

    ```sql
    SELECT platform,
           SUM(transactions) AS total_transactions
    FROM cs5.weekly_sales
    GROUP BY platform;
    ```

6. **What is the percentage of sales for Retail vs Shopify for each month?**

    ```sql
    SELECT DATE_PART('year', week_date) AS year,
           TO_CHAR(week_date, 'Month') AS month_name,
           ROUND(SUM(CASE WHEN platform = 'Retail' THEN sales ELSE 0 END) * 100.0 / SUM(sales), 2) AS retail_per,
           ROUND(SUM(CASE WHEN platform = 'Shopify' THEN sales ELSE 0 END) * 100.0 / SUM(sales), 2) AS shopify_per
    FROM cs5.weekly_sales
    GROUP BY year, month_name
    ORDER BY year, month_name;
    ```

7. **What is the percentage of sales by demographic for each year in the dataset?**

    ```sql
    WITH cte AS (
        SELECT DATE_PART('year', week_date) AS year,
               (CASE
                   WHEN LEFT(segment, 1) = 'C' THEN 'Couples'
                   WHEN LEFT(segment, 1) = 'F' THEN 'Families'
                   ELSE 'Unknown'
               END) AS demographic,
               sales
        FROM cs5.weekly_sales
    )
    SELECT year,
           ROUND(SUM(CASE WHEN demographic = 'Couples' THEN sales ELSE 0 END) * 100.0 / SUM(sales), 2) AS couples_per,
           ROUND(SUM(CASE WHEN demographic = 'Families' THEN sales ELSE 0 END) * 100.0 / SUM(sales), 2) AS families_per,
           ROUND(SUM(CASE WHEN demographic = 'Unknown' THEN sales ELSE 0 END) * 100.0 / SUM(sales), 2) AS unknown_per
    FROM cte
    GROUP BY year
    ORDER BY year;
    ```


8. **Which age_band and demographic values contribute the most to Retail sales?**

```sql
SELECT age_band,
       demographic,
       ROUND(100*sum(sales)/
               (SELECT SUM(sales)
                FROM clean_weekly_sales
                WHERE platform="Retail"), 2) AS retail_sales_percentage
FROM cs5.weekly_sales
WHERE platform="Retail"
GROUP BY 1,
         2
ORDER BY 3 DESC;
``` 
	
9. **Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?**

Hence, we can not use avg_transaction column to find the average transaction size for each year and sales platform, because the result will be incorrect if we calculate average of an average to calculate the average.

```sql
SELECT calendar_year,
       platform,
       ROUND(SUM(sales)/SUM(transactions), 2) AS correct_avg,
       ROUND(AVG(avg_transaction), 2) AS incorrect_avg
FROM cs5.weekly_sales
GROUP BY 1,
         2
ORDER BY 1,
         2;
``` 
	
