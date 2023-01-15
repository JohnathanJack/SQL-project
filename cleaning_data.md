What issues will you address by cleaning the data?

1. Unit Costs/Revenue are 1,000,000 more than they are on all tables
2. sales_report name != products name FOR 7 Dog Frisbee vs 7" Dog Frisbee
3. all_session table: Remove product_refund_amount, item_quantity, item_revenue, search_keyword as they have NULL values in each column 
4. Some currency_code are NULL -> Infer to change to USD 

Queries:
Below, provide the SQL queries you used to clean your data.


Clean-Up 1: sales_report name != products name for 7 Dog Frisbee vs 7" Dog Frisbee, 

Updated products table:
```sql
SELECT sku 
	,(CASE WHEN name = '7 Dog Frisbee' THEN '7" Dog Frisbee'
		ELSE name END) AS name
	,ordered_quantity
	,stock_level
	,restocking_lead_time
	,sentiment_score
	,sentiment_magnitude
FROM products;
```
Clean-Up 2: all_session table: 
- Remove  product_refund_amount, item_quantity, item_revenue, search_keyword as they are all NULL values
- Remove transaction_revenue as it is duplicated with total_transaction_revenue
- Remove session_quality_dim, page_path_level_1, ecommerce_action_type, e_commerce_action_step, ecommerce_action_option as they are irrelevant
- Some currency_code are NULL -> Infer to change to USD 
- Divide product_price by 1,000,000 as they are 1,000,000 more than they are
- Moved transaction_id column beside the other transactions for clarity
- Transformed city values that were '(not set)' or 'not available in demo dataset' to NULL
- Transformed product_variant values that were '(not set)' to NULL
- Removed duplicates at the end by making full_visitor_id and product_sku as the unique key

```sql
--Check if transaction_revenue is duplicated with total_transaction_revenue
SELECT total_transaction_revenue
    ,transaction_revenue 
FROM all_sessions
WHERE total_transaction_revenue IS NOT NULL 
AND transaction_revenue IS NOT NULL 
AND total_transaction_revenue != transaction_revenue
--0 queries shown indicating they are the same. There are more NULL values in transaction_revenue compared to total_transaction_revenue so I removed the transaction_revenue column. 

--Primary clean-up for a new table.
SELECT full_visitor_id
	,channel_grouping
	,time
	,country
	,(CASE 
	  WHEN city = '(not set)' THEN NULL
	  WHEN city = 'not available in demo dataset' THEN NULL
	  ELSE city
	  END) AS city
	,transaction_id
	,total_transaction_revenue/1000000 AS total_transaction_revenue
	,transactions
	,time_on_site
	,pageviews
	,date
	,visit_id
	,type
	,product_quantity
	,product_price/1000000 AS product_price
	,product_revenue/1000000 AS product_revenue
	,product_sku
	,v2_product_name
	,v2_product_category
	,(CASE
	  WHEN product_variant = '(not set)' THEN NULL
	  ELSE product_variant
	  END) AS product_variant
	,(CASE 
	  WHEN currency_code IS NULL THEN 'USD'
	  ELSE currency_code 
	  END) AS currency_code
	,page_title
FROM all_sessions

--Final clean-up removing duplications
WITH CTE AS 
(
SELECT full_visitor_id
	,channel_grouping
	,time
	,country
	,city
	,transaction_id
	,total_transaction_revenue
	,transactions
	,time_on_site
	,pageviews
	,date
	,visit_id
	,type
	,product_quantity
	,product_price
	,product_revenue
	,product_sku
	,v2_product_name
	,v2_product_category
	,product_variant
	,currency_code
	,page_title
	,ROW_NUMBER() OVER (PARTITION BY full_visitor_id,product_sku ORDER BY full_visitor_id,product_sku) AS row_num
FROM updated_all_sessions
)
SELECT full_visitor_id
	,channel_grouping
	,time
	,country
	,city
	,transaction_id
	,total_transaction_revenue
	,transactions
	,time_on_site
	,pageviews
	,date
	,visit_id
	,type
	,product_quantity
	,product_price
	,product_revenue
	,product_sku
	,v2_product_name
	,v2_product_category
	,product_variant
	,currency_code
	,page_title
FROM CTE
WHERE row_num = 1
```
Clean-up 3: analytics table
- Divide revenue and unit_price by 1,000,000 as they are 1,000,000 more than they are
```sql
SELECT visit_number
	,visit_id
	,visit_start_time
	,date
	,full_visitor_id
	,user_id
	,channel_grouping
	,social_engagement_type
	,units_sold
	,pageviews
	,timeonsite
	,bounces
	,revenue/1000000 AS revenue
	,unit_price/1000000 AS unit_price
FROM analytics
```
