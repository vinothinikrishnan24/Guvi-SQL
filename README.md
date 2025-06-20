-- Create a database named ecommerce
CREATE DATABASE ecommerce;
USE ecommerce;

-- Create customers table
CREATE TABLE customers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    address TEXT
);

-- Create products table
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    description TEXT
);

-- Create orders table
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT,
    order_date DATE NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

-- Insert sample data into customers table
INSERT INTO customers (name, email, address) VALUES
    ('Ravi', 'ravi11@gmail.com', 'Chennai'),
    ('Ramu', 'ramu10@gmail.com', 'Erode'),
    ('Raja', 'raja12@gmail.com', 'Coimbatore'),
    ('David', 'david13@gmail.com', 'Chennai'),
    ('Williams', 'williams14@gmail.com', 'Madurai'),
    ('Frank', 'frank15@gmail.com', 'Springfield'),
    ('Carol Williams', 'carol.williams16@gmail.com', 'Springfield'),
    ('Bob Smith', 'bob.smith17@gmail.com', 'Springfield');

-- Insert sample data into products table
INSERT INTO products (name, price, description) VALUES
    ('Laptop Pro', 1299.99, 'High-performance laptop with 16GB RAM and 512GB SSD'),
    ('Smartphone X', 799.99, 'Latest smartphone with 128GB storage and 5G'),
    ('Wireless Headphones', 199.99, 'Noise-cancelling over-ear headphones'),
    ('Smart Watch', 249.99, 'Fitness tracker with heart rate monitor'),
    ('Tablet', 499.99, '10-inch tablet with 64GB storage'),
    ('Gaming Mouse', 59.99, 'Ergonomic mouse with customizable buttons'),
    ('Bluetooth Speaker', 89.99, 'Portable speaker with 12-hour battery life');

-- Insert sample data into orders table
INSERT INTO orders (customer_id, order_date, total_amount) VALUES
    (1, '2025-06-01', 1299.99), 
    (2, '2025-06-02', 799.99),  
    (3, '2025-06-03', 449.98),  
    (4, '2025-06-04', 59.99),   
    (5, '2025-06-05', 1349.98),
    (6, '2025-06-06', 199.99),  
    (7, '2025-06-07', 89.99);   

-- Normalize the database by creating order_items table
CREATE TABLE order_items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);

-- Insert sample data into order_items table to reflect the orders
INSERT INTO order_items (order_id, product_id, quantity, price) VALUES
    (1, 1, 1, 1299.99), 
    (2, 2, 1, 799.99),  
    (3, 3, 1, 199.99),  
    (3, 4, 1, 249.99),  
    (4, 6, 1, 59.99),   
    (5, 1, 1, 1299.99), 
    (5, 6, 1, 59.99),   
    (6, 3, 1, 199.99),  
    (7, 7, 1, 89.99);  

-- Recalculate total_amount in orders table based on order_items
UPDATE orders o
SET total_amount = (
    SELECT SUM(quantity * price)
    FROM order_items oi
    WHERE oi.order_id = o.id
);

-- Retrieve all customers who have placed an order in the last 30 days
SELECT c.*
FROM customers c
JOIN orders o ON c.id = o.customer_id
WHERE o.order_date >= CURDATE() - INTERVAL 30 DAY;

-- Get the total amount of all orders placed by each customer
SELECT c.id, c.name, SUM(o.total_amount) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name
ORDER BY total_spent DESC;

-- Update the price of Product C (Wireless Headphones is Product C)
UPDATE products
SET price = 45.00
WHERE name = 'Wireless Headphones';

-- Add a new column discount to the products table
ALTER TABLE products
ADD discount DECIMAL(5, 2) DEFAULT 0.00;

-- Retrieve the top 3 products with the highest price
SELECT name, price
FROM products
ORDER BY price DESC
LIMIT 3;

-- Get the names of customers who have ordered Product A (Laptop Pro is Product A)
SELECT DISTINCT c.name
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE p.name = 'Laptop Pro';

-- Join the orders and customers tables to retrieve the customer's name and order date for each order
SELECT c.name, o.order_date
FROM orders o
JOIN customers c ON o.customer_id = c.id;

-- Retrieve the orders with a total amount greater than 150.00
SELECT *
FROM orders
WHERE total_amount > 150.00;

-- Retrieve the average total of all orders
SELECT ROUND(AVG(total_amount), 2) AS avg_total
FROM orders;
