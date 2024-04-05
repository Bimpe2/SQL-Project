## Question 1: Is there a relationship between the total revenue and the sentiment score of each product

### SQL Queries: 
```
WITH sentiment_price AS (
(SELECT sku,
	sentiment_score, 
	sentiment_magnitude,
	avg(product_price) AS avg_productprice,
	'higher price' AS price_level
FROM products p
JOIN view_allsessions4 vas ON p.sku = vas.product_sku
WHERE total_revenue > 0
GROUP BY sku
ORDER BY avg_productprice desc
LIMIT 5)

UNION 

(SELECT sku,
	sentiment_score, 
	sentiment_magnitude,
	avg(product_price) AS avg_productprice,
	'lower price' AS price_level
FROM products p
JOIN view_allsessions4 vas ON p.sku = vas.product_sku
WHERE total_revenue > 0
GROUP BY sku
ORDER BY avg_productprice 
LIMIT 5)
)

SELECT price_level, 
	avg(sentiment_score) AS avg_sentimentscore, 
	avg(avg_productprice) AS avg_productprice
FROM sentiment_price
GROUP BY price_level
;
```
### Query Comments: 
 
 1. While using the UNION function, I created a CTE that shows the sentiment score, and average product price as well as a price level column (to indicate high or low price).
 2. I also filtered for total_revenue greater than 0, as I was hoping to focus on transactions that resulted in revenue and what the average sentiment scores were for those products
 3. I used inner join when joining the products table, as I only wanted to focus on products that had both a sentiment score as well as price 
 3. I proceeded to check on the average of the sentiment score and price, and grouping with the pricel_level column that I had created for the seprate union table.
 
 

### Answer: 
Although, there's difference of 0.1 in the average sentiment of the customers depending on if a product price is high or low. The difference is not *material* relatively to the difference in average of the product price. 
As a result, I do not believe the pricing of the product has a substantial effect on the sentiment score of the customers

|price_level  |avg_sentimentscore|avg_productprice|
|-------------|------------------|----------------|
|high price   |0.3000000104308128|144.1600000000000000|
|low price    |0.4000000014901161|2.20000000000000000000|


## Question 2: Is there a trend in the months that the customers are making the purchase on

### SQL Queries: 
```
SELECT
	EXTRACT(month FROM session_date) AS months, count(total_transaction_revenue) AS revenue_count 
FROM all_sessions
WHERE total_transaction_revenue IS NOT NULL
GROUP BY EXTRACT(month FROM session_date)
ORDER BY  1
limit 50;
```

### Answer:
* There doesn't seem to be a trend in the months that the revenue are occuring. I was hoping to see if the seasons or quarter played an effect in the purchasing behavior of the customers  

|months       |revenue_count|
|-------------|-------------|
|1            |10           |
|2            |3            |
|3            |12           |
|4            |9            |
|5            |12           |
|6            |6            |
|7            |4            |
|8            |2            |
|9            |4            |
|10           |6            |
|11           |3            |
|12           |10           |




## Question 3: What country & city had the most search for each product category

### SQL Queries:
1. I created a category cte, and by using the `case when` function, I was able to clean up the "v2 product category" column, while exploring the data I noticed that there was a theme in how I could better categorize the data in the  "V2-product_category" column i.e clothes
``` 
With category AS (
SELECT
	fullvisitor_id,
	Country,
	city, 
	v2_product_category,
	total_ordered,
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
		WHEN lower (v2_product_category) LIKE '%fruit games%' THEN 'Fruit Games'
		WHEN lower (v2_product_category) LIKE '%fun%' THEN 'Fun'
		ELSE v2_product_category
END AS product_category
FROM view_allsessions3
	JOIN sales_by_sku USING (product_sku)
ORDER BY product_category
), 
```

2. I created a new cte called "product country rank", where I counted the number of unique visitors that were searching for the product.

3. cleaned the data by filtering out for the product_category that contained *CatTitle* or *not set* as I did not want to see that product category in my result. I also decided to eliminate the city and country that had NULL values from the dataset as I did not want those shown in my result

```
product_country_rank AS (
SELECT country, city, product_category, count(distinct fullvisitor_id ) AS visitor_inquiry
FROM category
WHERE city IS NOT NULL
		AND country IS NOT NULL
		AND product_category NOT LIKE '%CatTitle%' 
		AND product_category NOT LIKE '%not set%' 
GROUP BY product_category,country,city
ORDER BY 4 DESC ),
```

4. Created a new CTE called "highest rank product" while utilizing the window function.
I partitioned my data using the product category, so the `rank` function can rank the product category column while ordering the result by "visitor inquiry" *(an alias that counts unique visitor)* by descending order. To ensure that the highest search product or visitor inquiry for each product category would be ranked higher

```highest_rank_product AS (
SELECT *, 
	RANK() OVER (PARTITION BY product_category order by visitor_inquiry DESC) AS rank_prod
FROM product_country_rank
)
```
5. I filtered for just products that were ranked as 1, to show just the highest visitor inquiry product

```
SELECT *
FROM highest_rank_product
WHERE rank_prod = 1
```

### Answer:
* United states was the top country for all products but the fun product category.
* Product category such as Gift Cards and Sales had more than 1 country & city tying for the top position 

|country      |city         |product_category|visitor_inquiry|rank_prod|
|-------------|-------------|----------------|---------------|---------|
|United States|Mountain View|Accessories     |77             |1        |
|United States|Mountain View|Bags            |52             |1        |
|United States|Mountain View|Brands          |86             |1        |
|United States|Mountain View|Clothes         |280            |1        |
|United States|Mountain View|Drinkware       |25             |1        |
|United States|Mountain View|Electronics     |92             |1        |
|Canada       |Toronto      |Fun             |1              |1        |
|United States|Sunnyvale    |Gift Cards      |1              |1        |
|United States|San Francisco|Gift Cards      |1              |1        |
|United States|San Jose     |Gift Cards      |1              |1        |
|United States|Sunnyvale    |Houseware       |1              |1        |
|United States|Mountain View|Kids            |4              |1        |
|United States|Mountain View|Lifestyle       |29             |1        |
|United States|Mountain View|Nest            |85             |1        |
|United States|Mountain View|Office          |50             |1        |
|United States|San Jose     |Sales           |2              |1        |
|United States|New York     |Sales           |2              |1        |
|United States|Mountain View|Waze            |1              |1        |


