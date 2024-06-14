## A. Customer Nodes Exploration
### 1. How many unique nodes are there on the Data Bank system?
```sql
SELECT COUNT( DISTINCT node_id) as unique_nodes
FROM customer_nodes
```
### 2. What is the number of nodes per region?
```sql 
SELECT c.region_id,region_name, count(node_id) as total_nodes
FROM cs4.customer_nodes c 
JOIN cs4.regions r ON c.region_id = r.region_id
GROUP BY c.region_id,region_name
ORDER BY c.region_id
```

### 3. How many customers are allocated to each region?
```sql
SELECT c.region_id, region_name, COUNT(distinct customer_id) as total_customers
FROM cs4.customer_nodes c 
JOIN cs4.regions r ON c.region_id = r.region_id
GROUP BY c.region_id, region_name
ORDER BY c.region_id
```
## 4. How many days on average are customers reallocated to a different node?
```sql 
SELECT AVG(start_date - end_date)
FROM cs4.customer_nodes
WHERE end_date != '9999-12-31';
```

### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
```sql
WITH CTE AS (
    SELECT region_id,
           (end_date - start_date) AS allocation_days
    FROM cs4.customer_nodes
    WHERE end_date != '9999-12-31'
)
SELECT DISTINCT region_id,
       PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY allocation_days) OVER (PARTITION BY region_id) AS median,
       PERCENTILE_DISC(0.8) WITHIN GROUP (ORDER BY allocation_days) OVER (PARTITION BY region_id) AS 80th_percentile,
       PERCENTILE_DISC(0.95) WITHIN GROUP (ORDER BY allocation_days) OVER (PARTITION BY region_id) AS 95th_percentile
FROM CTE;
```
## B. Customer Transactions
### 1. What is the unique count and total amount for each transaction type?
```sql
SELECT  txn_type, 
        COUNT(*) as total_transaction,
        SUM(txn_amount) as total_amount
FROM cs4.customer_transactions
GROUP BY txn_type;
```

### 2. What is the average total historical deposit counts and amounts for all customers?
```sql
WITH DEPOSIT_CTE AS (
SELECT  customer_id, COUNT(customer_id) as time_deposit, AVG(txn_amount) as amount_deposit
FROM cs4.customer_transactions 
 WHERE txn_type = 'deposit'
 GROUP BY customer_id
 ) 
SELECT AVG(time_deposit) AS avg_count, AVG(amount_deposit) AS avg_amount
FROM DEPOSIT_CTE;
```
### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```sql
WITH CTE AS (
SELECT customer_id, 
 datepart(month,txn_date) as month,
SUM(CASE WHEN txn_type ='deposit' then 1 else 0 end) as  deposit_time,
SUM(CASE WHEN txn_type ='purchase' then 1 else 0 end) as  purchase_time,
SUM(CASE WHEN txn_type ='withdrawal' then 1 else 0 end) as  withdrawal_time   
 FROM customer_transactions
GROUP BY customer_id,datepart(month,txn_date)
)
SELECT month, count(*)
FROM CTE 
WHERE deposit_time > 1 and (purchase_time =1 or withdrawal_time =1)
GROUP BY month;
```
### 4. What is the closing balance for each customer at the end of the month? Also show the change in balance each month in the same table output.
```sql
WITH CTE as (
SELECT customer_id, 
DATEPART(MONTH,txn_date) as month, 
SUM(CASE WHEN txn_type ='deposit' then txn_amount else 0 end) as deposit,
SUM(CASE WHEN txn_type ='purchase' then - txn_amount else 0 end) as purchase ,
SUM(CASE WHEN txn_type ='withdrawal' then - txn_amount else 0 end)  as  withdrawal             
from customer_transactions
GROUP BY customer_id,DATEPART(MONTH,txn_date)
),
CTE_2 AS (
SELECT customer_id,
month,(deposit +purchase +withdrawal) as total
from CTE)
SELECT customer_id, month,  
SUM(total) OVER (PARTITION BY customer_id ORDER BY customer_id,month  ROWS BETWEEN UNBOUNDED PRECEDING AND current ROW) AS balance, total AS change_in_balance 
FROM CTE_2;
```

### 5. What is the percentage of customers who increase their closing balance by more than 5%?

