> **The following 5 questions have been answered and also provided the SQL queries used to find the answer.**

    
### **Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

```sql
-- To get city wise transaction revenue we run the following query
-- To get transcation_revenue we used SUM function on all_sessions.productprice
SELECT
city AS city_name,
country AS country_name,
-- ROUND function used for better readability
SUM(ROUND((productprice/1000000), 2)) AS transcation_revenue
FROM all_sessions
-- WHERE function used to clean missing data
WHERE city NOT IN ('(not set)', 'not available in demo dataset')
GROUP BY city, country
-- ORDER BY () DESC function used on transaction_revenue from highest to lowest to get all cities and countries
ORDER BY transcation_revenue DESC;
--
-- To get country wise total transaction revenue we run the following
-- Run the following query seperately
--
SELECT
country AS country_name,
SUM(ROUND((productprice/1000000), 2)) AS transcation_revenue
FROM all_sessions
WHERE country NOT IN ('(not set)', 'not available in demo dataset')
GROUP BY country
ORDER BY transcation_revenue DESC;
```


**Answer: Mountain View, New York and San Francisco of United States have the highest level of transaction revenues on the site. For countries United States, Canada and United Kingdom has the highest level of transaction revenue.**




### **Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:

```sql
-- First we find average number of products ordered from visitors in each city
-- Joined all_sessions and analytics table on fullvisitorid
-- Made the output a CTE
WITH all_sessions_analytics AS
(
SELECT *
FROM all_sessions alls
JOIN analytics ana
ON alls.fullvisitorid = ana.fullvisitorid
)
-- Average of units_sold
-- Grouped by city and country
-- Round the average till 2 decimal places
SELECT
city,
country,
ROUND (AVG(units_sold), 2) AS products_ordered
FROM all_sessions_analytics
WHERE city NOT IN ('(not set)', 'not available in demo dataset') 
AND units_sold IS NOT NULL
AND units_sold >= 0 
GROUP BY city, country
ORDER BY products_ordered DESC; 
-- Filtered output where city and units_sold is not empty or null
--
-- Then we find average number of products ordered from visitors in each country
--
WITH all_sessions_analytics AS
(
SELECT *
FROM all_sessions alls
JOIN analytics ana
ON alls.fullvisitorid = ana.fullvisitorid
)

SELECT
country,
ROUND (AVG(units_sold), 2) AS products_ordered
FROM all_sessions_analytics
WHERE units_sold IS NOT NULL
AND units_sold >= 0
GROUP BY country
ORDER BY products_ordered DESC; 
```


**Answer: The average number of products ordered from visitors in each city shows 64 output with San Bruno (US) 52.67, Mountain View (US) 16.17 and San Jose (US) 8.57. Then we run the second part of the SQL query and find that country shows 42 output, with United Stated 19.24, Czechia 15.18 and Mexico 1.83.**





### **Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

```sql
-- First we find the number of orders and product category for each city
-- Again we will use the same CTE function to make it easier
WITH all_sessions_analytics AS
(
SELECT *
FROM all_sessions alls
JOIN analytics ana
ON alls.fullvisitorid = ana.fullvisitorid
)
-- This time we will filter to ensure product category is not null or 'not set'
SELECT
city,
country,
SUM(units_sold) AS products_ordered,
v2productcategory AS product_category
FROM all_sessions_analytics
WHERE city NOT IN ('(not set)', 'not available in demo dataset') 
AND units_sold IS NOT NULL
AND v2productcategory IS NOT NULL
AND v2productcategory NOT IN ('(not set)')
GROUP BY city, country, v2productcategory
ORDER BY products_ordered DESC; 
-- Similarly we can run following second SQL query for getting country wise information

```



**Answer: We can see that in cities, Mountain View ordered 5807 from Home/Accesories/Sticker/ category, which is also the best selling category countrywise in United States. Home/Shop by Brand/YouTube/ comes in 2nd with 1488 orders in United States where 1397 orders were from San Bruno city. Most popular category in Hong Kong is Home/Electronics/Electronics Accessories/ and for Canada it is Apparel.**





### **Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

