Answer the following questions and provide the SQL queries used to find the answer.

    
## ** Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


###  SQL Queries: 
* Creating a CTE, I selected the country and city and their respective total_revenue, I filtered out for total revenue that were not zero as they were not adding any extra information to my query.
* I also utilized the `window` function for ntile to show the rank of the total transaction ordered by  total revenue in a descending order

```
WITH highlevel_transactions AS (
SELECT
	Country,
	city,
	total_revenue,
	NTILE(1000) OVER( ORDER BY total_revenue DESC  ) as ranking
FROM view_allsessions1
WHERE
	total_revenue != 0
	AND city IS NOT NULL 
)
```
* I then used the `Group by` function as I wanted to group all revenue transaction from the country & city into one row. 
* I used the `Limit 5`  function to show the top 5 country and city with the highest revenue
* I used the `order by desc` function to ensure that the highest amounts were being displayed first

```
SELECT country,city, sum(total_revenue) AS combine_revenue
FROM highlevel_transactions
GROUP BY country,city
ORDER BY sum(total_revenue) DESC
LIMIT 5;
```

### Answer:
* What I noticed from the result was that 4 of the 5 top revenue were from the United States, this made me curious about what other country or city were the other customers from?, and if the major or primary customers were from the united states

|country      |city         |combined_revenue|
|-------------|-------------|----------------|
|United States|San Francisco|1561            |
|United States|Sunnyvale    |991             |
|United States|Atlanta      |853             |
|United States|Palo Alto    |608             |
|Israel       |Tel Aviv-Yafo|602             |





## Question 2: What is the average number of products ordered from visitors in each city and country? 


#### SQL Queries:
* To further clean or prepare my data, I used the `case when` function to calculate for a new column called quantity. For every total revnue amount that was greater than zero, I was okay with calculating for the quantity that would have been sold. This is because the existence of a total revenue indicated to me that items were actually ordered.
* I also filtered out for rows that had a total revenue amout of zero, so it would not skew the result of the average of the products that were ordered from customers that actually made a transaction
```
WITH total_ordered AS (
	SELECT
		country,
		city,
		product_sku,
		CASE 
			WHEN total_revenue > 0 THEN total_revenue/product_price
			ELSE 0
		END AS quantity
	FROM view_allsessions2
	WHERE product_sku IN (SELECT product_sku FROM sales_by_sku)
	AND total_revenue != 0	
)		
```
*With the cte that I had already previoulsly made, I then calculated for the average quantity for each country and city. 
```
SELECT 
	country,
	city, 
	avg(quantity) AS avg_ordered
FROM total_ordered
WHERE city IS NOT NULL
GROUP BY country, city
	;
```

#### Answer:

|country      |city         |avg_ordered|
|-------------|-------------|-----------|
|United States|Palo Alto    |1.3333333333333333|
|United States|Sunnyvale    |216.6666666666666667|
|United States|Chicago      |22.6666666666666667|
|United States|Mountain View|1.2500000000000000|
|Australia    |Sydney       |3.0000000000000000|
|United States|San Francisco|4.2222222222222222|
|United States|Austin       |9.0000000000000000|
|United States|Houston      |1.00000000000000000000|
|United States|San Bruno    |5.0000000000000000|
|United States|Seattle      |3.0000000000000000|
|United States|New York     |5.6666666666666667|
|Canada       |New York     |1.00000000000000000000|
|United States|Nashville    |1.00000000000000000000|
|United States|Los Angeles  |5.0000000000000000|
|United States|San Jose     |1.00000000000000000000|



## Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**



#### SQL Queries:
* I used the `case when` function to tidy up the "v2 product category" table, so I can easily group common themed product tpgether
```
With category AS (
SELECT
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
		ELSE v2_product_category
END AS product_category
FROM view_allsessions3
	JOIN sales_by_sku USING (product_sku)
ORDER BY product_category
) 
```

