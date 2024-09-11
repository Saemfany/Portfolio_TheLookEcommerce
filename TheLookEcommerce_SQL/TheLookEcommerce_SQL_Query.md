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
-- 1. What is the total revenue received by theLook eCommerce?
-- 2. How many customers does theLook eCommerce have?
-- 3. How many orders does theLook eCommerce have?
-- 4. How many lost order opportunities does theLook eCommerce have?
SELECT
  ROUND(SUM(pendapatan)) AS total_pendapatan,
  COUNT(DISTINCT customer_id) AS total_customer,
  COUNT(DISTINCT order_id) AS total_order,
  COUNT(DISTINCT CASE WHEN status IN ('Cancelled', 'Returned') THEN order_id END) AS total_lost_order
FROM table1;

-- DEEP DIVE ANALYSIS
-- 1. What is the annual revenue trend for theLook eCommerce?
SELECT
  EXTRACT(YEAR FROM order_created_date) AS order_created_year,
  ROUND(SUM(pendapatan)) AS total_pendapatan
FROM table1
WHERE pendapatan IS NOT NULL 
GROUP BY order_created_year
ORDER BY order_created_year ASC;

-- 2. How is the performance of the traffic sources owned by theLook eCommerce?
SELECT
  traffic_source,
  ROUND(SUM(pendapatan)) AS total_pendapatan
FROM table1
GROUP BY traffic_source
ORDER BY total_pendapatan DESC;

-- 3. What are the top 5 product categories based on total orders in theLook eCommerce?
SELECT
  product_category,
  COUNT(DISTINCT order_id) AS total_order  
FROM table1
GROUP BY product_category
ORDER BY total_order DESC
LIMIT 5;

-- 4. What are the top 5 product name based on total orders in theLook eCommerce?
SELECT
  product_name,
  COUNT(DISTINCT order_id) AS total_order  
FROM table1
GROUP BY product_name
ORDER BY total_order DESC
LIMIT 5;

-- 5. What are the top 5 product brand based on total orders in theLook eCommerce?
SELECT
  product_brand,
  COUNT(DISTINCT order_id) AS total_order  
FROM table1
GROUP BY product_brand
ORDER BY total_order DESC
LIMIT 5;

-- 6. What are the top 5 countries based on total revenue for theLook eCommerce?
SELECT
  country,
  ROUND(SUM(pendapatan)) AS total_pendapatan 
FROM table1
GROUP BY country
ORDER BY total_pendapatan DESC
LIMIT 5;
```
