```sql
-- Buat Temporary Table
create temp table table1 as
select
  u.id as customer_id,
  concat(u.first_name,' ',u.last_name) as customer_name,
  u.age,
  u.gender,
  u.country,
  u.traffic_source,
  o.order_id,
  o.created_at as order_created_date,
  o.status,  
  oi.product_id,  
  p.category as product_category,
  p.name as product_name,
  p.brand as product_brand,
  o.num_of_item,
  oi.sale_price,
  o.num_of_item * oi.sale_price as pendapatan
from
  bigquery-public-data.thelook_ecommerce.users u
left join bigquery-public-data.thelook_ecommerce.orders o on u.id = o.user_id
left join bigquery-public-data.thelook_ecommerce.order_items oi on u.id = oi.user_id
left join bigquery-public-data.thelook_ecommerce.products p on oi.product_id = p.id;

-- OVERALL PERFORMANCE
-- 1. Berapa total pendapatan TheLook Ecommerce yang diterima?
-- 2. Berapa jumlah customer yang dimiliki oleh TheLook Ecommerce?
-- 3. Berapa jumlah order yang dimiliki oleh TheLook Ecommerce?
-- 4. Berapa jumlah lost order opportunity yang dimliki TheLook Ecommerce?
select
  round(sum(pendapatan)) as total_pendapatan,
  count(distinct customer_id) as total_customer,
  count(distinct order_id) as total_order,
  count(distinct case when status in ('Cancelled', 'Returned') then order_id end) as total_lost_order
from table1;

-- DEEP DIVE ANALYSIS
-- 1. Bagaimana tren pendapatan per tahun yang dimiliki TheLook Ecommerce?
select
  extract(year from order_created_date) as order_created_year,
  round(sum(pendapatan)) as total_pendapatan
from table1
WHERE pendapatan IS NOT NULL  -- Menghilangkan baris dengan pendapatan NULL
group by order_created_year
order by order_created_year asc;

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
