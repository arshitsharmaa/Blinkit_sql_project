# Blinkit Data Analysis and Analytics using SQL

![Blinkit logo](https://github.com/arshitsharmaa/Blinkit_sql_project/blob/main/blinkit-logo-vector_logoshape.com.png)

## Overview

The **Blinkit SQL Project** is a comprehensive data management and analytics solution designed to analyze the operations of Blinkit, a leading quick-commerce platform. SQL is used to create a database schema, manage transactional data, and perform insightful analytics to support business decision-making.

The project covers multiple Blinkit's operations, including customer management, order processing, delivery performance, inventory tracking, marketing effectiveness, and customer feedback analysis.

---

## Project Objectives

- Design and implement a relational database schema to model Blinkit's core business entities.
- Populate and maintain data integrity across customers, orders, products, deliveries, inventory, marketing campaigns, and customer feedback.
- Perform detailed SQL queries to extract actionable insights such as sales trends, customer segmentation, delivery efficiency, and product performance.
- Utilize advanced SQL features including window functions, common table expressions (CTEs), stored procedures, and user-defined functions to solve complex business problems.
- Enable data-driven decision-making by providing reports on customer lifetime value, marketing ROAS, delivery delays, and product sales contributions.

---

## Database Schema Overview

The schema consists of the following key tables:

## Creation of tables
```sql
DROP TABLE IF EXISTS customer_feedback;
CREATE TABLE customer_feedback (
    feedback_id SERIAL PRIMARY KEY,
    order_id BIGINT,
    customer_id INT,
    rating INT,
    feedback_text TEXT,
    feedback_category VARCHAR(50),
    sentiment VARCHAR(20),
    feedback_date DATE
);
```

```sql
DROP TABLE IF EXISTS customers;
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(100),
    phone BIGINT,
    address TEXT,
    area VARCHAR(50),
    pincode INT,
    registration_date DATE,
    customer_segment VARCHAR(50),
    total_orders INT,
    avg_order_value DECIMAL(10,2)
);
```

```sql
DROP TABLE IF EXISTS delivery_performance;
CREATE TABLE delivery_performance (
    order_id BIGINT PRIMARY KEY,
    delivery_partner_id INT,
    promised_time TIMESTAMP,
    actual_time TIMESTAMP,
    delivery_time_minutes INT,
    distance_km DECIMAL(5,2),
    delivery_status VARCHAR(100),
    reasons_if_delayed VARCHAR(100)
);
```

```sql
DROP TABLE IF EXISTS inventory;
CREATE TABLE inventory (
    product_id INT,
    date DATE,
    stock_received INT,
    damaged_stock INT,
    PRIMARY KEY (product_id, date)
);
```

```sql
DROP TABLE IF EXISTS marketing_performance;
CREATE TABLE marketing_performance (
    campaign_id INT PRIMARY KEY,
    campaign_name VARCHAR(50),
    date DATE,
    target_audience VARCHAR(50),
    channel VARCHAR(20),
    impressions INT,
    clicks INT,
    conversions INT,
    spend DECIMAL(10, 2),
    revenue_generated DECIMAL(10, 2),
    roas DECIMAL(5, 2)
);
```

```sql
DROP TABLE IF EXISTS order_items;
CREATE TABLE order_items (
    order_id BIGINT PRIMARY KEY,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10, 2)
);
```

```sql
DROP TABLE IF EXISTS orders;
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    customer_id BIGINT,
    order_date TIMESTAMP,
    promised_delivery_time TIMESTAMP,
    actual_delivery_time TIMESTAMP,
    delivery_status VARCHAR(60),
    order_total DECIMAL(10,2),
    payment_method VARCHAR(60),
    delivery_partner_id INT,
    store_id INT
);
```

```sql
DROP TABLE IF EXISTS products;
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(100),
    brand VARCHAR(200),
    price DECIMAL(10, 2),
    mrp DECIMAL(10, 2),
    margin_percentage DECIMAL(5, 2),
    shelf_life_days INT,
    min_stock_level INT,
    max_stock_level INT
);
```


## Making relationships

```sql
ALTER TABLE inventory ADD CONSTRAINT fk_inventory_products
FOREIGN KEY (product_id) REFERENCES products(product_id);
```

```sql
ALTER TABLE order_items ADD CONSTRAINT fk_order_items_products
FOREIGN KEY (product_id) REFERENCES products(product_id);
```

```sql
ALTER TABLE customer_feedback ADD CONSTRAINT fk_customer_feedback_customers
FOREIGN KEY (customer_id) REFERENCES customers(customer_id);
```

```sql
ALTER TABLE orders ADD CONSTRAINT fk_orders_customers
FOREIGN KEY (customer_id) REFERENCES customers(customer_id);
```

```sql
ALTER TABLE order_items ADD CONSTRAINT fk_order_items_orders
FOREIGN KEY (order_id) REFERENCES orders(order_id);
```

```sql
ALTER TABLE customer_feedback ADD CONSTRAINT fk_customer_feedback_orders
FOREIGN KEY (order_id) REFERENCES orders(order_id);
```

```sql
ALTER TABLE delivery_performance ADD CONSTRAINT fk_delivery_performance_orders
FOREIGN KEY (order_id) REFERENCES orders(order_id);
```

## Data
```sql
SELECT * FROM customer_feedback;
SELECT * FROM customers;
SELECT * FROM marketing_performance;
SELECT * FROM orders;
SELECT * FROM products;
SELECT * FROM delivery_performance;
SELECT * FROM inventory;
SELECT * FROM order_items;
```

# Business problems and solutions.
1. -- Display the first 10 records from the blinkit_customer_feedback table.

```sql
SELECT * FROM customer_feedback
ORDER BY feedback_id ASC
LIMIT 10;
```

2. -- List all unique feedback_category values from the feedback table.

```sql
SELECT DISTINCT(feedback_category) as feedback_category
FROM customer_feedback;
```

3. -- Count how many feedback entries are Positive, Negative, and Neutral.

```sql
SELECT sentiment,count(sentiment)
FROM customer_feedback
GROUP BY 1;
```

4. -- Retrieve customer names and emails for customers who registered after June 1, 2024.

```sql
SELECT customer_name,email
FROM customers
WHERE registration_date > '2024-06-01';
```

5. -- Find all orders where the customer gave a rating of 1 or 2.

```sql
SELECT *
FROM customer_feedback
WHERE rating IN (1, 2);
```

6. -- Calculate the average rating from the customer feedback table.

```sql
SELECT ROUND(AVG(rating),2) AS average_rating
FROM customer_feedback;
```

7. -- Show the most common payment method used in orders.

```sql
SELECT payment_method,COUNT(payment_method) AS payment_count
FROM orders
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

8. -- List all products with a price above ₹500 and a margin of 25%.

```sql
SELECT product_name,price,margin_percentage
FROM products
WHERE price > 500 AND margin_percentage = 25;
```

9. -- Count the number of orders placed by each customer segment (Premium, Regular, Inactive, etc.).

```sql
SELECT customer_segment,COUNT(customer_segment) orders_placed
FROM customers AS c
INNER JOIN orders AS o
ON c.customer_id = o.customer_id
GROUP BY 1;
```

10. -- Find the earliest and latest order dates in the dataset.

```sql
SELECT MAX(order_date) AS last_order_date,
MIN(order_date) AS first_order_date
FROM orders;
```

--11. Display order details (order_id, date, total) along with the customer name for orders placed in March 2024.

```sql
SELECT c.customer_name,o.order_id,o.order_date,o.order_total FROM orders AS o
INNER JOIN customers AS c
ON o.customer_id = c.customer_id
WHERE o.order_date BETWEEN '2024-03-01 00:00:00' AND '2024-03-31 23:59:59'
ORDER BY o.order_date ASC;
```

--12. Show product names and categories for all items in order #1961864118.

```sql
SELECT oit.*,p.product_name,p.category
FROM products AS p
JOIN order_items AS oit
ON p.product_id = oit.product_id
WHERE oit.order_id = 1961864118;
```

--13. Calculate the average order value (AOV) for each customer segment.

```sql
SELECT c.customer_segment,ROUND(AVG(o.order_total),2) AS avg_order_value FROM customers AS c
INNER JOIN orders AS o
ON c.customer_id = o.customer_id
GROUP BY 1
ORDER BY 2 DESC;
```

--14. Find the total revenue generated by each product category.

```sql
WITH revenue_each_category AS (
SELECT p.category AS category,oit.quantity * oit.unit_price AS Total_revenue FROM products AS p
LEFT JOIN order_items AS oit
ON p.product_id = oit.product_id
)

SELECT category,SUM(total_revenue) AS total_revenue
FROM revenue_each_category
GROUP BY 1
ORDER BY 2 DESC;
```

--15. Identify the most frequently damaged product in inventory.

```sql
WITH most_damaged_product AS (
SELECT p.product_name,inv.damaged_stock
FROM products AS p
INNER JOIN inventory AS inv
ON p.product_id = inv.product_id
)

SELECT product_name AS frequent_damaged_product,SUM(damaged_stock) AS total_damaged_stock
FROM most_damaged_product
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

```sql
--16. Calculate the average delivery time (in minutes) for on-time vs. delayed deliveries.

SELECT delivery_status,ROUND(AVG(delivery_time_minutes),4) AS avg_delivery_time 
FROM delivery_performance
GROUP BY 1;
```

--17. Determine the month with the highest number of orders in 2024.


```sql
SELECT TO_CHAR(order_date, 'Month') AS month_name,COUNT(EXTRACT(day FROM order_date)) AS highest_orders
FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 2024
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

--18. List customers who haven’t placed an order in the last 6 months.

```sql
SELECT o.customer_id,c.customer_name,c.email,MAX(o.order_date) AS last_order_date FROM customers AS c
LEFT JOIN orders AS o
ON c.customer_id = o.customer_id
GROUP BY o.customer_id,c.customer_name,c.email
HAVING MAX(o.order_date) < (CURRENT_DATE - INTERVAL '6 months')
OR MAX(o.order_date) IS NULL
ORDER BY 4 DESC;
```



--19. Find orders where the actual delivery time was later than promised.

```sql
SELECT order_id,
       promised_time,
	   actual_time,
	   delivery_time_minutes FROM delivery_performance
WHERE delivery_time_minutes > 0;
```

--20. Rank products by the number of times they were ordered (highest to lowest).

```sql
WITH rnking AS (
SELECT p.product_name,COUNT(p.product_id) AS order_count FROM products AS p
INNER JOIN order_items AS oit
ON p.product_id = oit.product_id
GROUP BY 1
ORDER BY 2 DESC
)
SELECT product_name,order_count,DENSE_RANK() OVER(ORDER BY order_count DESC) FROM rnking
```


--21. Identify customers who gave ratings lower than their personal average rating.

```sql
SELECT * FROM (
SELECT cf.customer_id,c.customer_name,cf.rating,
ROUND(AVG(rating) OVER(PARTITION BY cf.customer_id),2) AS avg_ratings FROM customer_feedback AS cf
INNER JOIN customers AS c
ON c.customer_id = cf.customer_id) AS T
WHERE rating < avg_ratings;
```

--22. Find products that have never been ordered.

```sql
SELECT * FROM products AS p
LEFT JOIN order_items AS oit
ON p.product_id = oit.product_id
WHERE oit.product_id IS NULL;
```

--23. Show marketing campaigns with ROAS (Return on Ad Spend) higher than the platform average.

```sql
WITH _roas_ AS (
SELECT campaign_name,ROUND(revenue_generated/spend,2)AS roas,date
FROM marketing_performance
)

SELECT * FROM(
SELECT *,
ROUND(AVG(roas) OVER(),2) AS platform_avg_roas FROM _roas_) AS T
WHERE roas > platform_avg_roas;
```


--24. Rank customers by total spending (highest to lowest) using window functions.

```sql
WITH rnk AS (
SELECT customer_id,
       SUM(order_total) AS total_orders FROM orders
GROUP BY 1
)

SELECT customer_id,
       total_orders,RANK() OVER(ORDER BY total_orders DESC) AS ranking 
FROM rnk;
```

--25. Calculate a 7-day moving average of order values.

```sql
WITH weekly_revenue AS (
SELECT order_date::DATE AS order_date,
       SUM(order_total) AS total_orders FROM orders
GROUP BY 1
ORDER BY 1 ASC
)

SELECT *,
       ROUND(AVG(total_orders) OVER(ROWS BETWEEN 6 PRECEDING AND CURRENT ROW),2) AS moving_avg_7day
FROM weekly_revenue;
```


--26. Compare each product’s price to the average price in its category.

```sql
WITH compare_prices AS (
SELECT category,product_name,price,
ROUND(AVG(price) OVER(PARTITION BY category),2) AS avg_price FROM products
)

SELECT *,
CASE 
  WHEN price > avg_price THEN 'Above average'
  WHEN price < avg_price THEN 'Below average'
  ELSE 'equal to average'
END AS price_status
FROM compare_prices;
```


--27. Create a customer lifetime value (LTV) report showing total spend, order count, and average order value.

```sql
SELECT o.customer_id,
       c.customer_name,
	   SUM(o.order_total) AS total_spend,
	   COUNT(c.total_orders) AS order_count,
	   ROUND(AVG(o.order_total),2) AS avg_order_value FROM customers AS c
INNER JOIN orders AS o
ON c.customer_id = o.customer_id
GROUP BY 1,2
ORDER BY 3 DESC;
```

--28. Analyze the relationship between delivery time and customer ratings (does longer delivery = lower ratings?).

```sql
SELECT df.delivery_status,ROUND(AVG(cf.rating),2) AS average_ratings,
COUNT(cf.rating) AS number_of_orders
FROM delivery_performance AS df
INNER JOIN customer_feedback AS cf
ON df.order_id = cf.order_id
GROUP BY 1;
```


-- 29. Create a function named GetCustomerSegment that accepts a customer_id as input and returns the customer_
-- segment (e.g., 'Premium', 'Regular', 'Inactive') for that customer from the blinkit_customers table.

```sql
SELECT * FROM customers
```

```sql
CREATE OR REPLACE FUNCTION get_customer_segment (p_customer_id BIGINT)
RETURNS VARCHAR(50)
LANGUAGE plpgsql
AS $$
DECLARE
    segment VARCHAR(50);
BEGIN
    SELECT customer_segment INTO segment FROM customers
	WHERE customer_id = p_customer_id;
	RETURN COALESCE(segment,'Not found');
END;
$$;
```

```sql
SELECT get_customer_segment(8536504);
```


--30. Create a procedure named UpdateDeliveryStatus that updates the delivery_status in the 
-- blinkit_delivery_performance table for a given order_id. The procedure should accept two parameters:
-- the order_id and the new status. It should also check if the provided order_id exists before attempting
-- the update.

```sql
SELECT distinct(delivery_status) FROM delivery_performance;
```
```sql
SELECT * FROM delivery_performance;
```

```sql
CREATE OR REPLACE PROCEDURE update_delivery_status(p_order_id BIGINT,p_new_status VARCHAR(100))
LANGUAGE plpgsql
AS $$
DECLARE
     order_exists VARCHAR(50);
BEGIN
    IF p_new_status NOT IN('Slightly Delayed','On Time','Significantly Delayed') THEN
	     RAISE EXCEPTION 'Invalid status: %. Allowed values are "Slightly Delayed","On Time","Significantly Delayed"',p_new_status;
	END IF;

    SELECT 
	    CASE
		   WHEN EXISTS(SELECT 1 FROM delivery_performance WHERE order_id = p_order_id) THEN 'exists'
		   ELSE 'Not exists' 
        END INTO order_exists;
		
	IF order_exists = 'Not exists' THEN 
	       RAISE EXCEPTION 'The order_id % is not avalible in table',p_order_id;
	ELSE
	    UPDATE delivery_performance
	    SET delivery_status = p_new_status
	    WHERE order_id = p_order_id OR delivery_status = p_new_status;
	END IF;

	RAISE NOTICE 'Delivery status for order % updated to %:',p_order_id,p_new_status;

END;
$$;
```
```sql
CALL update_delivery_status(1961864118,'On Time');
```
```sql
CALL update_delivery_status(9799231260,'Slightly Delayed');
```
```sql
SELECT * FROM delivery_performance WHERE order_id = 1961864118;
```
```sql
SELECT * FROM delivery_performance WHERE order_id = 9799231260;
```

-- 31. How do you find the % Contribution of Each Product to Total Sales using SQL?	

```sql
WITH product_sales AS (
SELECT p.product_id,p.product_name,SUM(oit.quantity*oit.unit_price) AS total_sales FROM products AS p
JOIN order_items AS oit
ON p.product_id = oit.product_id
GROUP BY p.product_id,p.product_name
ORDER BY 3 DESC
),
overall_sales AS (
SELECT *,SUM(total_sales) OVER() overall_total_sales FROM product_sales
)

SELECT product_id,product_name,total_sales,ROUND((total_sales/overall_total_sales)*100,3) AS percentage_sales_contribution
FROM overall_sales;
```

--32. How do you find TOP 10 Total Sales by Region in SQL?

```sql
SELECT c.area,SUM(order_total) AS total_sales FROM customers AS c
JOIN orders AS o
ON c.customer_id = o.customer_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```
