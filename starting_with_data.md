### **Question 1: What is the city wise average customer sentiment?**

**SQL Queries:**
```sql
-- Created a combined CTE for filtering by city, sentiment score and revenue
WITH revenue_sentiment AS
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
sentimentscore AS sentiment_score,
-- We divide revenue by 1000000 as per guideline
SUM((revenue/1000000)) AS total_revenue
FROM all_sessions_analytics_products
-- We avoid missing or invalid data by filtering
WHERE revenue IS NOT NULL
AND sentimentscore IS NOT NULL
AND sentimentscore >= 0
-- We use an additional filter for cities with unavailable data
AND city NOT IN ('(not set)', 'not available in demo dataset') 
GROUP BY city, revenue, sentimentscore
ORDER BY revenue DESC
)
-- We can validate our result with SELECT sentiment_score
SELECT  city, 
AVG(sentiment_score) AS sentiment_score,
ROUND(SUM(total_revenue), 2) AS total_revenue_rounded
FROM revenue_sentiment
GROUP BY city
ORDER BY total_revenue_rounded DESC;
-- We can see all unique city names with their total revenue and customer sentiment score
```
**Answer: Mountain view has the highest revenue and average customer score is .38, San Bruno has customer score of .45 and New york has customer score of .38.**



### **Question 2: Consider the following conditions for customer satisfaction and find which city falls in which category: score 0 to .3 is unsatisfied, score .4 to .6 is moderately satisfied and above .6 is satisfied.**

**SQL Queries:**
```sql
-- Created a combined CTE for filtering by city, sentiment score and revenue as before
WITH rev_sent AS
(
WITH revenue_sentiment AS
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
sentimentscore AS sentiment_score,
-- We divide revenue by 1000000 as per guideline
SUM((revenue/1000000)) AS total_revenue
FROM all_sessions_analytics_products
-- We avoid missing or invalid data by filtering
WHERE revenue IS NOT NULL
AND sentimentscore IS NOT NULL
AND sentimentscore >= 0
-- We use an additional filter for cities with unavailable data
AND city NOT IN ('(not set)', 'not available in demo dataset') 
GROUP BY city, revenue, sentimentscore
ORDER BY revenue DESC
)
-- We can validate our result with SELECT sentiment_score
SELECT  city, 
AVG(sentiment_score) AS sentiment_score,
ROUND(SUM(total_revenue), 2) AS total_revenue_rounded
FROM revenue_sentiment
GROUP BY city
ORDER BY total_revenue_rounded DESC
)
-- Now we can assign values for sentiment score given the condition parameters
SELECT city, total_revenue_rounded AS total_revenue,
CASE 
WHEN sentiment_score <= 0.399 THEN 'unsatisfied'
WHEN sentiment_score BETWEEN 0.4 AND 0.699 THEN 'moderately satisfied'
WHEN sentiment_score >= 0.7 THEN 'moderately satisfied'
ELSE 'satisfied'
END AS customer_sentiment
FROM rev_sent
-- We can see all unique city names with their total revenue and customer sentiment categories
```
**Answer: As per categories, the highest revenue earning Mountain View city, as well as San Bruno and New York has moderately satisfied customers. Chicago, Austin, Los Angeles has unsatisfied customers. Average customer satisfaction is very low.**



### **Question 3: Are there any products where order quantity is larger than the current stock level? If so then make a list along with restock priority assigned to each products, with priority level 1 being the highest.**

**SQL Queries:**
```sql
-- Consider products table for this date
SELECT orderedquantity, products.name as product_name, stocklevel,
-- Rank them in order to find restock priority level
RANK() OVER (ORDER BY (orderedquantity-stocklevel) DESC) AS restock_priority_level
FROM products
-- Select condition where stock level is lower than ordered quantity
WHERE orderedquantity > stocklevel
GROUP BY products.orderedquantity, products.name, stocklevel
```

**Answer: There are 15 products with more orders than existing stock of them. Kick Ball has the highest ordered quantity with 15,170 units but there are only 723 in stock, making it level 1 priority. Metallic Notebook Set and Maze Pen follows with 2,718 and 1,748 orders each and 610 and 324 stocks.**

