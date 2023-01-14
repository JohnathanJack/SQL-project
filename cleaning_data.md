What issues will you address by cleaning the data?

Unit Costs cost 1,000,000 more than they are on all tables

Some product SKU are not found in the sales_by_sku and all_sessions tables. 
    - 1092 unique sku from products
    - 536 unique sku from all_sessions
    - 462 unique sku from sales_by_sku
    - 454 unique sku from sales_report

Number of sales from sales_report does not match sales_by_sku
    -462 sales_by_sku
    -454 sales_report

Number of total orders from sales_report does not much sales_by_sku
    -5524 sales_by_sku
    -5519 sales_report

sales_report name != products name FOR 7 Dog Frisbee vs 7" Dog Frisbee


all_session table: Remove  product_refund_amount, item_quantity, item_revenue, search_keyword, 

Look into due to low # of values 
-> session_quality_dim 4000/60500 
-> transaction_revenue 16/60500
-> transaction_id 16/60500
-> total_transaction_revenue 324
-> transactions 324
-> product_quantity 212
-> product_revenue 16


Some currency_code are NULL -> Infer to change to USD (USE CASE WHEN)


analytics table: 4301122 total entries
user_id(4205975 IS NULL), units_sold(4205975 IS NULL), timeonsite(477465 IS NULL), bounces (474839 NOT null), revenue(4285767 IS NULL)




Queries:
Below, provide the SQL queries you used to clean your data.


Clean-Up Task 1: sales_report name != products name for 7 Dog Frisbee vs 7" Dog Frisbee, 

Fixed products table:
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
Clean-Up Task 2: all_session table: 
- Remove  product_refund_amount, item_quantity, item_revenue, search_keyword as they are all NULL values
- Remove transaction_revenue as it is duplicated with total_transaction_revenue
- Some currency_code are NULL -> Infer to change to USD 
- Divide product_price by 1,000,000 as they are 1,000,000 more than they are
- Moved transaction_id column beside the other transactions for clarity
- Transformed city values that were '(not set)' or 'not available in demo dataset' to NULL


```sql
--Check if transaction_revenue is duplicated with total_transaction_revenue
SELECT total_transaction_revenue
    ,transaction_revenue 
FROM all_sessions
WHERE total_transaction_revenue IS NOT NULL 
AND transaction_revenue IS NOT NULL 
AND total_transaction_revenue != transaction_revenue
--0 queries shown indicating they are the same. There are more NULL values in transaction_revenue compared to total_transaction_revenue so I removed the transaction_revenue column. 

--Final clean-up
SELECT full_visitor_id
	,channel_grouping
	,time
	,country
	,(CASE 
	  WHEN city = '(not set)' THEN NULL
	  WHEN city = 'not available in demo dataset' THEN NULL
	  ELSE city
	  END)
    ,transaction_id
	,total_transaction_revenue/1000000 AS total_transaction_revenue
	,transactions
	,time_on_site
	,pageviews
	,session_quality_dim
	,date
	,visit_id
	,type
	,product_quantity
	,product_price/1000000 AS product_price
	,product_revenue/1000000 AS product_revenue
	,product_sku
	,v2_product_name
	,v2_product_category
	,product_variant
	,(CASE WHEN currency_code IS NULL THEN 'USD'
	ELSE currency_code END) AS currency_code
	,page_title
	,page_path_level_1
	,ecommerce_action_type
	,ecommerce_action_step
	,ecommerce_action_option
FROM all_sessions;
```
Clean-up Task 3: analytics table
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
