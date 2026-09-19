# Assignment 1: Sunrise Supermarket Database

**Name:** [ahmed adam yasir ibrahim]
**Student ID:** [28865]
**Group:** [B]
**DBMS Used:** Oracle database

## 1. Business Scenario & Summary

**Business Scenario:** 
Sunrise Supermarket requires a robust data analysis of its sales, inventory, and customer purchasing habits. Management needs actionable insights to identify high-value customers, track revenue growth, and optimize inventory restocking.**

## 2. Create the tables

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10, 2)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

 <img width="2535" height="1278" alt="create tables" src="https://github.com/user-attachments/assets/0a72dd4e-f7c2-482e-b6cf-f2bf85dbd01c" />

## 3. Inserting
### insert orders_item
<img width="2534" height="1318" alt="insert order item" src="https://github.com/user-attachments/assets/cee7ce16-3baf-4ae2-a123-4f31d4550d94" />

### insert orders
<img width="2531" height="1175" alt="insert orders" src="https://github.com/user-attachments/assets/7fdbf22e-1c1c-4fec-971a-b2b919f1b85f" />

### insert custmers
<img width="2538" height="1201" alt="isnert customers" src="https://github.com/user-attachments/assets/d62586a6-266c-46d7-8311-1ced2648b65e" />

### insert products
<img width="2537" height="1166" alt="insert products" src="https://github.com/user-attachments/assets/1d44b18b-7964-497c-a1c4-c1f314558281" />

# Queries

## JOIN queries
#### 1-List every order with the customer's name, city, and order date (INNER JOIN: orders + customers).
```sql
SELECT orders.order_id,
       customers.customer_name,
       customers.city,
       orders.order_date
FROM orders
INNER JOIN customers
ON orders.customer_id = customers.customer_id
ORDER BY orders.order_date;
```
<img width="2539" height="1229" alt="query 1" src="https://github.com/user-attachments/assets/21ef252f-fcda-4bf6-83bc-7df60f1c3b99" />


#### 2-List every order item with product name, category, price, and quantity (JOIN: order_items + products).
```sql
SELECT order_items.order_id, order_items.order_item_id, products.product_name, products.category, products.price, order_items.quantity
FROM order_items
JOIN products ON order_items.product_id = products.product_id
ORDER BY order_items.order_id, order_items.order_item_id;
```
<img width="2547" height="1315" alt="join query 2" src="https://github.com/user-attachments/assets/5e368756-44d5-460f-b869-13b465ba7655" />


#### 3-List all customers and their orders where they exist, including customers with no orders (LEFT JOIN: customers + orders).
```sql
SELECT customers.customer_id, customers.customer_name, orders.order_id, orders.order_date
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id
ORDER BY customers.customer_id, orders.order_date;
```
<img width="2535" height="1317" alt="query join 3" src="https://github.com/user-attachments/assets/e360d6e6-d95a-4dda-9f59-6ddf89840d38" />


## CTE query
#### 1-Calculate each customer's total spend (quantity x price) and return customers above average spend. Use a CTE to compute customer totals first.
```sql
WITH CustomerTotals AS (
    SELECT customers.customer_id, customers.customer_name, SUM(order_items.quantity * products.price) AS total_spend
    FROM customers
    JOIN orders ON customers.customer_id = orders.customer_id
    JOIN order_items ON orders.order_id = order_items.order_id
    JOIN products ON order_items.product_id = products.product_id
    GROUP BY customers.customer_id, customers.customer_name
)
SELECT customer_name, total_spend
FROM CustomerTotals
WHERE total_spend > (SELECT AVG(total_spend) FROM CustomerTotals);
```
<img width="2532" height="1318" alt="query CTE" src="https://github.com/user-attachments/assets/6631c611-f9ae-4a38-93a0-12c2191ecc91" />


## Window-function queries
#### 1-Rank customers by total amount spent, highest first.
```sql
WITH CustomerSpend AS (
    SELECT customers.customer_id, customers.customer_name, SUM(order_items.quantity * products.price) AS total_spend
    FROM customers
    JOIN orders ON customers.customer_id = orders.customer_id
    JOIN order_items ON orders.order_id = order_items.order_id
    JOIN products ON order_items.product_id = products.product_id
    GROUP BY customers.customer_id, customers.customer_name
)
SELECT customer_name, total_spend,
       RANK() OVER(ORDER BY total_spend DESC) AS spend_rank
FROM CustomerSpend;
```
<img width="2529" height="1318" alt="query win 11" src="https://github.com/user-attachments/assets/a5f8805d-d757-4019-af80-440abe3afb2b" />



#### 2-Number each customer's orders in the order placed.
```sql
SELECT customers.customer_name, orders.order_id, orders.order_date,
       ROW_NUMBER() OVER(PARTITION BY customers.customer_id ORDER BY orders.order_date) AS order_sequence
FROM customers
JOIN orders ON customers.customer_id = orders.customer_id
ORDER BY customers.customer_name, orders.order_date;
```
<img width="2529" height="1304" alt="query win 2" src="https://github.com/user-attachments/assets/25c20a3b-3557-4434-9994-a1a6723f21b9" />



#### 3-Show a running total of revenue over time, ordered by order date.
```sql
WITH DailyRevenue AS (
    SELECT orders.order_date, SUM(order_items.quantity * products.price) AS daily_total
    FROM orders
    JOIN order_items ON orders.order_id = order_items.order_id
    JOIN products ON order_items.product_id = products.product_id
    GROUP BY orders.order_date
)
SELECT order_date, daily_total,
       SUM(daily_total) OVER(ORDER BY order_date) AS running_revenue_total
FROM DailyRevenue
ORDER BY order_date;
```
<img width="2542" height="1319" alt="query win 3" src="https://github.com/user-attachments/assets/412f66d3-5a31-4f5c-aaca-fb58247c2aef" />


#### 4-For each customer with more than one order, show days between the current and previous order.
```sql
WITH OrderDates AS (
    SELECT customers.customer_name, orders.order_id, orders.order_date,
           LAG(orders.order_date) OVER(PARTITION BY customers.customer_id ORDER BY orders.order_date) AS prev_order_date
    FROM customers
    JOIN orders ON customers.customer_id = orders.customer_id
)
SELECT customer_name, order_id, order_date, prev_order_date,
       (order_date - prev_order_date) AS days_between_orders
FROM OrderDates
WHERE prev_order_date IS NOT NULL;
```
<img width="2549" height="1325" alt="query win 4" src="https://github.com/user-attachments/assets/3b9a58ef-1a93-43f4-a185-0fd79ced24c7" />

