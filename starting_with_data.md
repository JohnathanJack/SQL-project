**Question 1: Did any order exceed the stock level?**

SQL Queries:
```sql
SELECT productsku
    ,name
FROM sales_report
WHERE total_ordered > stock_level
```

Answer: This query shows us that two items were ordered but the company does not have any in stock. This could be looked into and seeing if there is a need to increase the standard stock level. 



**Question 2: Which mode of searching(channel_grouping) gave the most amount of orders?**

SQL Queries:
```sql 
SELECT al.channel_grouping
	,COUNT(rep.total_ordered) AS total_orders_from_source
FROM no_dup_all_sessions al
JOIN sales_report rep ON al.product_sku = rep.productsku
GROUP BY al.channel_grouping
ORDER BY COUNT(rep.total_ordered) DESC
```
Answer: This query shows that the majority of orders are from people are getting to the site by an organic search followed by a direct search and then a referral. 



**Question 3: What percentage of customers that directly search the site are actually placing an order?**

SQL Queries:
```sql

SELECT 
	COUNT(al.channel_grouping) * 100.00 / 
	(SELECT 
		COUNT(al.channel_grouping) 
	FROM no_dup_all_sessions al
	JOIN sales_report rep ON al.product_sku = rep.productsku
	WHERE al.channel_grouping = 'Direct') AS percentage_of_buyers
FROM no_dup_all_sessions al
JOIN sales_report rep ON al.product_sku = rep.productsku
WHERE al.channel_grouping = 'Direct' AND total_ordered != 0;
```

Answer:From this query, it shows that around 74% of people who directly search the site actually places an order.




Question 4: 

SQL Queries:

Answer:



Question 5: 

SQL Queries:

Answer:
