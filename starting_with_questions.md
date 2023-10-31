Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
```
CREATE TEMPORARY TABLE trans_rev AS
SELECT DISTINCT	country,
		city,
		totaltransactionrevenue,
		transactionrevenue,
		transactions,
		productprice,
		productrevenue,
		productsku,
		v2productname,
		v2productcategory
FROM all_sessions

-- clean table
UPDATE trans_rev
SET productprice = 
	CASE
		WHEN productprice > 1000000 THEN (productprice / 1000000)::NUMERIC(10,2)
		ELSE productprice
	END
	
UPDATE trans_rev
SET totaltransactionrevenue = 
	CASE
		WHEN totaltransactionrevenue IS NULL THEN NULL
		ELSE (totaltransactionrevenue / 1000000)::NUMERIC(10,2)
	END

UPDATE trans_rev
SET transactionrevenue = 
	CASE
		WHEN transactionrevenue IS NULL THEN NULL
		ELSE (transactionrevenue / 1000000)::NUMERIC(10,2)
	END
	
UPDATE trans_rev
SET productrevenue = 
	CASE
		WHEN productrevenue IS NULL THEN NULL
		ELSE (productrevenue / 1000000)::NUMERIC(10,2)
	END
	
UPDATE trans_rev
SET city = 
	CASE
		WHEN city = '(not set)' THEN 'N/A'
		WHEN city = 'not available in demo dataset' THEN 'N/A'
		ELSE city
	END
	
UPDATE trans_rev
SET country = 
	CASE
		WHEN country = '(not set)' THEN 'N/A'
		ELSE country
	END

-- 1. country with highest level of transaction revenue
SELECT country, SUM(totaltransactionrevenue) AS totaltransaction
FROM trans_rev 
WHERE productprice > 0.01 AND totaltransactionrevenue IS NOT NULL
GROUP BY country
ORDER BY totaltransaction DESC

-- city with highest level of transaction revenue
SELECT city, SUM(totaltransactionrevenue) AS totaltransaction
FROM trans_rev 
WHERE productprice > 0.01 AND totaltransactionrevenue IS NOT NULL and city <> 'N/A'
GROUP BY city
ORDER BY totaltransaction DESC
```

Answer:
<img width="288" alt="Screen Shot 2023-10-30 at 6 21 23 PM" src="https://github.com/issajojoke/SQL-project/assets/140217218/c261a320-f607-4c1e-8782-ddb90be89293">

<img width="287" alt="Screen Shot 2023-10-30 at 6 22 01 PM" src="https://github.com/issajojoke/SQL-project/assets/140217218/f62cc40f-09bc-4ce6-8950-156158f65832">


**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
```
CREATE TEMPORARY TABLE trans_rev AS
SELECT DISTINCT	country,
		city,
		totaltransactionrevenue,
		transactionrevenue,
		transactions,
		productprice,
		productrevenue,
		productsku,
		v2productname,
		v2productcategory
FROM all_sessions

-- clean table
UPDATE trans_rev
SET productprice = 
	CASE
		WHEN productprice > 1000000 THEN (productprice / 1000000)::NUMERIC(10,2)
		ELSE productprice
	END
	
UPDATE trans_rev
SET totaltransactionrevenue = 
	CASE
		WHEN totaltransactionrevenue IS NULL THEN NULL
		ELSE (totaltransactionrevenue / 1000000)::NUMERIC(10,2)
	END
	
UPDATE trans_rev
SET transactionrevenue = 
	CASE
		WHEN transactionrevenue IS NULL THEN NULL
		ELSE (transactionrevenue / 1000000)::NUMERIC(10,2)
	END
	
UPDATE trans_rev
SET productrevenue = 
	CASE
		WHEN productrevenue IS NULL THEN NULL
		ELSE (productrevenue / 1000000)::NUMERIC(10,2)
	END
	
UPDATE trans_rev
SET city = 
	CASE
		WHEN city = '(not set)' THEN 'N/A'
		WHEN city = 'not available in demo dataset' THEN 'N/A'
		ELSE city
	END
	
UPDATE trans_rev
SET country = 
	CASE
		WHEN country = '(not set)' THEN 'N/A'
		ELSE country
	END

CREATE TEMPORARY TABLE ordered AS
SELECT DISTINCT	sku,
		totalordered,
		name
FROM sales_report 

ALTER TABLE ordered
ALTER COLUMN totalordered TYPE numeric

-- avg number of products ordered from visitors in each country
SELECT tr.country, round(AVG(o.totalordered),2) as avg_orders
FROM trans_rev tr
FULL JOIN (
	SELECT sku, totalordered
	FROM ordered) o 
	ON tr.productsku = o.sku
WHERE country IS NOT NULL
GROUP BY tr.country
HAVING round(AVG(o.totalordered),2) IS NOT NULL and round(AVG(o.totalordered),2) <> '0.00'

-- avg number of products ordered from visitors in each city
SELECT tr.city, round(AVG(o.totalordered),2) as avg_orders
FROM trans_rev tr
FULL JOIN (
	SELECT sku, totalordered
	FROM ordered) o 
	ON tr.productsku = o.sku
WHERE city IS NOT NULL
GROUP BY tr.city
HAVING round(AVG(o.totalordered),2) IS NOT NULL and round(AVG(o.totalordered),2) <> '0.00'
```

