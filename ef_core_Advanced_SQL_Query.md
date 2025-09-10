# Mastering Advanced SQL Query Techniques

## Introduction

Advanced SQL query techniques empower developers to manage complex data operations, optimize performance, and protect database integrity and security. Mastery of these skills is critical for efficient, reliable, and scalable database systems.

## Advanced Filtering and Query Techniques

### Intent
Master sophisticated filtering techniques to precisely extract and manipulate data from complex datasets, enabling dynamic data analysis and reporting.

### Key Techniques

#### Comparison and Logical Operators
**Intent**: Build complex filtering conditions that can handle multiple criteria and edge cases.

```sql
-- Example: Find employees with salary between 50K-100K in specific departments
SELECT employee_id, name, salary, department
FROM employees 
WHERE salary >= 50000 
  AND salary <= 100000 
  AND department IN ('Engineering', 'Marketing', 'Sales')
  AND hire_date > '2020-01-01';
```

#### Pattern Matching with LIKE
**Intent**: Search for partial matches and patterns in text data for flexible querying.

```sql
-- Example: Find customers with Gmail addresses or names starting with 'J'
SELECT customer_id, name, email
FROM customers
WHERE email LIKE '%@gmail.com'
   OR name LIKE 'J%'
   OR phone LIKE '555-%';
```

#### Dynamic Classification with CASE
**Intent**: Create calculated fields and categorize data on-the-fly based on business logic.

```sql
-- Example: Categorize products by price range and calculate discounts
SELECT 
    product_id,
    product_name,
    price,
    CASE 
        WHEN price < 50 THEN 'Budget'
        WHEN price BETWEEN 50 AND 200 THEN 'Mid-Range'
        WHEN price > 200 THEN 'Premium'
        ELSE 'Unclassified'
    END AS price_category,
    CASE 
        WHEN price > 100 THEN price * 0.9
        ELSE price
    END AS discounted_price
FROM products;
```

### Advanced Aggregation and Filtering

#### Intent
Perform complex data summarization and filter aggregate results to generate meaningful business insights.

```sql
-- Example: Find departments with average salary > 75K and more than 5 employees
SELECT 
    department,
    COUNT(*) as employee_count,
    AVG(salary) as avg_salary,
    MIN(salary) as min_salary,
    MAX(salary) as max_salary
FROM employees
WHERE status = 'Active'
GROUP BY department
HAVING COUNT(*) > 5 
   AND AVG(salary) > 75000
ORDER BY avg_salary DESC;
```

## Subqueries and Common Table Expressions (CTEs)

### Intent
Break down complex queries into manageable, readable components while enabling advanced data relationships and calculations.

### Subqueries in Different Clauses

#### Subquery in WHERE Clause
**Intent**: Filter main query results based on conditions from related tables.

```sql
-- Example: Find employees earning more than their department average
SELECT employee_id, name, salary, department
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department = e1.department
);
```

#### Subquery in SELECT Clause
**Intent**: Add calculated columns based on related data.

```sql
-- Example: Show customers with their total order count and value
SELECT 
    c.customer_id,
    c.name,
    (SELECT COUNT(*) FROM orders o WHERE o.customer_id = c.customer_id) as total_orders,
    (SELECT COALESCE(SUM(total_amount), 0) FROM orders o WHERE o.customer_id = c.customer_id) as total_spent
FROM customers c;
```

### Common Table Expressions (CTEs)

#### Intent
Create reusable, temporary result sets that improve query organization and enable recursive operations.

```sql
-- Example: Calculate running totals and rank customers by spending
WITH customer_spending AS (
    SELECT 
        customer_id,
        SUM(total_amount) as total_spent,
        COUNT(*) as order_count
    FROM orders
    WHERE order_date >= '2024-01-01'
    GROUP BY customer_id
),
customer_rankings AS (
    SELECT 
        customer_id,
        total_spent,
        order_count,
        ROW_NUMBER() OVER (ORDER BY total_spent DESC) as spending_rank,
        NTILE(4) OVER (ORDER BY total_spent) as quartile
    FROM customer_spending
)
SELECT 
    c.name,
    cr.total_spent,
    cr.order_count,
    cr.spending_rank,
    CASE cr.quartile
        WHEN 4 THEN 'Top Tier'
        WHEN 3 THEN 'High Value'
        WHEN 2 THEN 'Regular'
        ELSE 'New Customer'
    END as customer_tier
FROM customer_rankings cr
JOIN customers c ON cr.customer_id = c.customer_id
WHERE cr.spending_rank <= 100;
```