* I noticed that my result contained city or country that were null values, so I used `WHERE` to filter out for those and also for the product category that showed 'catTitle' or not set values 
```
SELECT country, city, Count(distinct product_category) AS unique_category
FROM category
WHERE city IS NOT NULL
		AND country IS NOT NULL
		AND product_category NOT LIKE '%CatTitle%' 
		AND product_category NOT LIKE '%not set%' 
GROUP BY country,city
ORDER BY 3 DESC 
```


#### Answer: 

|country      |city         |unique_category|
|-------------|-------------|---------------|
|United States|Sunnyvale    |13             |
|United States|Mountain View|12             |
|United States|San Jose     |11             |
|United States|New York     |11             |
|Canada       |Toronto      |10             |
|United States|Los Angeles  |10             |
|Australia    |Sydney       |10             |
|United States|Austin       |10             |
|United States|San Francisco|10             |
|United States|Palo Alto    |9              |
|Ireland      |Dublin       |9              |
|Hong Kong    |Hong Kong    |9              |
|United States|Seattle      |9              |
|United States|Chicago      |9              |
|United States|Irvine       |9              |
|United States|Salem        |9              |
|United States|Cambridge    |8              |
|United States|Washington   |8              |
|Brazil       |Sao Paulo    |8              |
|United Kingdom|London       |8              |
|United States|Kirkland     |8              |
|United States|Ann Arbor    |8              |
|United States|Santa Clara  |8              |
|United States|Atlanta      |8              |
|United States|Houston      |8              |
|Australia    |Melbourne    |8              |
|India        |Mumbai       |8              |
|United States|Fremont      |7              |
|India        |Bengaluru    |7              |
|Mexico       |Mexico City  |7              |
|Singapore    |Singapore    |7              |
|Canada       |Montreal     |7              |
|Israel       |Tel Aviv-Yafo|7              |
|United States|San Bruno    |7              |
|France       |Paris        |7              |
|India        |Hyderabad    |7              |
|United States|Pittsburgh   |6              |
|India        |New Delhi    |6              |
|United States|Charlotte    |6              |
|Japan        |Minato       |6              |
|United States|Boston       |6              |
|South Korea  |Seoul        |6              |
|Ukraine      |Kiev         |6              |
|Spain        |Barcelona    |6              |
|Poland       |Warsaw       |6              |
|Colombia     |Bogota       |6              |
|United States|Dallas       |6              |
|Thailand     |Bangkok      |6              |
|India        |Chennai      |5              |
|Turkey       |Istanbul     |5              |
|Switzerland  |Zurich       |5              |
|Sweden       |Stockholm    |5              |
|United States|Philadelphia |5              |
|Germany      |Hamburg      |5              |
|India        |Pune         |5              |
|Italy        |Milan        |5              |
|Japan        |Shinjuku     |5              |
|United States|San Diego    |5              |
|Malaysia     |Kuala Lumpur |5              |
|India        |Kolkata      |4              |
|Argentina    |Buenos Aires |4              |
|United States|Denver       |4              |
|Canada       |Vancouver    |4              |
|Japan        |Osaka        |4              |
|Romania      |Bucharest    |4              |
|United States|Oakland      |4              |
|United Arab Emirates|Dubai        |4              |
|Sri Lanka    |Colombo      |4              |
|United States|Phoenix      |4              |
|Chile        |Santiago     |4              |
|India        |Ahmedabad    |4              |
|Vietnam      |Ho Chi Minh City|4              |
|United States|Cupertino    |4              |
|Netherlands  |Amsterdam    |4              |
|Germany      |Berlin       |4              |
|Philippines  |Quezon City  |3              |
|Germany      |Munich       |3              |
|India        |Gurgaon      |3              |
|United States|Eau Claire   |3              |
|United States|Detroit      |3              |
|United States|Orlando      |3              |
|United States|San Antonio  |3              |
|Indonesia    |Jakarta      |3              |
|United States|Redwood City |3              |
|United States|South San Francisco|3              |
|Australia    |Brisbane     |3              |
|Canada       |Mississauga  |3              |
|Russia       |Moscow       |3              |
|Canada       |Kitchener    |3              |
|United States|Boulder      |3              |
|Russia       |Saint Petersburg|3              |
|Spain        |Madrid       |3              |
|Philippines  |Taguig       |3              |
|United States|Redmond      |2              |
|Austria      |Vienna       |2              |
|Belgium      |Antwerp      |2              |
|Brazil       |Rio de Janeiro|2              |
|Canada       |Calgary      |2              |
|Canada       |Quebec City  |2              |
|Canada       |Sherbrooke   |2              |
|Colombia     |Medellin     |2              |
|Czechia      |Prague       |2              |
|France       |Montreuil    |2              |
|Greece       |Athens       |2              |
|Hungary      |Budapest     |2              |
|India        |Indore       |2              |
|Italy        |Rome         |2              |
|Japan        |Yokohama     |2              |
|Kenya        |Nairobi      |2              |
|New Zealand  |Auckland     |2              |
|Pakistan     |Karachi      |2              |
|Peru         |La Victoria  |2              |
|Philippines  |Manila       |2              |
|Romania      |Timisoara    |2              |
|Spain        |Pozuelo de Alarcon|2              |
|Taiwan       |Zhongli District|2              |
|United States|Ashburn      |2              |
|United States|Avon         |2              |
|United States|Bellingham   |2              |
|United States|Council Bluffs|2              |
|United States|Jersey City  |2              |
|United States|Kansas City  |2              |
|United States|Lake Oswego  |2              |
|United States|Menlo Park   |2              |
|United States|Milpitas     |2              |
|United States|Norfolk      |2              |
|United States|Oviedo       |2              |
|United States|San Mateo    |2              |
|United States|Tempe        |2              |
|Venezuela    |Maracaibo    |2              |
|Vietnam      |Hanoi        |2              |
|Ukraine      |Kharkiv      |1              |
|Mexico       |Culiacan     |1              |
|Malaysia     |Petaling Jaya|1              |
|United Kingdom|Coventry     |1              |
|Malaysia     |Ipoh         |1              |
|United Kingdom|Manchester   |1              |
|United Kingdom|Salford      |1              |
|United Kingdom|Wrexham      |1              |
|United States|Akron        |1              |
|United States|Amsterdam    |1              |
|Lithuania    |Vilnius      |1              |
|United States|South El Monte|1              |
|Japan        |Shibuya      |1              |
|Japan        |San Francisco|1              |
|Australia    |Los Angeles  |1              |
|United States|Bangkok      |1              |
|United States|Bellflower   |1              |
|United States|St. Louis    |1              |
|Japan        |Nagoya       |1              |
|Japan        |Mountain View|1              |
|Japan        |Chuo         |1              |
|Ireland      |Cork         |1              |
|Indonesia    |Bandung      |1              |
|United States|Chico        |1              |
|United States|Columbia     |1              |
|United States|Stanford     |1              |
|India        |Patna        |1              |
|India        |Nanded       |1              |
|India        |Lucknow      |1              |
|India        |Kharagpur    |1              |
|United States|Druid Hills  |1              |
|United States|East Lansing |1              |
|India        |Jaipur       |1              |
|United States|Forest Park  |1              |
|India        |Chandigarh   |1              |
|United States|Goose Creek  |1              |
|United States|Greer        |1              |
|United States|Hayward      |1              |
|United States|Hong Kong    |1              |
|Hungary      |Istanbul     |1              |
|United States|Indianapolis |1              |
|Greece       |Thessaloniki |1              |
|United States|Jacksonville |1              |
|Australia    |Adelaide     |1              |
|Argentina    |Rosario      |1              |
|Germany      |Frankfurt    |1              |
|United States|LaFayette    |1              |
|United States|The Dalles   |1              |
|United States|Las Vegas    |1              |
|United States|London       |1              |
|France       |Villeneuve-d'Ascq|1              |
|United States|Madison      |1              |
|United States|Toronto      |1              |
|United States|Mexico City  |1              |
|United States|University Park|1              |
|United States|Minneapolis  |1              |
|France       |Singapore    |1              |
|United States|Nashville    |1              |
|France       |Marseille    |1              |
|Argentina    |Santa Fe     |1              |
|France       |Courbevoie   |1              |
|El Salvador  |San Salvador |1              |
|United States|Wellesley    |1              |
|Denmark      |Copenhagen   |1              |
|United States|Panama City  |1              |
|United States|Paris        |1              |
|Czechia      |Brno         |1              |
|Croatia      |Zagreb       |1              |
|United States|Piscataway Township|1              |
|China        |Beijing      |1              |
|United States|Pleasanton   |1              |
|United States|Portland     |1              |
|Canada       |Waterloo     |1              |
|Canada       |St. John's   |1              |
|United States|Rexburg      |1              |
|United States|Richardson   |1              |
|United States|Sacramento   |1              |
|Canada       |New York     |1              |
|Canada       |Edmonton     |1              |
|Canada       |Burnaby      |1              |
|Brazil       |Fortaleza    |1              |
|Brazil       |Belo Horizonte|1              |
|Belgium      |Ghent        |1              |
|Uruguay      |Montevideo   |1              |
|Australia    |Perth        |1              |
|Romania      |Iasi         |1              |
|Russia       |Vladivostok  |1              |
|Saudi Arabia |Riyadh       |1              |
|Romania      |Cluj-Napoca  |1              |
|South Africa |Westville    |1              |
|Qatar        |Doha         |1              |
|Portugal     |Lisbon       |1              |
|Poland       |Poznan       |1              |
|United States|Santa Monica |1              |
|Philippines  |Makati       |1              |
|Paraguay     |Asuncion     |1              |
|Pakistan     |Lahore       |1              |
|Taiwan       |Longtan District|1              |
|Australia    |Mountain View|1              |
|Norway       |Oslo         |1              |
|Netherlands  |Dublin       |1              |
|Turkey       |Izmir        |1              |

