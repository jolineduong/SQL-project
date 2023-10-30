What issues will you address by cleaning the data?
I will be able to analyze the data properly by being able to create queries that can properly display the necessary and relevant data. I also want to be able to reduce the abiguities of the dataset. 




Queries:
Below, provide the SQL queries you used to clean your data.

```
-- I worked off temporary tables to avoid directly affecting the dataset and to only display what I needed to work with. 
-- I cleaned the data within these temporary tables.
----------------------------------------
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

-- divided all financial columns by 1000000 and changed their datatype to numeric so that the division can be clean and rounded to 2 decimal places
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

-- replaced missing data from city and country column to 'N/A' to make it easier to filter out later on
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
----------------------------------------
CREATE TEMPORARY TABLE ordered AS
SELECT DISTINCT	sku,
		totalordered,
		name
FROM sales_report 

-- changed data type
ALTER TABLE ordered
ALTER COLUMN totalordered TYPE numeric
----------------------------------------
CREATE TEMPORARY TABLE time_date AS
SELECT DISTINCT	country,
				city,
				fullvisitorid,
				visitid,
				timeonsite,
				date
FROM all_sessions

-- changed timeonsite to numeric so it could divide and round to 2 decimal places
ALTER TABLE time_date 
ALTER COLUMN timeonsite TYPE NUMERIC 

UPDATE time_date
SET timeonsite = 
	CASE 
		WHEN timeonsite IS NULL THEN NULL
		ELSE (timeonsite / 60)::NUMERIC(10,2)
	END

-- separated the date column into 3 new columns
ALTER TABLE time_date
ADD COLUMN year INT,
ADD COLUMN month INT,
ADD COLUMN day INT

-- extracted year, month, and day from the date column to put into the 3 new columns
UPDATE time_date
SET year = EXTRACT(YEAR FROM date),
    month = EXTRACT(MONTH FROM date),
    day = EXTRACT(DAY FROM date)
	
UPDATE time_date
SET city = 
	CASE
		WHEN city = '(not set)' THEN 'N/A'
		WHEN city = 'not available in demo dataset' THEN 'N/A'
		ELSE city
	END
	
UPDATE time_date
SET country = 
	CASE
		WHEN country = '(not set)' THEN 'N/A'
		ELSE country
	END
```

