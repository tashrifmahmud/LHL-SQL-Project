# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
> Hello, this is **Tashrif Mahmud** and this is my first SQL project as part of my **Lighthouse Labs** datascience bootcamp curriculam! :computer:

The dataset we were given is a raw ecommerce dataset in .csv files. Five tables were created in PostgreSQL for further analysis. 
Details of it are given is **cleaning_data.md** file.

For answering each questions with queries, data was accordingly transformed, cleaned and processed. The question and answer along with all the SQL queries used are given in **starting_with_questions.md** and **starting_with_data.md** files.

The **QA.md** file contains all analysis and queries used for the QA process.

In this **README.md** file, some additional informations, queries and thought process is noted step by step.

## Process
### Step 1: Imported data into PostgreSQL
The database provided is in .csv format and thus it was first imported into PostgreSQL and a new database 'ecommerce' was created.
For all_sessions table used the following code:
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

### Step 2: The tables were populated using .csv files from the given dataset of ecommerce.
The PostgreSQL GUI was used for this step as running a query to import .csv file was showing error and could not be resolved.

### Step 3: The database was inspected table by table by using following queries:
```sql
-- For all tables
SELECT *
FROM all_sessions;
--
SELECT *
FROM analytics;
-- 
SELECT *
FROM products;
--
SELECT *
FROM sales_by_sku;
--
SELECT *
FROM sales_report;
```
### Step 3: Primary key 'sku' was identified in 'products' table and corresponding foreign key of 'sku' was discovered. 
```sql
-- Determining 'sku' as PK
SELECT 
COUNT(DISTINCT sku) AS unique_sku, 
COUNT(sku) AS total_sku
FROM products;
```
### Step 4: Other items in each table was tested for being PK. No other PK found.
Even without other PK and FK, columns with matching information set like userid can be used to JOIN tables.
```sql
--- using fullvisitorid to JOIN analytics and all_sessions table
SELECT *
FROM all_sessions alls
JOIN analytics ana
ON alls.fullvisitorid = ana.fullvisitorid
```
### Step 5: ERD was generated for viewing all tables and their content.
> Here is a link to the ERD : https://i.imgur.com/cEIncfj.png
> ![ERD](https://i.imgur.com/cEIncfj.png)
### Step 6: For the given five questions all relevant tables, columns and rows were checked and cleaned
Details of cleaning and QA can be found in **cleaning_data.md** and **QA.md** file
### Step 7: The five question answers are given by using results from SQL queries
Question and answers can be found in **starting_with_questions.md** file
### Step 8: Three more additional questions and answers were given
After exploring the database 3 more questions and their answers are given in **starting_with_data.md**


## Results

> The database houses 3 main categories of data, which are products, customers and overall ecommmerce company status, which are often found in different tables. 
* **Products:** For products we get data on sku, name, stock, ordered amount, price, product details like category and variant, restocking time, product review by sentiment score etc.
* **Customers:** For customers, we get data on their website visitid, visit and order date, time on site, searching behavior, purchased amount, purchased quantity, city and country from where they ordered etc.
* **Company:** For company, we can find out their revenue, sales data and overall various website performance statistics.

**For answering questions, we used selective data from each tables. Like from all_session table we took city and country names, product price. From analytics table we took revenue, units sold. From products table we took stock level, product name and ordered quantity. From sales_report table we took product sentiment score. We also joined tables together to get additional information like how much revenue each city or country has or what is the average sentiment score for each city and country etc.**

## Challenges 
The main challenge would be to identify all necessary data needed for answering a question and making sure the data is clean and it follows the quality assurance process. Once the data is clean and ready, it is easy to answer the questions. Another big challenge would be the amount of data that is missing from the dataset, for example, the city name is missing for the majority of the rows which can give misleading results.

## Future Goals
If I had more time then I would find more ways to use all the different kinds of data  which were available in the dataset to answer more questions. Also use different SQL query methods to write cleaner codes which is easily understood and more efficient.