* For each country and city, they usually ordered from 13 to 1 unique category. 


## Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?


#### SQL Queries:
```
WITH country_product_rank AS(
SELECT 
	country, 
	city,
	product_sku,
	totaL_revenue, 
	product_price, 
	sum(total_revenue/product_price) AS quantity,
	Rank() OVER(PARTITION BY country, city ORDER BY (sum(total_revenue/product_price)) DESC ) AS product_rank

FROM view_allsessions4
WHERE total_revenue != 0
GROUP BY 1,2,3,4,5
)

SELECT *
FROM country_product_rank 
WHERE product_rank = 1
```

#### Answer:

* I noticed that the city sunnyvale in the country united states had a large quanity of 649 which stood out in comparison to other top products in other country or cities
 

|country      |city         |product_sku|total_revenue|product_price|quantity|product_rank|
|-------------|-------------|-----------|-------------|-------------|--------|------------|
|Australia    |Sydney       |GGOENEBB078899|358          |119          |3       |1           |
|Canada       |New York     |GGOEGAAX0358|67           |55           |1       |1           |
|Canada       |Toronto      |GGOEGAEJ030715|82           |19           |4       |1           |
|Israel       |Tel Aviv-Yafo|GGOENEBB079399|602          |79           |7       |1           |
|Switzerland  |Zurich       |GGOEGAAX0314|16           |24           |0       |1           |
|United States|Atlanta      |GGOEGBJR018199|742          |11           |67      |1           |
|United States|Austin       |GGOEGAAX0106|122          |13           |9       |1           |
|United States|Chicago      |GGOEGHGR019499|123          |2            |61      |1           |
|United States|Columbus     |GGOEGAAJ032617|21           |18           |1       |1           |
|United States|Houston      |GGOEGDHQ015399|38           |24           |1       |1           |
|United States|Los Angeles  |GGOEGAAX0279|116          |16           |7       |1           |
|United States|Mountain View|GGOEGAAX0625|35           |16           |2       |1           |
|United States|Mountain View|GGOEGAAH034014|26           |10           |2       |1           |
|United States|Nashville    |GGOENEBJ079499|157          |149          |1       |1           |
|United States|New York     |GGOEGFKQ020399|57           |2            |28      |1           |
|United States|Palo Alto    |GGOENEBQ078999|305          |119          |2       |1           |
|United States|San Bruno    |GGOEGAAX0339|103          |18           |5       |1           |
|United States|San Francisco|GGOEADWQ015699|139          |12           |11      |1           |
|United States|San Jose     |GGOEGAFB035814|108          |44           |2       |1           |
|United States|Seattle      |GGOENEBB078899|358          |119          |3       |1           |
|United States|Sunnyvale    |GGOEGCBQ016499|649          |1            |649     |1           |

