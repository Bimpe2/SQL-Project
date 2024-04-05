## There are 6 issues I will be addressing by cleaning the data 

**Issue 1:** Cities are showing *(not set)* or *not available in demo dataset*. Query to change those cities to NULL values

**Issue 2:** Countries that are showing *(not set)* or *not available in demo dataset*. Query to change those countries to NULL values

**Issue 3:** Converting the total transaction revenue that are null values to zero and also dividing the reset of the transaction revenue amount by 1,000,000

**Issue 4:** Cleaning up rows that would not be needed for analyzing, by selecting only columns that we need and including them in a view

#### Queries:
```
CREATE OR REPLACE VIEW view_allsessions1 AS
	SELECT
		fullvisitor_id,
		total_transaction_revenue,
		CASE 
			WHEN total_transaction_revenue IS NULL THEN 0
			ELSE total_transaction_revenue/100000
		END AS total_revenue,
		CASE 
			WHEN city IN ('(not set)', 'not available in demo dataset') THEN NULL
			ELSE city
		END AS city,
		CASE 
			WHEN country IN ('(not set)', 'not available in demo dataset') THEN NULL
			ELSE country
		END AS country
		
	FROM all_sessions	
  ;
```

**Issue 5:** To tidy up the product category column by changing the v2_product_category attribute into a shortened category name when certain *string* appears in the column

```
Queries:
SELECT
	Country,
	city, 
	v2_product_category,
	CASE 
		WHEN lower(v2_product_category) LIKE '%kids%'  THEN 'Kids'
		WHEN lower(v2_product_category) LIKE '%apparel%'  THEN 'Clothes'
		WHEN lower(v2_product_category) LIKE '%shirts%'  THEN 'Clothes'
		WHEN lower(v2_product_category) LIKE '%bags%' THEN 'Bags'
		WHEN lower (v2_product_category) LIKE '%office%' THEN 'Office'
		WHEN lower (v2_product_category) LIKE '%electronics%' THEN 'Electronics'
		WHEN lower (v2_product_category) LIKE '%brand%' THEN 'Brands'
		WHEN lower (v2_product_category) LIKE '%accessories%' THEN 'Accessories'
		WHEN lower (v2_product_category) LIKE '%drinkware%' THEN 'Drinkware'
		WHEN lower (v2_product_category) LIKE '%houseware%' THEN 'Houseware'
		WHEN lower (v2_product_category) LIKE '%pet%' THEN 'Pet'
		WHEN lower (v2_product_category) LIKE '%sale%' THEN 'Sales'
		WHEN lower (v2_product_category) LIKE '%brand%' THEN 'Brands'
		WHEN lower (v2_product_category) LIKE '%lifestyle%' THEN 'Lifestyle'
	    WHEN lower (v2_product_category) LIKE '%gift card%' THEN 'Gift Cards'
		WHEN lower (v2_product_category) LIKE '%nest%' THEN 'Nest'
		ELSE v2_product_category
END AS product_category
FROM all_sessions
ORDER BY product_category

```
**Issue 6:** To ensure that all countries and cities displayed are not NULL values
Queries:
```
CREATE OR REPLACE VIEW view_allsessions4 AS
	WITH view_session3 AS (
	SELECT 
		fullvisitor_id,
			product_sku,
			product_price/1000000 AS product_price,
			v2_product_category,
			CASE 
				WHEN total_transaction_revenue IS NULL THEN 0
				ELSE total_transaction_revenue/1000000
			END AS total_revenue,
			CASE 
				WHEN city IN ('(not set)', 'not available in demo dataset') THEN NULL
				ELSE city
			END AS city,
			CASE 
				WHEN country IN ('(not set)', 'not available in demo dataset') THEN NULL
				ELSE country
			END AS country	
		FROM all_sessions
	) 
	SELECT *
	FROM view_session3
	WHERE city IS NOT NULL
		AND country IS NOT NULL
;
```
