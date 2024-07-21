# A. Data Exploration and Cleansing
###  1.Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month
```sql 
--Modify the length of column month_year so it can store 10 characters
ALTER TABLE cs8.interest_metrics
ALTER COLUMN month_year TYPE VARCHAR(10);


--Update values in month_year column
UPDATE cs8.interest_metrics
SET month_year =  CONVERT(DATE, '01-' + month_year, 105)

--Convert month_year to DATE
ALTER TABLE cs8.interest_metrics
ALTER COLUMN month_year DATE;

SELECT * FROM cs8.interest_metrics;
```

### 2 What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?
```sql 

```
