# A. Enterprise Relationship Diagram
# B. Digital Analysis
### Using the available datasets - answer the following questions using a single query for each one:

###  1. How many users are there?
```sql 
SELECT COUNT(DISTINCT user_id) AS User_count FROM cs6.users
```
###  2. How many cookies does each user have on average?
```sql
with cte As (
SELECT  user_id , COUNT(cookie_id )  per_user_cookies FROM cs6.users GROUP BY user_id ORDER BY user_id)
SELECT ROUND(AVG(per_user_cookies),2) As avg_cookie_per_user FROM cte
```
###  3. What is the unique number of visits by all users per month?
```sql 
SELECT  EXTRACT(MONTH FROM event_time) AS Month_Number,
  TO_CHAR(event_time, 'Month') AS Month_Name , COUNT(DISTINCT visit_id) unique_visitd FROM cs6.events group by Month_Number, Month_Name
```
###  4. What is the number of events for each event type?
```sql 
SELECT e.event_type , ei.event_name, COUNT(*) FROM cs6.events AS e 
JOIN  cs6.event_identifier AS ei  ON   e.event_type = ei.event_type 
GROUP BY  e.event_type , ei.event_name
```
###  5. What is the percentage of visits which have a purchase event?
```sql 
SELECT ROUND(COUNT(DISTINCT visit_id)* 100.0 / (SELECT COUNT(DISTINCT visit_id) FROM  cs6.events e),2)  AS purchase_prcnt FROM cs6.events e JOIN cs6.event_identifier ei ON e.event_type=ei.event_type WHERE event_name='Purchase'
```
###  6. What is the percentage of visits which view the checkout page but do not have a purchase event?
```sql 
with cte as(
   select  distinct visit_id,
   sum(case when event_name!='Purchase'and page_id=12 then 1 else 0 end) as checkouts,
   sum(case when event_name='Purchase' then 1 else 0 end) as purchases
   from
   cs6.events e join cs6.event_identifier ei
   on e.event_type=ei.event_type
   group by visit_id
   )
   select sum(checkouts) as total_checkouts,sum(purchases) as total_purchases,
   100-round(sum(purchases)*100.0/sum(checkouts),2) as prcnt
   from cte
```
###  7. What are the top 3 pages by number of views?
```sql 
SELECT  ph.page_name , COUNT(visit_id) visit_count  FROM cs6.page_hierarchy AS ph  JOIN cs6.events  AS e ON ph.page_id = e.page_id group by  ph.page_name  order by visit_count DESC LIMIT 3 
```
###  8. What is the number of views and cart adds for each product category?
```sql 
SELECT ph.product_category, SUM(case when  event_name ='Page View' then 1 else 0 end ) as views,  SUM(case when  event_name ='Add to Cart' then 1 else 0 end ) as cartadd
FROM cs6.events AS e JOIN cs6.event_identifier AS ei ON  e.event_type = ei.event_type 
JOIN cs6.page_hierarchy AS ph ON e.page_id = ph.page_id WHERE product_category IS NOT null   GROUP BY product_category


```
###  9. What are the top 3 products by purchases?
```sql 
```
