Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
```sql
--For country
SELECT country
	,SUM(total_transaction_revenue)
FROM no_dup_all_sessions
WHERE total_transaction_revenue IS NOT NULL 
GROUP BY country
ORDER BY SUM(total_transaction_revenue) DESC;

--For city
SELECT city
	,SUM(total_transaction_revenue)
FROM no_dup_all_sessions
WHERE total_transaction_revenue IS NOT NULL 
GROUP BY city
ORDER BY SUM(total_transaction_revenue) DESC;

--For Country/City Combination
SELECT country
    ,city
	,SUM(total_transaction_revenue)
FROM no_dup_all_sessions
WHERE total_transaction_revenue IS NOT NULL 
GROUP BY country, city
ORDER BY SUM(total_transaction_revenue) DESC;
```

Answer:
The country with the highest level of transaction is the United States with 13123. 
The city with the highest level of transaction that is known is San Francisco with 1561. Sorting just by city gives us a NULL city value with a transaction revenue of 6082. The country and city combination that gives the highest level of transaction revenue is United States and San Francisco (disregarding the NULL city value as we dont know how much of it is split amongst the other cities)


**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
```sql
--Country/City Combo
SELECT a.country
	,a.city
	,avg(s.total_ordered) 
FROM no_dup_all_sessions a
JOIN sales_by_sku s ON a.product_sku = s.productsku
GROUP BY a.country,a.city
ORDER BY AVG(s.total_ordered) DESC;

--Country Alone
SELECT a.country
	,avg(s.total_ordered) 
FROM no_dup_all_sessions a
JOIN sales_by_sku s ON a.product_sku = s.productsku
GROUP BY a.country
ORDER BY AVG(s.total_ordered) DESC;

--City Alone 
SELECT a.city
	,avg(s.total_ordered) 
FROM no_dup_all_sessions a
JOIN sales_by_sku s ON a.product_sku = s.productsku
GROUP BY a.city
ORDER BY AVG(s.total_ordered) DESC;
```

Answer:
The average number of products ordered from visitors in each country/city is as shown in the table provided by the SQL Query. Saurdi Arabia/Riyadh and Czechia/Brno at top with 319.0 average number of products ordered. When looking at the countries on their own, the highest average is 96.3 orders by Saudi Arabia. 


**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
```sql
with product_sold AS 
(
SELECT a.country AS country
	,a.city AS city
	,s.productsku AS productsku
	,s.total_ordered AS total_ordered
	,a.v2_product_category AS product_type
FROM no_dup_all_sessions a
JOIN sales_by_sku s ON a.product_sku = s.productsku
)
SELECT country
	,city
	,product_type
	,COUNT(product_type)
FROM product_sold
GROUP BY country,city,product_type 
--HAVING country != 'United States' -> used to show the other inquires as US dominated the query.
ORDER BY COUNT(product_type) DESC;
```


Answer:
From the above query, it is seen that a majority of the orders were from the category of apparel, brands and electronics. 




**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

```sql
--Created a new table to be able to query for top products
SELECT * INTO top_country_prods
FROM 
(
WITH order_table AS
(
SELECT a.country AS country
	,a.city AS city
	,rep.name AS name
	,count(rep.name) AS num_prod
FROM no_dup_all_sessions a
JOIN sales_report rep ON rep.productsku = a.product_sku
GROUP BY a.country,a.city,rep.name
ORDER BY count(rep.name) DESC
)
SELECT country
	,city
	,name
	,num_prod
	,ROW_NUMBER() OVER (PARTITION BY country ORDER BY num_prod DESC) AS rankings
FROM order_table
) AS subq;

--Now I query for the top rankings for each country
SELECT * FROM top_country_prods
WHERE rankings = 1 
ORDER BY num_prod DESC;

```
Answer:
From this new table, it looks like the most bought item is the Twill Cap for many countries. 

**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
```sql
--Create a table with the revenue, country and city
--DISCLAIMER: I could not distinguish between duplicate data in the analytics table, so I used the full_visitor_id as my primary key and took the average of the products sold and price to calculate the revenue

SELECT * INTO revenue_table
FROM 
(
WITH rev_tab AS
(
SELECT 
	full_visitor_id
	,visit_id
	,AVG(units_sold) OVER (PARTITION BY full_visitor_id) AS avg_units_sold
	,AVG(unit_price) OVER (PARTITION BY full_visitor_id) AS avg_unit_price
	,ROW_NUMBER() OVER (PARTITION BY full_visitor_id) AS rank
FROM updated_analytics
)
SELECT 
	full_visitor_id
	,visit_id
	,ROUND(avg_units_sold,0) AS avg_units_sold
	,ROUND(avg_unit_price,2) AS avg_unit_price
	,(CASE
	  WHEN avg_units_sold IS NOT NULL AND avg_unit_price IS NOT NULL THEN ROUND(avg_units_sold * avg_unit_price,2) 
	  ELSE 0
	  END) AS revenue
FROM rev_tab 
WHERE rank = 1
) AS subq;

--From the table, 
--For Country Alone
SELECT country
	,SUM(revenue) AS total_revenue
FROM revenue_table 
GROUP BY country
ORDER BY SUM(revenue) DESC;

--For City Alone
SELECT city
	,SUM(revenue) AS total_revenue
FROM revenue_table 
GROUP BY city
ORDER BY SUM(revenue) DESC;

--For Country/City Together
SELECT country
	,city
	,SUM(revenue) AS total_revenue
FROM revenue_table 
GROUP BY country,city
ORDER BY SUM(revenue) DESC;
```


Answer:

From the data generate from the tables above, it is seen that the united states generates the most revenue for the website. The city that generates the most revenue is Mountain View from the United States. 





