
# SQL-Based Insights – Myntra Pants Sales

This report summarizes the key insights obtained using pure SQL on the `Myntra_DataCleaning.csv` dataset, loaded into a table named `myntra_pants_sales`.

---

## 1. Data Overview

- **Total number of records (products):** 35,073
- **Average Selling Price:** ₹1,618.15  
- **Average MRP:** ₹3,277.91  
- **Average Discount:** **46.84%** off MRP

This indicates that, on average, products in this category are sold at **almost half of their MRP**, suggesting a highly discount-driven market.

---

## 2. Brand Performance (by SQL Aggregations)

### Top Brands by Number of Products

From:

```sql
SELECT brand_name, COUNT(*) AS product_count
FROM myntra_pants_sales
GROUP BY brand_name
ORDER BY product_count DESC
LIMIT 10;
```

**Top 5 brands by product count:**

1. United Colors Of Benetton – 3,300 products  
2. Flying Machine – 2,576 products  
3. Roadster – 1,796 products  
4. Spykar – 1,149 products  
5. Wrogn – 1,101 products  

These brands dominate the assortment and should be treated as **key strategic partners**.

### Top Brands by Customer Rating (min 50 total ratings)

Using:

```sql
SELECT brand_name,
       ROUND(AVG(ratings),2) AS avg_rating,
       SUM(number_of_ratings) AS total_ratings
FROM myntra_pants_sales
GROUP BY brand_name
HAVING total_ratings >= 50
ORDER BY avg_rating DESC, total_ratings DESC
LIMIT 10;
```

This query highlights brands that combine **high satisfaction (rating)** with **meaningful engagement (rating volume)**. These are candidates for **premium placement and promotion**.

---

## 3. Discount Strategy Insights

Discounts were bucketed using:

```sql
WITH buckets AS (
    SELECT 
        CASE 
            WHEN discount_percent < 0.2 THEN '<20%'
            WHEN discount_percent < 0.4 THEN '20-40%'
            WHEN discount_percent < 0.6 THEN '40-60%'
            ELSE '60%+'
        END AS discount_bucket,
        price,
        ratings,
        number_of_ratings
    FROM myntra_pants_sales
)
SELECT 
    discount_bucket,
    COUNT(*) AS products,
    ROUND(AVG(price),2) AS avg_price,
    ROUND(AVG(ratings),2) AS avg_rating,
    SUM(number_of_ratings) AS total_ratings
FROM buckets
GROUP BY discount_bucket
ORDER BY 
    CASE discount_bucket 
        WHEN '<20%' THEN 1
        WHEN '20-40%' THEN 2
        WHEN '40-60%' THEN 3
        ELSE 4
    END;
```

**Results:**

| Discount Bucket | Products | Avg Price (₹) | Avg Rating | Total Ratings |
|-----------------|----------|---------------|------------|---------------|
| <20%            | 3786      | 1301.03        | 3.97       | 372338        |
| 20–40%          | 7107      | 2131.79        | 4.02       | 610852        |
| 40–60%          | 13904      | 1895.48        | 4.00       | 1248645        |
| 60%+            | 10276      | 1004.51        | 3.92       | 1405491        |

### Interpretation

- The **40–60% discount bucket** has the highest number of products with solid ratings (~4.0).  
- The **60%+ bucket** has **lower average prices** (₹1004.51) and slightly lower ratings (3.92), but **very high engagement** (1.4M+ ratings).  
- **Lower discount segments (<20%)** show relatively fewer products and lower engagement.

**Business takeaway:** Myntra’s customers are highly responsive to **heavy discounting**, especially in the 40–60% and 60%+ ranges. However, quality perception (ratings) is slightly better in mid discount ranges than in extreme ones.

---

## 4. Price Segment Analysis

Segments were created using:

```sql
WITH segments AS (
    SELECT 
        CASE 
            WHEN price < 1000 THEN 'Budget (<1000)'
            WHEN price < 2000 THEN 'Mid (1000-1999)'
            ELSE 'Premium (2000+)'
        END AS price_segment,
        price,
        ratings,
        number_of_ratings
    FROM myntra_pants_sales
)
SELECT 
    price_segment,
    COUNT(*) AS products,
    ROUND(AVG(price),2) AS avg_price,
    ROUND(AVG(ratings),2) AS avg_rating,
    SUM(number_of_ratings) AS total_ratings
FROM segments
GROUP BY price_segment
ORDER BY 
    CASE price_segment
        WHEN 'Budget (<1000)' THEN 1
        WHEN 'Mid (1000-1999)' THEN 2
        ELSE 3
    END;
```

**Results:**

| Price Segment       | Products | Avg Price (₹) | Avg Rating | Total Ratings |
|---------------------|----------|---------------|------------|---------------|
| Budget (<1000)      | 10816      | 708.09        | 3.88       | 1839195        |
| Mid (1000–1999)     | 17461      | 1516.88        | 4.01       | 1315477        |
| Premium (2000+)     | 6796      | 3326.75        | 4.04       | 482654        |

### Interpretation

- **Mid segment (₹1000–1999)** has the highest number of products (17,461) and strong ratings (~4.01).  
- **Budget segment** accounts for 10,816 products and the **highest engagement** (~1.84M ratings), but slightly lower average rating (3.88).  
- **Premium segment** has fewer products (6,796) but the **highest average rating** (4.04), indicating that customers who buy premium are generally more satisfied.

**Business takeaway:**  
- Focus **volume strategies** (discounts, visibility) on **Budget and Mid** segments.  
- Position **Premium** as a high-satisfaction, curated collection with selective promotions rather than deep discounting.

---

## 5. Best-Selling Products by Engagement

Using:

```sql
SELECT 
    brand_name,
    pants_description,
    price,
    MRP,
    ROUND(discount_percent * 100,2) AS discount_pct,
    ratings,
    number_of_ratings
FROM myntra_pants_sales
ORDER BY number_of_ratings DESC
LIMIT 10;
```

This query surfaces **hero SKUs** – items with very high rating counts. These are ideal for:
- Featuring in **home page banners**  
- Including in **recommendation carousels**  
- Using as **reference price anchors** during campaigns

---

## 6. Strategic Recommendations from SQL Analysis

1. **Discount Optimization**
   - Maintain strong focus on **40–60% discount** for scaling both sales and satisfaction.
   - Use **60%+** discounts sparingly for clearance, as ratings are slightly weaker.

2. **Assortment & Brand Strategy**
   - Prioritize brands like **United Colors Of Benetton, Flying Machine, Roadster, Spykar, Wrogn** for joint campaigns and exclusive launches.
   - Use high-rated brands (from rating-based SQL query) as **trust builders** in listings.

3. **Price Segment Targeting**
   - **Budget (<₹1000):** High engagement, good for new customer acquisition and festivals.  
   - **Mid (₹1000–1999):** Core revenue driver; ensure deep assortment and smart discounting.  
   - **Premium (₹2000+):** Smaller base but high satisfaction; ideal for premium branding and loyalty customers.


---

**All of the above insights are derived using only SQL queries on the cleaned dataset (`Myntra_DataCleaning.csv`).**
