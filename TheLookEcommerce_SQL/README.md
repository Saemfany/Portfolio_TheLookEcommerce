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
| 36721671.0 | 100000 | 124823 | 31152 |

## Deep Dive Analysis
### 1. What is the annual revenue trend for theLook eCommerce?

**Query:**
```sql
SELECT
  EXTRACT(YEAR FROM order_created_date) AS order_created_year,
  ROUND(SUM(pendapatan)) AS total_pendapatan
FROM table1
WHERE pendapatan IS NOT NULL
GROUP BY order_created_year
ORDER BY order_created_year asc;
```

**Result:**
| order_created_year | total_pendapatan |
| ---: | ---: |
| 2019 | 591760.0 |
| 2020 | 1969735.0 |
| 2021 | 3744144.0 |
| 2022 | 6172828.0 |
| 2023 | 10068376.0 |
| 2024 | 14174828.0 |

**Graph:**

<img src="https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/4ba2d9d2c3a5c5abdb4ef24c50bda883f3e086c6/TheLookEcommerce_SQL/assets/fig_1_total_revenue_per_year.png" width="400" alt="Total Revenue per Year">

From the data above, here are the key insights regarding the annual revenue trend for theLook eCommerce:
1. **Consistent Growth**: theLook eCommerce has experienced continuous revenue growth each year from 2019 to 2024. This indicates a healthy business with increasing sales and customer engagement.
2. **Significant Revenue Increase**:
   - The revenue **triples** from 2019 (591,760) to 2020 (1,969,735), indicating a major boost in sales or customer base during that year.
   - From 2020 to 2021, there is an almost **85% increase** in revenue, showing strong momentum.
   - The growth accelerates further between 2022 and 2023, with a **65% rise** in revenue.
   - Between 2023 and 2024, the revenue jumps by about **45%**, reaching over 14 million.
3. **Exponential Growth in Later Years**: From 2021 onward, the revenue starts to grow at a faster rate. This could be due to expanded operations, new product offerings, marketing strategies, or increased customer loyalty.
4. **Pandemic Influence**: The large jump from 2019 to 2020 might be partly due to the global shift toward online shopping during the COVID-19 pandemic, which many ecommerce platforms benefited from.
5. **Future Potential**: If the current growth trend continues, TheLook Ecommerce is poised for even higher revenue figures in the coming years. However, sustaining this growth would likely require continued innovation, customer acquisition strategies, and market expansion.

In conclusion, theLook eCommerce is experiencing robust and accelerating growth, particularly from 2020 onwards, and is well-positioned to continue capitalizing on the e-commerce boom.

### 2. How is the performance of the traffic sources owned by theLook eCommerce?

**Query:**
```sql
SELECT
  traffic_source,
  ROUND(SUM(pendapatan)) AS total_pendapatan
FROM table1
GROUP BY traffic_source
ORDER BY total_pendapatan desc;
```

**Result:**
| traffic_source | total_pendapatan |
| :--- | ---: |
| Search | 25805557.0 |
| Organic | 5453897.0 |
| Facebook | 2186340.0 |
| Email | 1781605.0 |
| Display | 1494272.0 |

**Graph:**

<img src="https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/d3336e2f0561c5bb42280a926770462e8d38b931/TheLookEcommerce_SQL/assets/fig_2_traffic_source.png" width="400" alt="Traffic Source">

Here are the key insights from the traffic source performance data for theLook eCommerce:
1. **Search as the Dominant Traffic Source**:
   - The **Search** traffic source contributes significantly more than any other channel, generating over **25.8 million** in revenue. This suggests that paid search ads (or organic search visibility) are highly effective for driving sales.
2. **Organic Traffic**:
   - **Organic traffic** generates around **5.45 million** in revenue, which indicates strong performance from non-paid search results. It suggests that TheLook Ecommerce has a good presence in search engines without relying entirely on paid ads.
3. **Social Media Impact**:
   - **Facebook** is the highest-performing social media source, contributing **2.18 million** in revenue. This suggests effective use of social media marketing strategies, although its impact is smaller compared to search and organic channels.
