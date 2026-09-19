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

