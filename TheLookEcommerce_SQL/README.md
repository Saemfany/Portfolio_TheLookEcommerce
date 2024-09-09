# THELOOK ECOMMERCE DATA ANALYSIS WITH SQL

## Introduction

In this project, I conducted an in-depth analysis of the theLook eCommerce dataset using SQL queries. The objective was to answer specific business questions by extracting insights from the data. This document outlines the queries used, the logic behind them, and the conclusions drawn.

This project was carried out in [Google Cloud Console](https://console.cloud.google.com/) using [theLook eCommerce](https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce) dataset available in the BigQuery Public Data warehouse.

For the complete SQL code, it can be viewed [here](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/d2bf884cad380429c49b97449e4e749d857b0fc8/TheLookEcommerce_SQL/TheLookEcommerce_SQL_Query.md).

---
## Data Analysis
First, create a temporary table in the query that contains the columns needed for this project. This temporary table will be used in query to answer business questions.
```sql
-- Buat Temporary Table
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
```

## Overall Performance
1. What is the total revenue received by theLook eCommerce?
2. How many customers does theLook eCommerce have?
3. How many orders does theLook eCommerce have?
4. How many lost order opportunities does theLook eCommerce have?

Query:
```sql
SELECT
  ROUND(SUM(pendapatan)) AS total_pendapatan,
  COUNT(DISTINCT customer_id) AS total_customer,
  COUNT(DISTINCT order_id) AS total_order,
  COUNT(DISTINCT CASE WHEN status IN ('Cancelled', 'Returned') THEN order_id END) AS total_lost_order
FROM table1;
```

Result:
| total_pendapatan | total_customer | total_order | total_lost_order |
| ---: | ---: | ---: | ---: |
| 36743548.0 | 100000 | 125225 | 31430 |

## Deep Dive Analysis
1. What is the annual revenue trend for theLook eCommerce?

Query:
```sql
SELECT
  EXTRACT(YEAR FROM order_created_date) AS order_created_year,
  ROUND(SUM(pendapatan)) AS total_pendapatan
FROM table1
WHERE pendapatan IS NOT NULL
GROUP BY order_created_year
ORDER BY order_created_year asc;
```

Result:
| order_created_year | total_pendapatan |
| ---: | ---: |
| 2019 | 587454.0 |
| 2020 | 1972233.0 |
| 2021 | 3658586.0 |
| 2022 | 6029152.0 |
| 2023 | 9932753.0 |
| 2024 | 14406299.0 |

Graph:
![Total Revenue per Year](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/64e356f5ab7966a3983948846e853778e7e83016/TheLookEcommerce_SQL/assets/fig_1%20total%20revenue%20per%20year.png)