4. **Email Marketing**:
   - Email marketing contributes **1.78 million** in revenue. This shows that email campaigns still play a role in generating sales, although it's less impactful compared to other channels like search or Facebook.
5. **Display Ads**:
   - **Display ads** generate around **1.49 million** in revenue, showing that banner or other types of visual ads are the least effective among the traffic sources, though they still make a contribution.

**Key Takeaways:**
- **Search** is by far the most effective channel, followed by **Organic traffic**, demonstrating that both paid and organic search strategies are driving the majority of revenue.
- **Social media and email marketing** contribute moderately, but there may be opportunities to improve these channels for greater returns.
- **Display ads** are the lowest-performing, which may suggest reconsidering the budget or strategy for this type of advertising.

In summary, focusing on search strategies (both paid and organic) while exploring ways to boost social media and email marketing performance could help improve overall revenue generation.

### 3. What are the top 5 product categories based on total orders in TheLook Ecommerce?
**Query:**
```sql
SELECT
  product_category,
  COUNT(DISTINCT order_id) AS total_order  
FROM table1
GROUP BY product_category
ORDER BY total_order DESC
LIMIT 5;
```

**Result:**
| product_category | total_order |
| :--- | ---: |
| Jeans | 22837 |
| Tops & Tees | 21911 |
| Intimates | 21615 |
| Fashion Hoodies & Sweatshirts | 21505 |
| Shorts | 20746 |

**Graph:**

<img src="https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/39392397499a6856d5684f479380fbe1cf2b14ca/TheLookEcommerce_SQL/assets/fig_3_top_5_product_category.png" width="400" alt="Top 5 Product Category">

Here are the insights from the top 5 product categories based on total orders at theLook eCommerce:
1. **Jeans as the Top-Selling Category**:
   - **Jeans** lead in total orders with **22,837**, making it the most popular product category. This indicates a strong demand for jeans among customers, suggesting that it could be a staple or high-demand item in the store.
2. **Tops & Tees and Intimates**:
   - **Tops & Tees** (21,911) and **Intimates** (21,615) follow closely behind jeans. These categories also show high popularity, which reflects customer interest in everyday wear and essential clothing items.
3. **Fashion Hoodies & Sweatshirts**:
   - **Fashion Hoodies & Sweatshirts** rank fourth with **21,505** orders, showing that customers are interested in more casual and comfortable clothing styles. This might indicate trends in casual wear or seasonal demand (e.g., colder months).
4. **Shorts**:
   - **Shorts**, with **20,746** orders, round out the top five categories. While slightly lower in total orders compared to the others, this still reflects a strong preference for seasonal or casual clothing items.

**Key Takeaways:**
- **Basic and casual wear** items like jeans, tees, hoodies, and shorts dominate the top product categories, indicating customer preference for versatile, everyday clothing.
- The minimal difference in total orders across the top categories suggests consistent demand across these key clothing categories.
- **Fashion Hoodies & Sweatshirts** could indicate a trend toward comfort-focused fashion, possibly linked to remote work or lifestyle changes.

In conclusion, theLook eCommerce should continue focusing on these popular categories while also exploring new trends and seasonal demand to optimize sales.

### 4. What are the top 5 product name based on total orders in TheLook Ecommerce?
**Query:**
```sql
SELECT
  product_name,
  COUNT(DISTINCT order_id) AS total_order  
FROM table1
GROUP BY product_name
ORDER BY total_order DESC
LIMIT 5;
```

**Result:**
| Product Name | Total Order |
| :--- | ---: |
| Wrangler Men's Premium Performance Cowboy Cut Jean | 99 |
| Pearl iZUMi Attack Sock 3-Pack | 79 |
| Puma Men's Socks | 78 |
| Hanes Men's 4 Pack Boxer Brief | 77 |
| Fred Perry Men's Crew Neck Sweater | 70 |

**Graph:**


### 5. What are the top 5 product brand based on total orders in TheLook Ecommerce?
**Query:**
```sql
SELECT
  product_brand,
  COUNT(DISTINCT order_id) AS total_order  
FROM table1
GROUP BY product_brand
ORDER BY total_order DESC
LIMIT 5;
```

**Result:**
