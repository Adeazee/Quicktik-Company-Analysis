# Quicktik-Company-Analysis: Sales and Profit Analysis for Business Growth

## Project Overview
This project analyzes a company’s sales, profit, and customer behavior to understand performance trends and identify opportunities for business growth.
The analysis focuses on examining the relationship between sales, profit, and customer distribution in order to determine why the company is not experiencing steady sales growth.

Tools used in this project include:
* SQL – data cleaning and transformation
* Excel – initial data exploration

## Business Background
A newly appointed manager reviewed the company’s performance reports and believed that sales were increasing while profits were decreasing. This raised concerns that the company might be experiencing rising operational costs or inefficiencies that were eroding profitability.
As a result, a data analysis was requested to investigate the relationship between sales and profit and identify the root cause of the suspected decline in profitability.

## Business Problem
The initial concern from management was:
* Why are sales increasing while profits appear to be decreasing?

## Dataset Description
### The dataset consists of multiple tables representing different aspects of the business:
##### Orders Table
- order_id 
- customer_id 
- product_id
- order_date
- quantity
- unit_price
- discount_percent
- shipping_fee
#### Customers Table
- customer_id
-signup_date
- region
- acquisition_channel
- age
- gender
#### Product Table
- product_id
- product_name
- category
- cost_price
- supplier
- launch_date
- status

## Data Cleaning
cleaning order table
``` sql SELECT * FROM easyshop.orders;

-- working on data type
describe orders;
UPDATE orders
SET order_date = STR_TO_DATE(order_date, '%c/%e/%Y');

ALTER TABLE orders
MODIFY column order_date DATE;

-- clearing duplicates
# for orders
select * FROM orders;

With duplicate_cte as
(
SELECT *,
       ROW_NUMBER() OVER (
           PARTITION BY order_id, customer_id, order_date, quantity, unit_price, discount_percent, order_status
           ORDER BY order_id
       ) AS row_num
from orders
)
select * 
from duplicate_cte
where row_num > 1;

CREATE TABLE `orders2` (
  `order_id` int DEFAULT NULL,
  `order_date` text,
  `customer_id` int DEFAULT NULL,
  `product_id` int DEFAULT NULL,
  `quantity` int DEFAULT NULL,
  `unit_price` double DEFAULT NULL,
  `discount_percent` double DEFAULT NULL,
  `shipping_fee` double DEFAULT NULL,
  `order_status` text,
  `payment_method` text,
  `row_num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

select * 
from orders2
where row_num > 1;

insert into orders2
SELECT *,
       ROW_NUMBER() OVER (
           PARTITION BY order_id, customer_id, order_date, quantity, unit_price, discount_percent, order_status
           ORDER BY order_id
       ) AS row_num
from orders;

SET SQL_SAFE_UPDATES = 0;

delete
from orders2
where row_num > 1;

select * 
from orders2;


select * from orders;

-- checking for missing values
-- for orders

SELECT 
    SUM(CASE WHEN order_id IS NULL THEN 1 ELSE 0 END) AS missing_order_id,
    SUM(CASE WHEN order_date IS NULL THEN 1 ELSE 0 END) AS missing_order_date,
    SUM(CASE WHEN customer_id IS NULL THEN 1 ELSE 0 END) AS missing_customer_id,
    SUM(CASE WHEN product_id IS NULL THEN 1 ELSE 0 END) AS missing_product_id,
    SUM(CASE WHEN quantity IS NULL OR quantity = 0 THEN 1 ELSE 0 END) AS missing_quantity,
    SUM(CASE WHEN unit_price IS NULL OR unit_price = 0 THEN 1 ELSE 0 END) AS missing_unit_price,
    SUM(CASE WHEN discount_percent IS NULL THEN 1 ELSE 0 END) AS missing_discount_percent,
    SUM(CASE WHEN shipping_fee IS NULL THEN 1 ELSE 0 END) AS missing_shipping_fee,
    SUM(CASE WHEN order_status IS NULL OR TRIM(order_status) = '' THEN 1 ELSE 0 END) AS missing_order_status,
    SUM(CASE WHEN payment_method IS NULL OR TRIM(payment_method) = '' THEN 1 ELSE 0 END) AS missing_payment_method
FROM orders;
```

Cleaning customer table
```sql
SELECT * FROM easyshop.customers;

-- working on data type
describe customers;

SELECT age
FROM customers
WHERE age IS NOT NULL AND age NOT REGEXP '^[0-9]+$';

ALTER TABLE customers
MODIFY COLUMN age INT;

-- clearing duplicates

#for customer table
With duplicate_cte as
(
SELECT *,
       ROW_NUMBER() OVER (
           PARTITION BY customer_id, region
           ORDER BY customer_id
       ) AS row_num
from customers
)
select * 
from duplicate_cte
where row_num > 1;

-- standardizing data

