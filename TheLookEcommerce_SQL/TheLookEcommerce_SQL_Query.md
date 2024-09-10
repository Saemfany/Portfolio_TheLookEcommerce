```sql
-- Create Temporary Table
CREATE TEMP TABLE table1 AS
SELECT
  u.id AS customer_id,
  CONCAT(u.first_name,' ',u.last_name) AS customer_name,
  u.age,
  u.gender,
  u.country,
  u.traffic_source,
  o.order_id,
  o.created_at AS order_created_date,
  o.status,  
  oi.product_id,  
  p.category AS product_category,
  p.name AS product_name,
  p.brand AS product_brand,
  o.num_of_item,
  oi.sale_price,
  o.num_of_item * oi.sale_price AS pendapatan
FROM
  bigquery-public-data.thelook_ecommerce.users u
LEFT JOIN bigquery-public-data.thelook_ecommerce.orders o ON u.id = o.user_id
LEFT JOIN bigquery-public-data.thelook_ecommerce.order_items oi ON u.id = oi.user_id
LEFT JOIN bigquery-public-data.thelook_ecommerce.products p ON oi.product_id = p.id;

-- OVERALL PERFORMANCE
-- 1. Berapa total pendapatan TheLook Ecommerce yang diterima?
-- 2. Berapa jumlah customer yang dimiliki oleh TheLook Ecommerce?
-- 3. Berapa jumlah order yang dimiliki oleh TheLook Ecommerce?
-- 4. Berapa jumlah lost order opportunity yang dimliki TheLook Ecommerce?
SELECT
  ROUND(SUM(pendapatan)) AS total_pendapatan,
  COUNT(DISTINCT customer_id) AS total_customer,
  COUNT(DISTINCT order_id) AS total_order,
  COUNT(DISTINCT CASE WHEN status IN ('Cancelled', 'Returned') THEN order_id END) AS total_lost_order
FROM table1;

-- DEEP DIVE ANALYSIS
-- 1. Bagaimana tren pendapatan per tahun yang dimiliki TheLook Ecommerce?
SELECT
  EXTRACT(YEAR FROM order_created_date) AS order_created_year,
  ROUND(SUM(pendapatan)) AS total_pendapatan
FROM table1
WHERE pendapatan IS NOT NULL 
GROUP BY order_created_year
ORDER BY order_created_year ASC;

-- 2. Bagaimana performance traffic source yang dimiliki oleh TheLook Ecommerce?
select
  traffic_source,
  round(sum(pendapatan)) as total_pendapatan
from table1
group by traffic_source
order by total_pendapatan desc;

-- 3. Apa top 5 product category berdasarkan total order di TheLook Ecommerce?
select
  product_category,
  count(distinct order_id) as total_order  
from table1
group by product_category
order by total_order desc
limit 5;

-- 4. Apa top 5 product name berdasarkan total order di TheLook Ecommerce?
select
  product_name,
  count(distinct order_id) as total_order  
from table1
group by product_name
order by total_order desc
limit 5;

-- 5. Apa top 5 product brand berdasarkan total order di TheLook Ecommerce?
select
  product_brand,
  count(distinct order_id) as total_order  
from table1
group by product_brand
order by total_order desc
limit 5;

-- 6. Apa top 5 negara berdasarkan total pendapatan TheLook Ecommerce?
select
  country,
  round(sum(pendapatan)) as total_pendapatan 
from table1
group by country
order by total_pendapatan desc
limit 5;
```