#### Recursive CTE Example
**Intent**: Handle hierarchical data structures like organizational charts or category trees.

```sql
-- Example: Build employee hierarchy with levels
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: Top-level managers
    SELECT 
        employee_id,
        name,
        manager_id,
        0 as level,
        CAST(name AS VARCHAR(1000)) as hierarchy_path
    FROM employees 
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: Employees with managers
    SELECT 
        e.employee_id,
        e.name,
        e.manager_id,
        eh.level + 1,
        CONCAT(eh.hierarchy_path, ' > ', e.name)
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT 
    employee_id,
    REPEAT('  ', level) || name as indented_name,
    level,
    hierarchy_path
FROM employee_hierarchy
ORDER BY hierarchy_path;
```

## Performance Tuning and Optimization

### Intent
Ensure queries execute efficiently as data volumes grow, maintaining system responsiveness and resource utilization.

### Key Optimization Strategies

#### Early Filtering and Indexing
**Intent**: Reduce dataset size as early as possible and leverage database indexes effectively.

```sql
-- Example: Optimized query with proper filtering order
-- GOOD: Filter first, then join
SELECT 
    o.order_id,
    c.name,
    o.total_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date >= '2024-01-01'  -- Filter early
  AND o.status = 'completed'        -- Use indexed columns
  AND c.region = 'North America'
ORDER BY o.order_date DESC
LIMIT 100;

-- Create supporting indexes
-- CREATE INDEX idx_orders_date_status ON orders(order_date, status);
-- CREATE INDEX idx_customers_region ON customers(region);
```

#### Query Restructuring
**Intent**: Transform complex nested queries into more efficient forms.

```sql
-- Example: Replace correlated subquery with window function
-- LESS EFFICIENT: Correlated subquery
SELECT 
    employee_id,
    name,
    salary,
    (SELECT AVG(salary) FROM employees e2 WHERE e2.department = e1.department) as dept_avg
FROM employees e1;

-- MORE EFFICIENT: Window function
SELECT 
    employee_id,
    name,
    salary,
    AVG(salary) OVER (PARTITION BY department) as dept_avg
FROM employees;
```

## Stored Procedures and Functions

### Intent
Encapsulate complex business logic, improve code reusability, and enhance database security through controlled access patterns.

### Stored Procedure Example
**Intent**: Handle complex multi-step operations with transaction control.

```sql
-- Example: Process customer order with inventory management
DELIMITER //
CREATE PROCEDURE ProcessOrder(
    IN p_customer_id INT,
    IN p_product_id INT,
    IN p_quantity INT,
    OUT p_order_id INT,
    OUT p_status VARCHAR(50)
)
BEGIN
    DECLARE v_available_stock INT DEFAULT 0;
    DECLARE v_product_price DECIMAL(10,2) DEFAULT 0;
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_status = 'ERROR: Transaction failed';
    END;
    
    START TRANSACTION;
    
    -- Check inventory
    SELECT stock_quantity, price
    INTO v_available_stock, v_product_price
    FROM products 
    WHERE product_id = p_product_id;
    
    IF v_available_stock >= p_quantity THEN
        -- Create order
        INSERT INTO orders (customer_id, order_date, total_amount, status)
        VALUES (p_customer_id, NOW(), v_product_price * p_quantity, 'pending');
        
        SET p_order_id = LAST_INSERT_ID();
        
        -- Add order items
        INSERT INTO order_items (order_id, product_id, quantity, unit_price)
        VALUES (p_order_id, p_product_id, p_quantity, v_product_price);
        
        -- Update inventory
        UPDATE products 
        SET stock_quantity = stock_quantity - p_quantity
        WHERE product_id = p_product_id;
        
        COMMIT;
        SET p_status = 'SUCCESS: Order created';
    ELSE
        SET p_status = 'ERROR: Insufficient inventory';
        ROLLBACK;
    END IF;
END //
DELIMITER ;

-- Usage example
CALL ProcessOrder(123, 456, 2, @order_id, @status);
SELECT @order_id, @status;
```