select * from customers;
SELECT DISTINCT region FROM customers;
select * from customers
where region like 'p%';
update customers
set region = 'Port Harcourt'
where region like 'p%';
where region like 'l%';
update customers
set region = 'Lagos'
where region like 'l%';
select * from customers
where acquisition_channel like 'f%';
update customers
set acquisition_channel = 'Facebook'
where acquisition_channel like 'f%';


-- checking for missing values
-- for customer table
SELECT 
    SUM(CASE WHEN customer_id IS NULL OR TRIM(customer_id) = '' THEN 1 ELSE 0 END) AS missing_customer_id,
    SUM(CASE WHEN signup_date IS NULL THEN 1 ELSE 0 END) AS missing_signup_date,
    SUM(CASE WHEN region IS NULL OR TRIM(region) = '' THEN 1 ELSE 0 END) AS missing_region,
    SUM(CASE WHEN acquisition_channel IS NULL OR TRIM(acquisition_channel) = '' THEN 1 ELSE 0 END) AS missing_acquisition_channel,
    SUM(CASE WHEN age IS NULL OR age = 0 THEN 1 ELSE 0 END) AS missing_age,
    SUM(CASE WHEN gender IS NULL OR TRIM(gender) = '' THEN 1 ELSE 0 END) AS missing_gender
FROM customers;

UPDATE customers
SET acquisition_channel = 'Unknown'
WHERE acquisition_channel IS NULL 
   OR TRIM(acquisition_channel) = '';
   
UPDATE customers
SET age = NULL
WHERE age = 'Null';
```

Cleaning product table
```sql
SELECT * FROM easyshop.product;

-- working on data type
describe product;

UPDATE product
SET launch_date = STR_TO_DATE(launch_date, '%c/%e/%Y');

ALTER TABLE product
MODIFY column launch_date DATE;

-- clearing duplicates
# for product
With duplicate_cte as
(
SELECT *,
       ROW_NUMBER() OVER (
           PARTITION BY product_id
           ORDER BY product_id
       ) AS row_num
from product
)
select * 
from duplicate_cte
where row_num > 1;

select * from product;
SELECT DISTINCT category FROM product;
SELECT category, COUNT(*)
FROM product
GROUP BY category;
SELECT BINARY category AS category, COUNT(*)
FROM product
GROUP BY BINARY category;
UPDATE product
SET category = upper(TRIM(category));
SELECT DISTINCT category FROM product;


-- checking for missing values
-- for product table
SELECT 
    SUM(CASE WHEN product_id IS NULL THEN 1 ELSE 0 END) AS missing_product_id,
    SUM(CASE WHEN product_name IS NULL OR TRIM(product_name) = '' THEN 1 ELSE 0 END) AS missing_product_name,
    SUM(CASE WHEN category IS NULL OR TRIM(category) = '' THEN 1 ELSE 0 END) AS missing_category,
    SUM(CASE WHEN cost_price IS NULL OR cost_price = 0 THEN 1 ELSE 0 END) AS missing_cost_price,
    SUM(CASE WHEN supplier IS NULL OR TRIM(supplier) = '' THEN 1 ELSE 0 END) AS missing_supplier,
    SUM(CASE WHEN launch_date IS NULL THEN 1 ELSE 0 END) AS missing_launch_date,
    SUM(CASE WHEN status IS NULL OR TRIM(status) = '' THEN 1 ELSE 0 END) AS missing_status
FROM product;

```

## Analysis
```sql
-- verifying the data
SELECT COUNT(*) FROM orders;
SELECT COUNT(*) FROM inventory;
SELECT COUNT(*) FROM customers;
SELECT COUNT(*) FROM marketing;
SELECT COUNT(*) FROM product;

/* all imported well other than product table */

SELECT * FROM products LIMIT 5;
SELECT DATABASE();
SHOW TABLES;
DROP TABLE IF EXISTS product;

SELECT COUNT(*) FROM product;
/* fixed(product has been sucessfully imported) */

-- Calculating completed sales
SELECT 
order_id,
order_date,
quantity,
unit_price,
discount_percent,
(quantity * unit_price) AS sales
FROM orders
WHERE order_status = 'Completed';

SELECT 
SUM(quantity * unit_price) AS total_sales
FROM orders
WHERE order_status = 'Completed';

-- Checking completed sales trend  
SELECT 
YEAR(order_date) AS year,
MONTH(order_date) AS month,
SUM(quantity * unit_price) AS total_sales
FROM orders
WHERE order_status = 'Completed'
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY year, month;

SELECT 
DATE_FORMAT(order_date, '%Y-%m') AS month,
SUM(quantity * unit_price) AS total_sales
FROM orders
WHERE order_status = 'Completed'
GROUP BY month
ORDER BY month;

-- calculating profit
SELECT 
SUM(
(o.quantity * o.unit_price)
- p.cost_price
- o.discount_percent
- o.shipping_fee
) AS total_profit
FROM orders o
JOIN product p
ON o.product_id = p.product_id
WHERE o.order_status = 'Completed';

