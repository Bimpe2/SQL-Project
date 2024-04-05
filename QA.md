## What are your risk areas? Identify and describe them.

#### **Risk 1 -** Revenue being recorded for a session when no visitor was browsing. If there's a revenue is attached to a time on site session with zero value, there's a risk that the revenue is not accurate

*QA Process :* 
My QA process is to validate that for every time on site, there's no accompanying revenue recorded for that session.

```
SELECT *
FROM all_sessions
WHERE time_on_site IS NULL
	AND total_transaction_revenue IS NOT NULL
;
```
##### Result: This query showed that there was no time on site with a null value that resulted in revenue 

#### ** Risk 2 -** Products sku that are included in the "all session" *table* but not found in the products *table*. It would mean that those products should not exist as the products *table* should have every product sku listed on there

``` 
SELECT count(distinct product_sku)
FROM all_sessions
WHERE product_sku NOT IN (
	SELECT sku
	FROM products)
LIMIT 50;
```

##### Result: This process showed that there were 147 unique products that can be found in the "all sessions" *table*, that can not be found in the "products" *table*. This would mean either the products *table* is not complete or there are incorrect information in the "all sessions" *table*

#### **Risk 3 -** To determine if there are irregular activities in the session dates: if the disparity from the average of the transactions in a day is great it might mean either that are lots of *duplicate rows* or *incorrect data* on that day. If the day is actually a special day  with for example Christmas or boxing day, it would still be explainable why alot of transactions could be occurring on such a day

* I used this query to calculate the avg rows from each day in the session_day column.
``` 
SELECT ROUND(AVG(row_count),2) AS avg_ct
FROM (
	SELECT DISTINCT * FROM (
	SELECT DISTINCT DATE(session_date) AS session_date, COUNT(*) AS

	row_count FROM all_sessions

	WHERE session_date IS NOT NULL
	GROUP BY 1
	ORDER BY session_date DESC
	) AS svg
) as rnd
```
* It showed on average there are 41.35 rows for each date.*

``` 
SELECT DISTINCT *,
	CASE 
		WHEN row_count > (2 * 41.35) THEN 'too many'
		WHEN row_count < (41.35/2) THEN 'too less'
		ELSE 'around average rows'
		END AS valid_test
FROM (
	SELECT DISTINCT DATE(session_date) AS session_date, COUNT(*) AS row_count FROM all_sessions
	WHERE session_date IS NOT NULL
	GROUP BY 1
	ORDER BY session_date DESC
	)  AS distinct_date_count
	```
	
##### Result: With this query, we can determine what dates might contain error or irregular transactions and investigate the rows related to those days

