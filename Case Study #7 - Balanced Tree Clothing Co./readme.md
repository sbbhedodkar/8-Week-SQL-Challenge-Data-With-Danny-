# A. High Level Sales Analysis
### 1. What was the total quantity sold for all products?
```sql
SELECT SUM(qty) as total_qty_sold FROM cs7.sales s JOIN cs7.product_details p ON s.prod_id=p.product_id
```
### 2. What is the total generated revenue for all products before discounts?
```sql
SELECT SUM(qty * price) AS total_sales FROM cs7.sales
 ```
### 3. What was the total discount amount for all products?
```sql
SELECT ROUND(SUM((discount*(qty*s.price)/100.0)),2) AS Total_Discount
FROM cs7.sales s JOIN cs7.product_details pd
ON s.prod_id=pd.product_id
```
# B. Transaction Analysis
### 1. How many unique transactions were there?
```sql 
select count(distinct txn_id) as unique_transactions from cs7.sales
```
### 2. What is the average unique product purchased in each transaction?
```sql 
WITH CTE AS( SELECT txn_id, COUNT(DISTINCT prod_id) As uniq_prod_per_txn FROM  cs7.sales GROUP BY txn_id)
SELECT ROUND(AVG(uniq_prod_per_txn),2) AS avg_prod FROM CTE
```

### 3. What are the 25th, 50th, and 75th percentile values for the revenue per transaction?
```sql 
select  distinct
percentile_cont(0.25)within group(order by ((qty*price)*(1-discount*0.01)))over() as percentile_25,
percentile_cont(0.5)within group(order by ((qty*price)*(1-discount*0.01)))over() as percentile_50,
percentile_cont(0.75)within group(order by ((qty*price)*(1-discount*0.01)))over() as percentile_75
from cs7.sales
```
### 4. What is the average discount value per transaction?
```sql 
select round(avg(discount*qty*price/100.0),2) as avg_discount from cs7.sales
```
### 5. What is the percentage split of all transactions for members vs non-members?
```sql
select sum(case when member='t' then 1 else 0 end)*100.0/count(*) as member,
sum(case when member='f' then 1 else 0 end)*100.0/count(*) as non_member
from sales
```

### 6. What is the average revenue for member transactions and non-member transactions?
```sql
select 
avg(case when member='true' then (qty*price)*(1-discount*0.01) end) as avg_revenue_member,
avg(case when member='false' then (qty*price)*(1-discount*0.01) end) as avg_revenue_non_member
from cs7.sales
```

# C. Product Analysis
### 1. What are the top 3 products by total revenue before discount?
```sql 
SELECT p.product_name, SUM(s.qty * s.price) as total_rev
FROM cs7.sales As s JOIN cs7.product_details  As p ON s.prod_id = p.product_id
GROUP BY  p.product_name
ORDER BY total_rev DESC LIMIT 3
```
### 2. What is the total quantity, revenue and discount for each segment?
```sql
SELECT p.segment_name,
sum(qty) AS total_quantity,
round(sum((qty*s.price)*(1-discount*0.01)),2) AS total_rev,
round(sum(discount*qty*s.price/100.0),2) as total_disc									
FROM cs7.sales As s JOIN cs7.product_details  As p ON s.prod_id = p.product_id
GROUP BY  p.segment_name
```

### 3. What is the top selling product for each segment?

```sql
WITH cte AS (
SELECT p.segment_name, p.product_name, sum(qty) AS total_quantity,
RANK() OVER(PARTITION BY segment_name order by sum(qty) desc )AS rnk						
FROM cs7.sales As s JOIN cs7.product_details  As p ON s.prod_id = p.product_id
GROUP BY  p.segment_name , p.product_name )
SELECT segment_name,product_name,total_quantity FROM cte WHERE rnk = 1
```


### 4. What is the total quantity, revenue, and discount for each category?
```sql
SELECT p.category_name, sum(qty) AS total_quantity,
round(sum((qty*s.price)*(1-discount*0.01)),2) AS total_rev,
round(sum(discount*qty*s.price/100.0),2) as total_disc						
FROM cs7.sales As s JOIN cs7.product_details  As p ON s.prod_id = p.product_id
GROUP BY  p.category_name
```

### 5. What is the top-selling product for each category?
```sql
with cte as(
select category_name, product_name,sum(qty) as total_qty,rank()over(partition by category_name order by sum(qty) desc) as rk
from cs7.sales s
join cs7.product_details p on s.prod_id = p.product_id
group by category_name,product_name
)
select category_name, product_name, total_qty
from cte
where rk=1
```
### 6. What is the percentage split of revenue by product for each segment?
```sql 
with cte as (
select segment_name,product_name,sum((qty*s.price)*(1-discount*0.01)) as rev_prod from cs7.sales s
join cs7.product_details pd on s.prod_id = pd.product_id
group by segment_name,product_name
)
select segment_name,product_name,round(rev_prod*100.0/ (select sum((qty*price)*(1-discount*0.01)) from cs7.sales),2) as rev_prcnt from cte 
order by segment_name,rev_prod
```

### 7. What is the percentage split of revenue by segment for each category?
```sql 
with cte as 
(select category_name, segment_name,sum((qty*s.price)*(1-discount*0.01)) as rev_seg from cs7.sales s
join cs7.product_details pd on s.prod_id = pd.product_id
group by category_name,segment_name
)
select category_name,segment_name, round(rev_seg*100.0/ (select sum((qty*price)*(1-discount*0.01)) from sales),2) as rev_seg_prcnt
from cte order by category_name, rev_seg_prcnt
```

### 8. What is the percentage split of total revenue by category?
```sql
select category_name,
round(sum((qty*s.price)*(1-discount*0.01)) *100.0/ (select sum((qty*price)*(1-discount*0.01)) from sales),2) 
as rev_cat_prcnt
from cs7.sales s
join cs7.product_details pd on s.prod_id = pd.product_id
group by category_name
```

### 9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by the total number of transactions)

### 10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?