```sql
-- First we find the list of top selling products from each cities
-- Created a combined CTE for filtering by city from all_sessions and product_name from analytics
WITH city_rank AS
(
-- This CTE is for joining all_sessions and analytics table
WITH all_sessions_analytics AS
(
SELECT *
FROM all_sessions alls
JOIN analytics ana
ON alls.fullvisitorid = ana.fullvisitorid
)
SELECT
city,
v2productname AS product_name,
SUM(units_sold) AS total_sale,
-- We use ROW NUMBER() instead of RANK or DENSE_RANK to avoid duplicates while filtering later on
ROW_NUMBER() OVER (PARTITION BY city ORDER BY SUM(units_sold) DESC) AS total_sale_ranking
FROM all_sessions_analytics
-- We avoid missing or invalid data by filtering
WHERE units_sold IS NOT NULL
AND v2productname IS NOT NULL
AND v2productname NOT IN ('(not set)')
-- We use an additional filter for cities with unavailable data
AND city NOT IN ('(not set)', 'not available in demo dataset') 
GROUP BY city,
v2productname,
units_sold
ORDER BY total_sale DESC
)
-- We can validate our result with SELECT total_sale, total_sale_ranking
SELECT city, product_name
FROM city_rank
WHERE total_sale_ranking = 1;
-- We can see all unique city names with their best selling products in output
-- Then we can run a second SQL query to find the best selling products grouped by country
--
--Run the follwing query seperately
--
-- Created a combined CTE for filtering by country from all_sessions and product_name from analytics
WITH country_rank AS
(
-- This CTE is for joining all_sessions and analytics table
WITH all_sessions_analytics AS
(
SELECT *
FROM all_sessions alls
JOIN analytics ana
ON alls.fullvisitorid = ana.fullvisitorid
)
SELECT
country,
v2productname AS product_name,
SUM(units_sold) AS total_sale,
-- We use ROW NUMBER() instead of RANK or DENSE_RANK to avoid duplicates while filtering later on
ROW_NUMBER() OVER (PARTITION BY country ORDER BY SUM(units_sold) DESC) AS total_sale_ranking
FROM all_sessions_analytics
-- We avoid missing or invalid data by filtering
WHERE units_sold IS NOT NULL
AND v2productname IS NOT NULL
AND v2productname NOT IN ('(not set)')
GROUP BY country,
v2productname,
units_sold
ORDER BY total_sale DESC
)
-- We can validate our result with SELECT total_sale, total_sale_ranking
SELECT country, product_name
FROM country_rank
WHERE total_sale_ranking = 1;
-- WE can see the result in output
```



**Answer: Top Selling products by country shows, United States with 'Waze Baby on Board Window Decal', Czechia with 'Google Snapback Hat Black' and Hong Kong with 'Google Device Stand'. There are total 42 observations. Product from Google is the most obeserved top selling products accross all countries.**





### **Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

```sql
-- First we start with each cities and their revenue
WITH revenue AS
(
-- This CTE is for joining all_sessions, products and analytics table
WITH all_sessions_analytics_products AS
(
SELECT *
FROM all_sessions alls
JOIN analytics ana
ON alls.fullvisitorid = ana.fullvisitorid
JOIN products pro
ON alls.productsku = pro.sku
)
SELECT
city,
SUM((revenue/1000000)) AS total_revenue
FROM all_sessions_analytics_products
-- We avoid missing or invalid data by filtering
WHERE revenue IS NOT NULL
-- We use an additional filter for cities with unavailable data
AND city NOT IN ('(not set)', 'not available in demo dataset')
GROUP BY city, revenue
ORDER BY revenue DESC
)
-- We can validate our result with SELECT
SELECT  city, 
ROUND(SUM(total_revenue), 2) AS total_revenue_rounded
FROM revenue
GROUP BY city
ORDER BY total_revenue_rounded DESC;
-- Output shows revenue earned for 24 cities
-- Next, we start with country and their total revenue
--
-- Run the follwing query seperately
--
-- We will use the same CTE for easier approach
WITH revenue AS
(
-- This CTE is for joining all_sessions, products and analytics table
WITH all_sessions_analytics_products AS
(
SELECT *
FROM all_sessions alls
JOIN analytics ana
ON alls.fullvisitorid = ana.fullvisitorid
JOIN products pro
ON alls.productsku = pro.sku
)
SELECT
country,
SUM((revenue/1000000)) AS total_revenue
FROM all_sessions_analytics_products
-- We avoid missing or invalid data by filtering
WHERE revenue IS NOT NULL
-- We use an additional filter for country with unavailable data
AND country NOT IN ('(not set)', 'not available in demo dataset') 
GROUP BY country, revenue
ORDER BY revenue DESC
)
-- We can validate our result with SELECT
SELECT  country, 
ROUND(SUM(total_revenue), 2) AS total_revenue_rounded
FROM revenue
GROUP BY country
ORDER BY total_revenue_rounded DESC;
-- Output shows revenue earned for 5 countries, possibility of missing data
```

**Answer: For cities, US cities leads the revenue chart with Mountain View attaining highest revenue of 9799.52. San Bruno and New York trails behind it with 4103.99 and 3450.97 each. For countries, United States has the highest revenue of 93,925.81 which means consumers are mostly from US. Then there are customers from Canada with 229.95 and Germany with 69.98 revenue each.**







