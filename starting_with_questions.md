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
ORDER BY SUM(total_transaction_revenue) DESC

--For city
SELECT city
	,SUM(total_transaction_revenue)
FROM no_dup_all_sessions
WHERE total_transaction_revenue IS NOT NULL 
GROUP BY city
ORDER BY SUM(total_transaction_revenue) DESC

--For Country/City Combination
SELECT 
	country
    ,city
	,SUM(total_transaction_revenue)
FROM updated_all_sessions
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
ORDER BY AVG(s.total_ordered) DESC

--Country Alone
SELECT a.country
	,avg(s.total_ordered) 
FROM no_dup_all_sessions a
JOIN sales_by_sku s ON a.product_sku = s.productsku
GROUP BY a.country
ORDER BY AVG(s.total_ordered) DESC

--City Alone 
SELECT a.city
	,avg(s.total_ordered) 
FROM no_dup_all_sessions a
JOIN sales_by_sku s ON a.product_sku = s.productsku
GROUP BY a.city
ORDER BY AVG(s.total_ordered) DESC
```

Answer:
The average number of products ordered from visitors in each country/city is as shown in the table provided by the SQL Query. Saurdi Arabia/Riyadh and Czechia/Brno at top with 319.0 average number of products ordered. When looking at the countries on their own, the highest average its 96.3 orders by Saudi Arabia. 


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
ORDER BY COUNT(product_type) DESC
```


Answer:
From the above query, it is seen that a majority of the orders were from the category of apparel, brands and electronics. 




**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







