**What are your risk areas? Identify and describe them.**


1. Check to make sure the no_dup_all_sessions does not contain any duplicate data. If there are any duplicates, it will alter the results that query through the country/city. 

2. Check the new table top_country_prods max num_prods value matches the top query provided

**QA Process:
Describe your QA process and include the SQL queries used to execute it.**

```SQL
--QA Check for seeing no_dup_all_sessions contains any duplication
SELECT full_visitor_id
	,product_sku
	,SUM(1) AS count
FROM no_dup_all_sessions
GROUP BY full_visitor_id,product_sku
HAVING SUM(1) > 1 
--No rows given back indicating the cleaning of duplicates was successful

--QA Check to ensure top product matches the max number 
SELECT MAX(num_prod) from top_country_prods
--Both query gives us a max of 86 indicating the table is correct in that regard
```

