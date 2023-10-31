Question 1: How many visits did the website receive in 2016 for each country/city?

SQL Queries:
```
CREATE TEMPORARY TABLE time_date AS
SELECT DISTINCT	country,
				city,
				fullvisitorid,
				visitid,
				timeonsite,
				date
FROM all_sessions

-- clean table
ALTER TABLE time_date 
ALTER COLUMN timeonsite TYPE NUMERIC 

UPDATE time_date
SET timeonsite = 
	CASE 
		WHEN timeonsite IS NULL THEN NULL
		ELSE (timeonsite / 60)::NUMERIC(10,2)
	END

ALTER TABLE time_date
ADD COLUMN year INT,
ADD COLUMN month INT,
ADD COLUMN day INT

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

-- avg time per country
SELECT country, ROUND(avg(timeonsite),2) as avg_minutes
FROM time_date
GROUP BY country
HAVING ROUND(avg(timeonsite),2) IS NOT NULL

-- avg time per city 
SELECT city, ROUND(avg(timeonsite),2) as avg_minutes
FROM time_date
GROUP BY city
HAVING ROUND(avg(timeonsite),2) IS NOT NULL
```

Answer: 
<img width="283" alt="Screen Shot 2023-10-30 at 6 37 41 PM" src="https://github.com/issajojoke/SQL-project/assets/140217218/b39faa7a-0eae-4197-805e-f0837f673814">
<img width="285" alt="Screen Shot 2023-10-30 at 6 37 57 PM" src="https://github.com/issajojoke/SQL-project/assets/140217218/53994f84-0d9b-4204-92f4-a7030bfde3ff">



Question 2: What was the avg time spent on the website in each city/country?

SQL Queries:
```
CREATE TEMPORARY TABLE time_date AS
SELECT DISTINCT	country,
				city,
				fullvisitorid,
				visitid,
				timeonsite,
				date
FROM all_sessions

select * from time_date where timeonsite is not null

-- clean table
ALTER TABLE time_date 
ALTER COLUMN timeonsite TYPE NUMERIC 

UPDATE time_date
SET timeonsite = 
	CASE 
		WHEN timeonsite IS NULL THEN NULL
		ELSE (timeonsite / 60)::NUMERIC(10,2)
	END

ALTER TABLE time_date
ADD COLUMN year INT,
ADD COLUMN month INT,
ADD COLUMN day INT

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
	
CREATE TEMPORARY TABLE visits AS
SELECT DISTINCT	visitnumber,
				timeonsite
FROM analytics

-- visits per country in 2016
SELECT td.country, SUM(v.visitnumber) as totalvisits
FROM time_date td
FULL JOIN visits v
ON td.timeonsite = v.timeonsite
WHERE td.year = 2016 and country <> 'N/A'
GROUP BY td.country
HAVING SUM(v.visitnumber) IS NOT NULL

-- visits per city in 2016
SELECT td.city, SUM(v.visitnumber) as totalvisits
FROM time_date td
FULL JOIN visits v
ON td.timeonsite = v.timeonsite
WHERE td.year = 2016 and city <> 'N/A'
GROUP BY td.city
HAVING SUM(v.visitnumber) IS NOT NULL
```

Answer:
<img width="259" alt="Screen Shot 2023-10-30 at 6 41 50 PM" src="https://github.com/issajojoke/SQL-project/assets/140217218/4ba60715-12c1-44da-b426-862b0c0b4833">
<img width="260" alt="Screen Shot 2023-10-30 at 6 42 09 PM" src="https://github.com/issajojoke/SQL-project/assets/140217218/b478fa40-e3f5-4328-881c-177a19de0ade">


Question 3: Which country had the greatest number of website visits? 

SQL Queries:
```
CREATE TEMPORARY TABLE time_date AS
SELECT DISTINCT	country,
				city,
				fullvisitorid,
				visitid,
				timeonsite,
				date
FROM all_sessions

-- clean table
ALTER TABLE time_date 
ALTER COLUMN timeonsite TYPE NUMERIC 

UPDATE time_date
SET timeonsite = 
	CASE 
		WHEN timeonsite IS NULL THEN NULL
		ELSE (timeonsite / 60)::NUMERIC(10,2)
	END

ALTER TABLE time_date
ADD COLUMN year INT,
ADD COLUMN month INT,
ADD COLUMN day INT

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
	
CREATE TEMPORARY TABLE visits AS
SELECT DISTINCT	visitnumber,
				timeonsite
FROM analytics

-- total visits per country
SELECT td.country, SUM(v.visitnumber) as totalvisits
FROM time_date td
FULL JOIN visits v
ON td.timeonsite = v.timeonsite
WHERE country IS NOT NULL and country <> 'N/A'
GROUP BY td.country 
HAVING SUM(v.visitnumber) IS NOT NULL
ORDER BY totalvisits DESC

-- total visits per city
SELECT td.city, SUM(v.visitnumber) as totalvisits
FROM time_date td
FULL JOIN visits v
ON td.timeonsite = v.timeonsite
WHERE city IS NOT NULL and city <> 'N/A'
GROUP BY td.city
HAVING SUM(v.visitnumber) IS NOT NULL
ORDER BY totalvisits DESC
```

Answer:
<img width="267" alt="Screen Shot 2023-10-30 at 6 43 39 PM" src="https://github.com/issajojoke/SQL-project/assets/140217218/f6bf93c6-0f16-48ed-ae0e-ea52dc4c0119">
<img width="257" alt="Screen Shot 2023-10-30 at 6 43 15 PM" src="https://github.com/issajojoke/SQL-project/assets/140217218/e14d3e40-c2bf-488b-93c2-a2eccbe12667">



