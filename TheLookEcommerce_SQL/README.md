# THELOOK ECOMMERCE DATA ANALYSIS WITH SQL

### Table of Contents

[Introduction](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/main/TheLookEcommerce_SQL/README.md#introduction)

[Data Analysis](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/main/TheLookEcommerce_SQL/README.md#data-analysis)

* [Overall Performance](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/main/TheLookEcommerce_SQL/README.md#overall-performance)
* [Deep Dive Analysis](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/main/TheLookEcommerce_SQL/README.md#deep-dive-analysis)
  - [Annual Revenue Trend](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/main/TheLookEcommerce_SQL/README.md#1-what-is-the-annual-revenue-trend-for-thelook-ecommerce)
  - [Performance of the Traffic Source](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/main/TheLookEcommerce_SQL/README.md#2-how-is-the-performance-of-the-traffic-sources-owned-by-thelook-ecommerce)
  - [Top 5 Product Category](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/main/TheLookEcommerce_SQL/README.md#3-what-are-the-top-5-product-categories-based-on-total-orders-in-thelook-ecommerce)
  - [Top 5 Product Name](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/main/TheLookEcommerce_SQL/README.md#4-what-are-the-top-5-product-name-based-on-total-orders-in-thelook-ecommerce)
  - [Top 5 Product Brand](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/main/TheLookEcommerce_SQL/README.md#5-what-are-the-top-5-product-brand-based-on-total-orders-in-thelook-ecommerce)
  - [Top 5 Country](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/main/TheLookEcommerce_SQL/README.md#6-what-are-the-top-5-countries-based-on-total-revenue-for-thelook-ecommerce)

---

## Introduction

In this project, I conducted an in-depth analysis of the theLook eCommerce dataset using SQL queries. The objective was to answer specific business questions by extracting insights from the data. This document outlines the queries used, the logic behind them, and the conclusions drawn.

This project was carried out in [Google Cloud Console](https://console.cloud.google.com/) using [theLook eCommerce](https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce) dataset available in the BigQuery Public Data warehouse.

For the complete SQL code, it can be viewed [here](https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/d2bf884cad380429c49b97449e4e749d857b0fc8/TheLookEcommerce_SQL/TheLookEcommerce_SQL_Query.md).

For the graph, I used Ms. Excel.

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

### Overall Performance
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

### Deep Dive Analysis
#### 1. What is the annual revenue trend for theLook eCommerce?

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

#### 2. How is the performance of the traffic sources owned by theLook eCommerce?

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

#### 3. What are the top 5 product categories based on total orders in theLook eCommerce?
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

#### 4. What are the top 5 product name based on total orders in theLook eCommerce?
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
| product_name | total_order |
| :--- | ---: |
| Wrangler Men's Premium Performance Cowboy Cut Jean | 99 |
| Pearl iZUMi Attack Sock 3-Pack | 79 |
| Puma Men's Socks | 78 |
| Hanes Men's 4 Pack Boxer Brief | 77 |
| Fred Perry Men's Crew Neck Sweater | 70 |

**Graph:**

<img src="https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/f1a9609b0f8dfd9db181340292e2708eda774476/TheLookEcommerce_SQL/assets/fig_4_top_5_product_name.png" height="300" alt="Top 5 Product Name">

Here are the insights based on the top 5 products by total orders at theLook eCommerce:
1. **Wrangler Men's Premium Performance Cowboy Cut Jean** leads with **99 orders**, highlighting that jeans, particularly from well-known brands like Wrangler, are highly popular among customers. This reinforces the earlier insight that jeans are a top-selling category.
2. **Socks Are in High Demand**:
   - Both the **Pearl iZUMi Attack Sock 3-Pack** (79 orders) and **Puma Men's Socks** (78 orders) are among the top-selling items. This indicates a strong demand for essential, everyday items like socks, possibly because they are frequently repurchased or bought in bulk.
3. **Hanes Men's 4 Pack Boxer Brief**:
   - With **77 orders**, this product shows that **underwear and intimate wear** are also in high demand, aligning with the earlier insight about the "Intimates" category being one of the top product categories.
4. **Fred Perry Men's Crew Neck Sweater**:
   - This product, with **70 orders**, reflects customer interest in **premium casual wear**, suggesting that certain brands like Fred Perry resonate with customers seeking stylish but casual items.

**Key Takeaways:**
- **Essential and functional items** like jeans, socks, and underwear are the most popular products, likely due to their necessity and frequent usage.
- **Brand strength** plays a significant role, with well-established brands like Wrangler, Puma, and Fred Perry driving customer interest.
- **Comfort and practicality** seem to be driving customer preferences, with basics like socks and underwear being top purchases, alongside premium casual items like crewneck sweaters.

In conclusion, focusing on popular, high-demand essentials and maintaining strong relationships with trusted brands could help maximize future sales for theLook eCommerce.

#### 5. What are the top 5 product brand based on total orders in theLook eCommerce?
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
| product_brand | total_order |
| :--- | ---: |
| Allegra K | 11449 |
| Calvin Klein | 6603 |
| Carhartt | 4951 |
| Hanes | 3909 |
| Volcom | 3633 |

**Graph:**

<img src="https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/f1a9609b0f8dfd9db181340292e2708eda774476/TheLookEcommerce_SQL/assets/fig_5_top_5_product_brand.png" height="300" alt="Top 5 Product Brand">

Here are the insights from the top 5 product brands by total orders at theLook eCommerce:
1. **Allegra K Dominates**:
   - **Allegra K** leads significantly with **11,449 orders**, nearly double the total orders of the second-place brand, Calvin Klein. This suggests that Allegra K has a strong appeal, possibly due to affordability, wide product variety, or targeting a large customer segment.
2. **Calvin Klein's Strong Presence**:
   - **Calvin Klein** comes in second with **6,603 orders**, highlighting the brand’s popularity, likely due to its reputation for offering high-quality, stylish apparel. Calvin Klein’s strong brand equity makes it a preferred choice for many customers, especially for underwear, intimates, and fashion essentials.
3. **Carhartt's Niche Appeal**:
   - **Carhartt**, with **4,951 orders**, is a well-known brand for durable, workwear clothing. Its inclusion in the top brands suggests a dedicated customer base that values **functionality and durability**, likely appealing to customers seeking rugged, long-lasting apparel.
4. **Hanes as a Basic Essential**:
   - **Hanes** received **3,909 orders**, which aligns with earlier insights about the demand for **everyday essentials** like underwear and socks. Hanes' focus on providing affordable and basic items makes it a go-to brand for customers looking for comfort and practicality.
5. **Volcom and Casualwear**:
   - **Volcom** rounds out the top five with **3,633 orders**, reflecting a strong preference for **casual and lifestyle clothing**. Volcom's appeal may be tied to its connection to skate, surf, and streetwear cultures, catering to a specific audience interested in casual, active fashion.

**Key Takeaways:**
- **Allegra K's dominance** suggests it might offer a combination of affordability and style that attracts a broad customer base. The brand's wide product range may also contribute to its large order volume.
- **Brand loyalty and recognition** play a significant role, with iconic names like **Calvin Klein** and **Carhartt** maintaining strong positions due to their established reputations.
- **Essentials and casualwear** brands like **Hanes** and **Volcom** maintain steady demand, indicating customers' focus on comfort and everyday wear.

In conclusion, theLook eCommerce benefits from a diverse range of brands that cater to both **affordable fashion** and **premium, durable goods**, ensuring a balanced product offering that appeals to various customer segments.

#### 6. What are the top 5 countries based on total revenue for theLook eCommerce?
**Query:**
```sql
SELECT
  country,
  COUNT(DISTINCT pendapatan) AS total_pendapatan  
FROM table1
GROUP BY country
ORDER BY total_pendapatan DESC
LIMIT 5;
```

**Result:**
| country | total_pendapatan |
| :--- | ---: |
| China | 12790660.0 |
| United States | 8066352.0 |
| Brasil | 5250632.0 |
| South Korea | 1928882.0 |
| France | 1791401.0 |

**Graph:**

<img src="https://github.com/Saemfany/Portfolio_TheLookEcommerce/blob/f1a9609b0f8dfd9db181340292e2708eda774476/TheLookEcommerce_SQL/assets/fig_6_top_5_country.png" height="300" alt="Top 5 Country">

Here are the insights from the top 5 countries by total revenue for theLook eCommerce:
1. **China Leads the Market**:
   - With **$12.79 million** in total revenue, **China** is the largest contributor to theLook eCommerce's sales, indicating a strong presence and demand in this market. This could be driven by China's vast population, growing middle class, and increasing online shopping activity.
2. **United States as a Major Market**:
   - **The United States** comes in second with **$8.06 million** in revenue. This reflects the country's robust e-commerce landscape and the high purchasing power of American consumers, who may be attracted to a variety of product offerings, particularly in fashion and lifestyle categories.
3. **Brazil as a Growing E-commerce Hub**:
   - **Brazil** ranks third with **$5.25 million** in revenue, highlighting the country’s rising role in e-commerce. With a large population and increasing digital connectivity, Brazil represents an important and expanding market for online retailers.
4. **South Korea and France** as Niche but Significant Markets:
   - **South Korea** and **France** generate **$1.92 million** and **$1.79 million** in revenue, respectively. While smaller than China and the U.S., these countries show a strong affinity for theLook eCommerce’s products, possibly due to their high internet penetration rates and strong consumer interest in international brands.

**Key Takeaways:**
- **China and the U.S.** are the dominant revenue drivers, contributing the bulk of the income, signaling strong customer bases and effective market strategies in these regions.
- **Brazil’s growth** points to the importance of tapping into **emerging markets** with a large population and increasing access to e-commerce platforms.
- **South Korea and France** show potential for growth in more niche markets, where **brand appeal** and targeted marketing could further boost revenue.

In conclusion, theLook eCommerce is benefiting from a mix of **large and growing international markets**, with opportunities to scale further, particularly in countries like Brazil and emerging Asian markets.