Answer:
<img width="275" alt="Screen Shot 2023-10-30 at 6 26 26 PM" src="https://github.com/issajojoke/SQL-project/assets/140217218/79aa8092-8234-4ef8-943d-57737787c1ca">
<img width="279" alt="Screen Shot 2023-10-30 at 6 27 31 PM" src="https://github.com/issajojoke/SQL-project/assets/140217218/088fbe98-788a-466d-acb1-13e9204306f2">



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
```
CREATE TEMPORARY TABLE prod_cat AS
SELECT DISTINCT	country,
				city,
				v2productname as productname,
				v2productcategory as productcategory
FROM all_sessions

CREATE TEMPORARY TABLE prod_ord AS
SELECT DISTINCT	totalordered,
				name as productname
FROM sales_report

-- clean table
UPDATE prod_cat
SET productcategory = TRIM(TRAILING '/' FROM productcategory)

UPDATE prod_cat
SET city = 
	CASE
		WHEN city = '(not set)' THEN 'N/A'
		WHEN city = 'not available in demo dataset' THEN 'N/A'
		ELSE city
	END
	
UPDATE prod_cat
SET country = 
	CASE
		WHEN country = '(not set)' THEN 'N/A'
		ELSE country
	END

ALTER TABLE prod_ord
ALTER COLUMN totalordered TYPE NUMERIC

-- to only get the string after the first '/' learned this via postgresltutorial
UPDATE prod_cat
SET productcategory = SPLIT_PART(productcategory, '/', 2);

-- to get country
WITH prod_cat_ord AS (
	SELECT	pc.country, pc.city, pc.productname, pc.productcategory, SUM(po.totalordered) as total
	FROM prod_cat pc
	FULL JOIN prod_ord po
	ON pc.productname = po.productname
	WHERE country <> 'N/A' and po.productname IS NOT NULL
	GROUP BY pc.country, pc.city, pc.productname, pc.productcategory
	HAVING SUM(po.totalordered) IS NOT NULL and SUM(po.totalordered) <> '0')
SELECT p1.country, p1.productcategory, SUM(p1.total) as total
FROM prod_cat_ord p1
WHERE p1.total = (
	-- subquery to ensure max total matches outer query using country
	SELECT	MAX(p2.total)
			FROM prod_cat_ord p2
			WHERE p1.country = p2.country)
GROUP BY p1.country, p1.productcategory

-- to get city 
WITH prod_cat_ord AS (
	SELECT	pc.city, pc.country, pc.productname, pc.productcategory, SUM(po.totalordered) as total
	FROM prod_cat pc
	FULL JOIN prod_ord po
	ON pc.productname = po.productname
	WHERE city <> 'N/A' and po.productname IS NOT NULL
	GROUP BY pc.city, pc.country, pc.productname, pc.productcategory
	HAVING SUM(po.totalordered) IS NOT NULL and SUM(po.totalordered) <> '0')
SELECT p1.city, p1.productcategory, SUM(p1.total) as total
FROM prod_cat_ord p1
WHERE p1.total = (
	-- subquery to ensure max total matches outer query using city
	SELECT	MAX(p2.total)
			FROM prod_cat_ord p2
			WHERE p1.city = p2.city)
GROUP BY p1.city, p1.productcategory
```