* Another pattern worth nothing

#### Queries:
```
WITH country_product_rank AS(
SELECT 
	country, 
	city,
	product_sku,
	total_revenue, 
	product_price, 
	sum(total_revenue/product_price) AS quantity,
	Rank() OVER(PARTITION BY country, city ORDER BY (sum(total_revenue/product_price)) DESC ) AS product_rank

FROM view_allsessions4
WHERE total_revenue != 0
GROUP BY 1,2,3,4,5
),

top_selling_product AS (
SELECT *
FROM country_product_rank 
WHERE product_rank = 1
)

SELECT product_sku, product_name, count(product_sku) as count_country
FROM top_selling_product tsp
JOIN products p ON tsp.product_sku = p.sku
group by product_sku, product_name
HAVING count(product_sku) > 1
```

#### Answer:
* The product Cam Indoor Security Camera - USA is the only product that is the top product in more than 1 country

|product_sku  |product_name |count_country|
|-------------|-------------|-------------|
|GGOENEBB078899| Cam Indoor Security Camera - USA|2            |

## Question 5: Can we summarize the impact of revenue generated from each city/country?**

#### SQL Queries:

  ```
  SELECT 
  	Country, 
  	city, 
  	sum(total_revenue) as sum_revenue, 
  	avg(total_revenue) as avg_revenue, 
  	max(total_revenue) as max_revenue, 
  	min(total_revenue) as min_revenue
  FROM view_allsessions4
  WHERE total_revenue > 0
  GROUP BY 1, 2
  ORDER BY sum_revenue desc;
```

