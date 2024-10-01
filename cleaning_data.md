# Issues which will be addressed by cleaning the data:
* Make code writing and reading experience better
* Assign data types
* Ensure data falls in specific range, like units_sold in analytics cannot be negative
* Ensure null, not set or unavailable rows are not counted or shown
* Round up numbers or values accordingly, like price needs to be divided by 1,000,000
* Filter output according to question's needs and answer recquirements



## **Queries:**
Here some of the SQL queries are given to clean the data along with the purpose and use.

### Made the column names all lowercase while creating tables for better code writing and reading and added data type accordingly.
Example with table all_sessions is given below:
```sql
CREATE TABLE public.all_sessions
(
    fullvisitorid numeric,
    channelgrouping character varying(15),
    "time" text,
    country character varying(20),
    city character varying(30),
    totaltransaction numeric,
    transactions integer,
    timeonsite integer,
    pageviews integer,
    sessionqualitydim integer,
    date date,
    visitid numeric,
    type character varying(5),
    productrefundamount numeric,
    productquantity integer,
    productprice numeric,
    productrevenue numeric,
    productsku character varying(20),
    v2productname text,
    v2productcategory text,
    productvariant character varying(20),
    currencycode character(3),
    itemquantity integer,
    itemrevenue integer,
    transactionrevenue numeric,
    transactionid character varying(15),
    pagetitle text,
    searchkeyword text,
    pagepathlevel1 text,
    ecommerceaction_type integer,
    ecommerceaction_step integer,
    ecommerceaction_option character varying(20)
);
ALTER TABLE IF EXISTS public.all_sessions
    OWNER to postgres;
```
For analytics table used the following code:
```sql
CREATE TABLE public.analytics
(
    visitnumber numeric,
    visitid numeric,
    visitstarttime text,
    date date,
    fullvisitorid numeric,
    userid text,
    channelgrouping character varying(15),
    socialengagementtype text
);

ALTER TABLE IF EXISTS public.analytics
    OWNER to postgres;
```
For products table used the following code:
```sql
CREATE TABLE public.products
(
    sku character varying(20),
    name text,
    orderedquantity integer,
    stocklevel integer,
    restockingleadtime integer,
    sentimentscore double precision,
    sentimentmagnitude double precision
);

ALTER TABLE IF EXISTS public.products
    OWNER to postgres;
```
For sales_by_sku table used the following code:
```sql
CREATE TABLE public.sales_by_sku
(
    productsku character varying(20),
    total_ordered integer,
);

ALTER TABLE IF EXISTS public.sales_by_sku
    OWNER to postgres;
```
For sales_report table used the following code:
```sql
CREATE TABLE public.sales_report
(
    productsku character varying(20),
    total_ordered integer,
    name text,
    stocklevel integer,
    restockingleadtime integer,
    sentimentscore double precision,
    sentimentmagnitude double precision,
    ratio double precision
);

ALTER TABLE IF EXISTS public.sales_report
    OWNER to postgres;
```
### Divided product_price with 1,000,000 as per guideline and also used ROUND function to for showing value after two decimal places
### City with '(not set)', 'not available in demo dataset' values have been filtered out
### Used ORDER BY () DESC function for better clarity in output
```sql
SELECT
city AS city_name,
country AS country_name,
SUM(ROUND((productprice/100000), 2)) AS transcation_revenue
FROM all_sessions
WHERE city NOT IN ('(not set)', 'not available in demo dataset')
GROUP BY city, country
ORDER BY transcation_revenue DESC;
```
### Used WHERE function to filter units_sold column with NULL and negative values
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

### In all_sessions table v2productcategory has '(not set)' values which is filtered out, column name changed with AS function
```sql
SELECT v2productcategory AS product_category
FROM all_sessions
WHERE v2productcategory IS NOT NULL
AND v2productcategory NOT IN ('(not set)')
```

### Similarly alias is given for v2productname column as product_name
```sql
SELECT v2productname AS product_name
FROM all_sessions
```

### Revenue column in analytics has no outliers but needs formatting like price
```sql
SELECT ROUND((revenue/1000000), 2) AS revenue
FROM analytics
WHERE revenue IS NOT NULL;
```
### Sentiment_score column in products have NULL and negative values which is filtered by WHERE
```sql
-- sentimentscore cleaning by removing NULL and negative values
-- NULL value is filtered out
SELECT sentimentscore
FROM products
WHERE sentimentscore IS NOT NULL
AND sentimentscore >= 0;
```