### User-Defined Function Example
**Intent**: Create reusable calculations and data transformations.

```sql
-- Example: Calculate customer lifetime value
DELIMITER //
CREATE FUNCTION CalculateCustomerLTV(p_customer_id INT)
RETURNS DECIMAL(10,2)
READS SQL DATA
DETERMINISTIC
BEGIN
    DECLARE v_total_spent DECIMAL(10,2) DEFAULT 0;
    DECLARE v_months_active INT DEFAULT 0;
    DECLARE v_avg_monthly DECIMAL(10,2) DEFAULT 0;
    DECLARE v_ltv DECIMAL(10,2) DEFAULT 0;
    
    -- Get total spending
    SELECT COALESCE(SUM(total_amount), 0)
    INTO v_total_spent
    FROM orders
    WHERE customer_id = p_customer_id AND status = 'completed';
    
    -- Calculate months active
    SELECT DATEDIFF(MAX(order_date), MIN(order_date)) / 30
    INTO v_months_active
    FROM orders
    WHERE customer_id = p_customer_id;
    
    -- Calculate average monthly spend
    IF v_months_active > 0 THEN
        SET v_avg_monthly = v_total_spent / v_months_active;
        -- Assume 24-month LTV projection
        SET v_ltv = v_avg_monthly * 24;
    ELSE
        SET v_ltv = v_total_spent;
    END IF;
    
    RETURN v_ltv;
END //
DELIMITER ;

-- Usage example
SELECT 
    customer_id,
    name,
    CalculateCustomerLTV(customer_id) as projected_ltv
FROM customers
ORDER BY projected_ltv DESC
LIMIT 10;
```

## SQL Security Best Practices

### Intent
Protect sensitive data and prevent security vulnerabilities while maintaining system functionality and compliance requirements.

### Parameterized Queries
**Intent**: Prevent SQL injection attacks through proper parameter handling.

```sql
-- Example: Secure user authentication query (using prepared statements)
-- VULNERABLE: Direct string concatenation
-- SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'

-- SECURE: Parameterized query
PREPARE stmt FROM 'SELECT user_id, username, role FROM users WHERE username = ? AND password_hash = SHA2(?, 256) AND status = ?';
SET @username = 'john_doe';
SET @password = 'user_password';
SET @status = 'active';
EXECUTE stmt USING @username, @password, @status;
DEALLOCATE PREPARE stmt;
```

### Access Control and Auditing
**Intent**: Implement role-based security and maintain audit trails for compliance.

```sql
-- Example: Create role-based access control
-- Create roles
CREATE ROLE 'app_read_only';
CREATE ROLE 'app_data_entry';
CREATE ROLE 'app_manager';

-- Grant permissions to roles
GRANT SELECT ON company_db.customers TO 'app_read_only';
GRANT SELECT ON company_db.orders TO 'app_read_only';

GRANT SELECT, INSERT, UPDATE ON company_db.orders TO 'app_data_entry';
GRANT SELECT ON company_db.customers TO 'app_data_entry';

GRANT ALL PRIVILEGES ON company_db.* TO 'app_manager';

-- Create users and assign roles
CREATE USER 'analyst'@'%' IDENTIFIED BY 'secure_password123';
CREATE USER 'clerk'@'%' IDENTIFIED BY 'secure_password456';
CREATE USER 'manager'@'%' IDENTIFIED BY 'secure_password789';

GRANT 'app_read_only' TO 'analyst'@'%';
GRANT 'app_data_entry' TO 'clerk'@'%';
GRANT 'app_manager' TO 'manager'@'%';

-- Audit table for tracking changes
CREATE TABLE audit_log (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(50),
    operation VARCHAR(10),
    user_name VARCHAR(50),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    old_values JSON,
    new_values JSON
);

-- Example audit trigger
DELIMITER //
CREATE TRIGGER audit_customer_changes
AFTER UPDATE ON customers
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (table_name, operation, user_name, old_values, new_values)
    VALUES (
        'customers',
        'UPDATE',
        USER(),
        JSON_OBJECT('name', OLD.name, 'email', OLD.email, 'status', OLD.status),
        JSON_OBJECT('name', NEW.name, 'email', NEW.email, 'status', NEW.status)
    );
END //
DELIMITER ;
```