#### Answer:
* Was able to obtain the sum, average, maximum revenue, and minimum revenue and the count of the revenue amounts for each country and city that had revenue amounts that were greater than 0.

|country      |city         |sum_revenue|avg_revenue         |max_revenue|min_revenue|
|-------------|-------------|-----------|--------------------|-----------|-----------|
|United States|San Francisco|1561       |130.0833333333333333|301        |12         |
|United States|Sunnyvale    |991        |247.7500000000000000|649        |22         |
|United States|Atlanta      |853        |426.5000000000000000|742        |111        |
|United States|Palo Alto    |608        |202.6666666666666667|305        |151        |
|Israel       |Tel Aviv-Yafo|602        |602.0000000000000000|602        |602        |
|United States|New York     |526        |65.7500000000000000 |152        |7          |
|United States|Mountain View|479        |59.8750000000000000 |156        |8          |
|United States|Los Angeles  |479        |239.5000000000000000|363        |116        |
|United States|Chicago      |448        |149.3333333333333333|306        |19         |
|United States|Seattle      |358        |358.0000000000000000|358        |358        |
|Australia    |Sydney       |358        |358.0000000000000000|358        |358        |
|United States|San Jose     |262        |131.0000000000000000|154        |108        |
|United States|Austin       |157        |78.5000000000000000 |122        |35         |
|United States|Nashville    |157        |157.0000000000000000|157        |157        |
|United States|San Bruno    |103        |103.0000000000000000|103        |103        |
|Canada       |Toronto      |82         |82.0000000000000000 |82         |82         |
|Canada       |New York     |67         |67.0000000000000000 |67         |67         |
|United States|Houston      |38         |38.0000000000000000 |38         |38         |
|United States|Columbus     |21         |21.0000000000000000 |21         |21         |
|Switzerland  |Zurich       |16         |16.0000000000000000 |16         |16         |

