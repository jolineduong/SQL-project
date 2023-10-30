What are your risk areas? Identify and describe them.
1. incomplete data - many columns in these tables are >75% null
2. different values in different tables for the same columns - columns in different tables that should contain the same information do not match
3. duplicates in product name and skus - there are many products that have different skus


QA Process:
Describe your QA process and include the SQL queries used to execute it.
For a lot of these risk areas, I decided to find the missing/problem values instead. 
```
SELECT * 
FROM all_sessions
WHERE country = '(not set)' or city = '(not set)' or city = 'not available in demo dataset'

SELECT * 
FROM all_sessions
WHERE totaltransactionrevenue IS NULL

SELECT timeonsite 
FROM all_sessions
WHERE timeonsite NOT IN (SELECT timeonsite FROM analytics)
limit 20

SELECT totalordered 
FROM sales_by_sku
WHERE totalordered NOT IN (SELECT totalordered FROM sales_report)
limit 20

SELECT v2productname, productsku, COUNT(*)
FROM all_sessions
GROUP BY v2productname, productsku
HAVING COUNT(*) > 1
```