### Data Encryption and Masking
**Intent**: Protect sensitive information both at rest and in transit.

```sql
-- Example: Encrypt sensitive data
-- Create table with encrypted columns
CREATE TABLE customer_secure (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email_encrypted VARBINARY(255),
    ssn_encrypted VARBINARY(255),
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert encrypted data
INSERT INTO customer_secure (name, email_encrypted, ssn_encrypted)
VALUES (
    'John Doe',
    AES_ENCRYPT('john@example.com', 'encryption_key_123'),
    AES_ENCRYPT('123-45-6789', 'encryption_key_123')
);

-- Query with decryption (only for authorized users)
SELECT 
    customer_id,
    name,
    AES_DECRYPT(email_encrypted, 'encryption_key_123') as email,
    CONCAT('***-**-', RIGHT(AES_DECRYPT(ssn_encrypted, 'encryption_key_123'), 4)) as masked_ssn
FROM customer_secure;

-- Create view for data masking
CREATE VIEW customer_masked AS
SELECT 
    customer_id,
    name,
    CONCAT(LEFT(AES_DECRYPT(email_encrypted, 'encryption_key_123'), 3), '***@***.com') as email_masked,
    CONCAT('***-**-', RIGHT(AES_DECRYPT(ssn_encrypted, 'encryption_key_123'), 4)) as ssn_masked,
    created_date
FROM customer_secure;
```

## Advanced Query Patterns

### Intent
Demonstrate sophisticated query techniques for complex business scenarios and data analysis requirements.

### Window Functions and Analytics
**Intent**: Perform advanced analytical calculations across result sets.

```sql
-- Example: Comprehensive sales analysis with rankings and running totals
WITH monthly_sales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') as month,
        salesperson_id,
        SUM(total_amount) as monthly_total
    FROM orders
    WHERE order_date >= '2024-01-01'
    GROUP BY DATE_FORMAT(order_date, '%Y-%m'), salesperson_id
)
SELECT 
    month,
    salesperson_id,
    monthly_total,
    -- Ranking within each month
    ROW_NUMBER() OVER (PARTITION BY month ORDER BY monthly_total DESC) as monthly_rank,
    -- Running total for each salesperson
    SUM(monthly_total) OVER (PARTITION BY salesperson_id ORDER BY month) as running_total,
    -- Moving 3-month average
    AVG(monthly_total) OVER (
        PARTITION BY salesperson_id 
        ORDER BY month 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as moving_avg_3m,
    -- Percentage of monthly total sales
    ROUND(
        monthly_total * 100.0 / SUM(monthly_total) OVER (PARTITION BY month), 2
    ) as pct_of_monthly_total
FROM monthly_sales
ORDER BY month, monthly_rank;
```

### Pivot Operations
**Intent**: Transform row data into columnar format for reporting and analysis.

```sql
-- Example: Create sales pivot table by quarter and region
SELECT 
    region,
    SUM(CASE WHEN QUARTER(order_date) = 1 THEN total_amount ELSE 0 END) as Q1_sales,
    SUM(CASE WHEN QUARTER(order_date) = 2 THEN total_amount ELSE 0 END) as Q2_sales,
    SUM(CASE WHEN QUARTER(order_date) = 3 THEN total_amount ELSE 0 END) as Q3_sales,
    SUM(CASE WHEN QUARTER(order_date) = 4 THEN total_amount ELSE 0 END) as Q4_sales,
    SUM(total_amount) as total_annual_sales
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE YEAR(order_date) = 2024
GROUP BY region
ORDER BY total_annual_sales DESC;
```

## Conclusion

Mastering advanced SQL techniques allows developers to write smarter, faster, and more secure queries. Through effective querying, careful optimization, reliable transaction handling, and strong security practices, developers can meet the demands of real-world applications with confidence and precision.

### Key Takeaways

- **Use appropriate techniques for each scenario**: Choose between subqueries, CTEs, and joins based on complexity and reusability needs
- **Optimize early and often**: Apply filters early, use indexes effectively, and monitor query performance
- **Prioritize security**: Always use parameterized queries, implement proper access controls, and protect sensitive data
- **Leverage advanced features**: Window functions, stored procedures, and CTEs can significantly improve code quality and performance
- **Plan for scalability**: Design queries and database structures that can handle growing data volumes and user loads
