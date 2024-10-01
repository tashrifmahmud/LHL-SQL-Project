# **Risk Areas Identification and Description:**

* As we are dealing with city and country column in all_sessions table, it is found from QA process that city column has '(not set)' and 'not available' data, filtering them out poses a problem of reliability and accuracy. There are no alternative table to get 'city' column data from.

* As we use analytics table, most columns have NULL values and it needs to be filtered out. 

* Price and revenue needs to be adjusted by dividing it by 1,000,000.

* Units sold and sentiment score has negative values which needs to be filtered out for analysis.


## **QA Process:**
> Description of Quality Assurance process including the SQL queries used to execute it.

### Determining 'sku' as Primary Key in products table.
```sql
-- Determining 'sku' as PK
SELECT 
COUNT(DISTINCT sku) AS unique_sku, 
COUNT(sku) AS total_sku
FROM products;
```

### PK was further validated and explored using JOIN function.
```sql
-- Joining products and all_sessions table using PK and FK
SELECT *
FROM products pro
JOIN all_sessions alls
ON pro.sku = alls.productsku;
-- Joining sales_report and sales_by_sku tables using sku, both FK
SELECT *
FROM sales_report sr
JOIN sales_by_sku ss
ON sr.productsku = ss.productsku;
-- Joining products and sales_report tables using PK and FK
SELECT *
FROM products pro
JOIN sales_report sr
ON pro.sku = sr.productsku;
```

### While answering questions we must consider city column from all_sessions, the following query was run to understand the column
```sql
-- Go through all rows to identify potential issues in city column
SELECT*
FROM all_sessions;
-- To understand city in all_sessions
SELECT 
COUNT(city) AS total,  -- Total count of non-null cities
COUNT(DISTINCT city) AS unique_city,  -- Count of distinct cities
SUM(CASE WHEN city IS NULL THEN 1 ELSE 0 END) AS null_count,  -- Count of null cities
SUM(CASE WHEN city = '(not set)' THEN 1 ELSE 0 END) AS not_set,  -- Count of cities marked as '(not set)'
SUM(CASE WHEN city LIKE '%not available%' THEN 1 ELSE 0 END) AS not_available  -- Count of cities containing 'not available'
FROM all_sessions;
-- There are no null, but 354 not set and 8302 not available
```
As the number of city '(not set)' or not available is very high, filtering them out poses a problem of reliability and accuracy

### We must also consider country column from all_sessions, the following query was run to understand the column country
```sql
-- Go through all rows to identify potential issues in country column
SELECT*
FROM all_sessions;
-- To understand country in all_sessions
SELECT 
COUNT(country) AS total,  -- Total count of non-null country
COUNT(DISTINCT country) AS unique_city,  -- Count of distinct country
SUM(CASE WHEN country IS NULL THEN 1 ELSE 0 END) AS null_count,  -- Count of null country
SUM(CASE WHEN country = '(not set)' THEN 1 ELSE 0 END) AS not_set,  -- Count of country marked as '(not set)'
SUM(CASE WHEN country LIKE '%not available%' THEN 1 ELSE 0 END) AS not_available  -- Count of country containing 'not available'
FROM all_sessions;
-- There are no null, but 24 not set
-- We can ignore not set countries in further analysis safely
```
As the number of country '(not set)' is low, we can safely filter them out in out output

### The units_sold column in analytics have null and negative values
```sql
-- Check units_sold column in analytics
SELECT units_sold
FROM analytics;
-- has null values
SELECT MAX (units_sold)
FROM analytics;
-- shows 4324 max
SELECT MIN (units_sold)
FROM analytics;
-- shows '-89' which is not possible
-- units_sold cleaning
SELECT units_sold
FROM analytics
WHERE units_sold IS NOT NULL
AND units_sold >= 0;
```

### Productprice in all_session needs to be divided by 1,000,000 and there are no outliers found
```sql
-- Check productprice column in analytics
SELECT productprice
FROM all_sessions
WHERE productprice IS NULL;
-- has null values
SELECT MAX (productprice)
FROM all_sessions;
-- shows 298000000
SELECT MIN (productprice)
FROM all_sessions;
-- shows 0
-- productprice needs to be divided by 1,000,000
```

### In all_sessions table v2productcategory has '(not set)' values
```sql
-- v2productcategory column is checked
SELECT v2productcategory
FROM all_sessions;
--
SELECT v2productcategory
FROM all_sessions
WHERE v2productcategory IS NULL;
-- There are no NULL values
SELECT v2productcategory
FROM all_sessions
WHERE v2productcategory = '(not set)';
-- There are not set values
SELECT v2productcategory
FROM all_sessions
WHERE LOWER(v2productcategory) LIKE '%available%'
-- no such unavailable date
```

### In all_sessions table v2productname values are clean
```sql
-- v2productname column is checked
SELECT v2productname
FROM all_sessions;
--
SELECT v2productname
FROM all_sessions
WHERE v2productname IS NULL;
-- There are no NULL values
SELECT v2productname
FROM all_sessions
WHERE v2productname = '(not set)';
-- There are no 'not set' values
SELECT v2productname
FROM all_sessions
WHERE LOWER(v2productname) LIKE '%available%'
-- no such unavailable date
```

### Revenue column in analytics has no outliers but needs formatting
```sql
-- Check revenue column in analytics
SELECT revenue
FROM analytics;
-- has null values
SELECT MAX (revenue)
FROM analytics;
-- shows 6252750000
SELECT MIN (revenue)
FROM analytics;
-- shows 1123333
-- revenue formatting by dividing with 1,000,000 and rounded
-- NULL value is filtered out
SELECT ROUND((revenue/1000000), 2) AS revenue
FROM analytics
WHERE revenue IS NOT NULL;
```

### Sentiment_score column in products have NULL and negative values
```sql
-- Check sentimentscore column in products
SELECT sentimentscore
FROM products;
-- has null values
SELECT MAX (sentimentscore)
FROM products;
-- shows 1
SELECT MIN (sentimentscore)
FROM products;
-- shows -0.6, if its percentage value then it cannot be negative
-- sentimentscore formatting by removing NULL and negative values
-- NULL value is filtered out
SELECT sentimentscore
FROM products
WHERE sentimentscore IS NOT NULL
AND sentimentscore >= 0;
```

### Checking stocklevel and orderedquantity from products table, no QA issue found
```sql
SELECT stocklevel
FROM products;
-- for stocklevel
SELECT stocklevel
FROM products
WHERE stocklevel IS NULL;
-- no NULL values
SELECT MIN(stocklevel)
FROM products;
-- 0 is lowest with no negative values
SELECT MAX(stocklevel)
FROM products;
-- 19678 is maximum stock
SELECT orderedquantity
FROM products;
-- for orderedquantity
SELECT orderedquantity
FROM products
WHERE orderedquantity IS NULL;
-- no NULL values
SELECT MIN(orderedquantity)
FROM products;
-- 0 is lowest with no negative values
SELECT MAX(orderedquantity)
FROM products;
-- 15170 is maximum stock