-- checking profit trends
SELECT 
YEAR(o.order_date) AS year,
MONTH(o.order_date) AS month,
SUM(
(o.quantity * o.unit_price)
- p.cost_price
- o.discount_percent
- o.shipping_fee
) AS total_profit
FROM orders o
JOIN product p
ON o.product_id = p.product_id
WHERE o.order_status = 'Completed'
GROUP BY YEAR(o.order_date), MONTH(o.order_date)
ORDER BY year, month;

/* checked profit and sales trend to verify if what the manager said about sales increasing and profit shrinking is true or false.
 From the analysis above it isn't true, sales and profit have been fluctuating in the same range rather than stesdily increasing or decreasing  since 2023.

This is not good though as the company is meant to be growing.

important question:  what is the profit margin?
                     why has the company been fluctuating and not steadily growing? */

-- Total Profit Margin
SELECT 
SUM(
(o.quantity * o.unit_price)
- (p.cost_price * o.quantity)
- o.discount_percent
- o.shipping_fee
) 
/
SUM(o.quantity * o.unit_price) * 100 AS profit_margin

FROM orders o
JOIN product p
ON o.product_id = p.product_id

WHERE o.order_status = 'Completed';

/* The analysis shows that the company has a total profit margin of 23%, meaning that for every 1 unit of revenue generated, the company retains approximately 0.23 as profit after accounting for costs, discounts, and other expenses. This indicates that the company is still operating profitably and maintains a relatively strong margin.

Although the company maintains a healthy profit margin of 23%, the sales and profit trends fluctuate over time rather than showing consistent growth. This suggests that while the company is profitable, it may not be experiencing steady expansion.*/


-- investigating why has the company sales been fluctuating and not steadily growing
-- numbers of orders by customers (that is how many times did a customer order)
SELECT 
customer_id,
COUNT(order_id) AS number_of_orders
FROM orders
GROUP BY customer_id
ORDER BY number_of_orders DESC;

-- one-time customers vs repeat customers
SELECT 
CASE 
    WHEN order_count = 1 THEN 'One-time customers'
    ELSE 'Repeat customers'
END AS customer_type,
COUNT(*) AS number_of_customers

FROM (
    SELECT customer_id, COUNT(order_id) AS order_count
    FROM orders
    GROUP BY customer_id
) t

GROUP BY customer_type;

/*This shows that the majority of your customers are repeat buyers, which is a good sign for customer loyalty.
Only ~19% of your customers are new, amd this explains the flucatuating around the same range because Most sales come from existing customers, who tend to buy regularly but not in increasing amounts.

If your existing customers buy about the same amount each period, total sales will fluctuate around the same range, rather than steadily grow..*/


/*INSIGHTS
* Total sales is $17343005
* sales has been fluctuating around the same range
* Total profit is $10912908
* profit has also been fluctuating around the same range
* The profit margin is 23%
* customers distribution is : Repeat customers	13733
                              one-time customers	3146*/
							
/* RECOMMENDATIONS
* Focus on acquiring new customers
   Only ~19% of your customers are new, so total sales growth is limited.
Strategies:
Run targeted marketing campaigns to attract new customer segments.
Offer referral incentives to encourage repeat customers to bring friends.
Explore new acquisition channels.
* Increase purchase frequency or value from repeat customers
  Repeat customers make up 81%, but if their purchase size or frequency doesn’t grow, sales stagnate.
Strategies:
Introduce loyalty programs or subscription offers.
Bundle products or offer upsells/cross-sells to increase average order value.
Use personalized promotions based on purchase history.
* Maintain healthy profit margins
  Your profit margin is already strong at 23%, so avoid cutting prices too aggressively.
Strategies:
Focus on operational efficiency to reduce costs without lowering prices.
Keep promotions targeted to avoid unnecessary margin erosion.*/
```

## Key Insights
* Total sales is $17343005
* sales has been fluctuating around the same range
* Total profit is $10912908
* profit has also been fluctuating around the same range
* The profit margin is 23%
* customers distribution is : Repeat customers	13733
                              one-time customers	3146

## Recommendations
1. Focus on acquiring new customers: Only ~19% of your customers are new, so total sales growth is limited.
- Strategies:
Run targeted marketing campaigns to attract new customer segments.
Offer referral incentives to encourage repeat customers to bring friends.
Explore new acquisition channels.
2. Increase purchase frequency or value from repeat customers: Repeat customers make up 81%, but if their purchase size or frequency doesn’t grow, sales stagnate.
-  Strategies:
Introduce loyalty programs or subscription offers.
Bundle products or offer upsells/cross-sells to increase average order value.
Use personalized promotions based on purchase history.
3. Maintain healthy profit margins: Your profit margin is already strong at 23%, so avoid cutting prices too aggressively.
-  Strategies:
Focus on operational efficiency to reduce costs without lowering prices.
Keep promotions targeted to avoid unnecessary margin erosion.