Answer:



**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:
```
CREATE TEMPORARY TABLE products_sold AS
SELECT DISTINCT	country,
				city,
				v2productname as productname
FROM all_sessions 

CREATE TEMPORARY TABLE orders AS 
SELECT DISTINCT	totalordered,
				name as productname
FROM sales_report

UPDATE products_sold
SET city = 
	CASE
		WHEN city = '(not set)' THEN 'N/A'
		WHEN city = 'not available in demo dataset' THEN 'N/A'
		ELSE city
	END
	
UPDATE products_sold
SET country = 
	CASE
		WHEN country = '(not set)' THEN 'N/A'
		ELSE country
	END

-- create CTE for country
WITH top_prod AS (
	SELECT ps.country, ps.productname, SUM(o.totalordered) as totalordered
	FROM products_sold ps
	FULL JOIN orders o ON ps.productname = o.productname
	WHERE ps.country <> 'N/A'
	GROUP BY ps.country, ps.productname
	HAVING SUM(o.totalordered) IS NOT NULL)
SELECT tp.country, tp.productname, tp.totalordered
FROM top_prod tp
WHERE tp.totalordered = (
	-- subquery to ensure max totalordered matches outer query using country
	SELECT	MAX(bs.totalordered)
			FROM top_prod bs
			WHERE bs.country = tp.country)
			
-- create CTE for city
WITH top_prod AS (
	SELECT ps.city, ps.productname, SUM(o.totalordered) as totalordered
	FROM products_sold ps
	FULL JOIN orders o ON ps.productname = o.productname
	WHERE ps.city <> 'N/A'
	GROUP BY ps.city, ps.productname
	HAVING SUM(o.totalordered) IS NOT NULL)
SELECT tp.city, tp.productname, tp.totalordered
FROM top_prod tp
WHERE tp.totalordered = (
	-- subquery to ensure max totalordered matches outer query using city
	SELECT	MAX(bs.totalordered)
			FROM top_prod bs
			WHERE bs.city = tp.city)
```


Answer:



**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
```
CREATE TEMPORARY TABLE revenue AS
SELECT DISTINCT	country,
				city,
				fullvisitorid,
				totaltransactionrevenue,
				transactionrevenue,
				transactions
FROM all_sessions

UPDATE revenue
SET totaltransactionrevenue =
CASE
	WHEN totaltransactionrevenue IS NULL THEN NULL
	ELSE (totaltransactionrevenue / 1000000)::NUMERIC(10,2)
END

ALTER TABLE revenue
ALTER COLUMN transactionrevenue TYPE NUMERIC

UPDATE revenue
SET transactionrevenue = 
CASE
	WHEN transactionrevenue IS NULL THEN NULL
	ELSE (transactionrevenue / 1000000)::NUMERIC(10,2)
END

UPDATE revenue
SET city = 
CASE
	WHEN city = '(not set)' THEN 'N/A'
	WHEN city = 'not available in demo dataset' THEN 'N/A'
	ELSE city
END

UPDATE trans_rev
SET country = 
	CASE
		WHEN country = '(not set)' THEN 'N/A'
		ELSE country
	END

-- they are the same 
select totaltransactionrevenue, transactionrevenue from revenue
where revenue is not null

-- impact of revenue for each city 
SELECT city, SUM(totaltransactionrevenue) as totalrevenue
FROM revenue
WHERE totaltransactionrevenue IS NOT NULL
GROUP BY city
HAVING SUM(totaltransactionrevenue) IS NOT NULL

-- impact of revenue for each city
SELECT country, SUM(totaltransactionrevenue) as totalrevenue
FROM revenue 
WHERE totaltransactionrevenue IS NOT NULL
GROUP BY country 
HAVING SUM(totaltransactionrevenue) IS NOT NULL
```


Answer:







