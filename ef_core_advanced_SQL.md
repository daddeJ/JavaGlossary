# SQL Security & Validation Commands: API Scenarios & Solutions

## 🔒 Security-Related Commands

### 1. **GRANT** - Role-Based API Access Control

**Intent:** Control what data different API users can access based on their roles.

**Problem Scenario:**
An e-commerce API serves multiple user types: customers, store managers, and admins. Each should only access data relevant to their role.

**API Context:**
```javascript
// API endpoint: GET /api/orders
app.get('/api/orders', authenticateUser, (req, res) => {
    const userRole = req.user.role;
    // Different database users for different roles
    const dbConnection = getConnectionForRole(userRole);
    // Query executes with role-specific permissions
});
```

**SQL Solution:**
```sql
-- Create role-specific database users
CREATE USER customer_api IDENTIFIED BY 'SecurePass123!';
CREATE USER manager_api IDENTIFIED BY 'SecurePass456!';
CREATE USER admin_api IDENTIFIED BY 'SecurePass789!';

-- Grant specific permissions per role
GRANT SELECT ON Orders TO customer_api;
GRANT SELECT ON Products TO customer_api;

GRANT SELECT, INSERT, UPDATE ON Orders TO manager_api;
GRANT SELECT, INSERT, UPDATE ON Products TO manager_api;
GRANT SELECT ON Analytics TO manager_api;

GRANT ALL PRIVILEGES ON *.* TO admin_api;
```

**Result:** API automatically enforces data access boundaries at the database level, preventing privilege escalation attacks.

---

### 2. **REVOKE** - Dynamic Permission Management

**Intent:** Temporarily or permanently remove access when user roles change or security incidents occur.

**Problem Scenario:**
A former employee's API key is compromised, or a user's role is downgraded. Need immediate access revocation.

**API Context:**
```javascript
// API endpoint: POST /api/admin/revoke-access
app.post('/api/admin/revoke-access', requireAdmin, async (req, res) => {
    const { userId, permissions } = req.body;
    
    // Revoke database permissions
    await revokeUserPermissions(userId, permissions);
    
    // Invalidate API tokens
    await invalidateUserTokens(userId);
});
```

**SQL Solution:**
```sql
-- Emergency revocation scenario
REVOKE INSERT, UPDATE, DELETE ON Orders FROM compromised_user;
REVOKE SELECT ON CustomerData FROM compromised_user;

-- Partial revocation for role downgrade
REVOKE DELETE ON Products FROM manager_api;
REVOKE SELECT ON Analytics FROM manager_api;
```

**Result:** Immediate security response capability through API-triggered permission changes.

---

### 3. **DENY** - Explicit Security Boundaries

**Intent:** Explicitly block dangerous operations even if accidentally granted elsewhere.

**Problem Scenario:**
Financial API must never allow certain users to delete transaction records, regardless of other permissions.

**API Context:**
```javascript
// API endpoint: DELETE /api/transactions/:id
app.delete('/api/transactions/:id', authenticateUser, (req, res) => {
    // Even if user somehow gets DELETE permission,
    // DENY at database level prevents execution
    if (req.user.role === 'auditor' || req.user.role === 'viewer') {
        return res.status(403).json({ error: 'Operation denied' });
    }
});
```

**SQL Solution:**
```sql
-- Explicit denials for sensitive operations
DENY DELETE ON Transactions TO auditor_api;
DENY DELETE ON AuditLog TO ALL;  -- Nobody can delete audit logs
DENY TRUNCATE ON CustomerData TO manager_api;  -- Prevent accidental data loss
```

**Result:** Defense-in-depth security where critical operations are blocked at multiple layers.

---

### 4. **CREATE USER/ROLE** - Principle of Least Privilege

**Intent:** Create API service accounts with minimal necessary permissions.

**Problem Scenario:**
Microservices architecture where each service should only access its required data.

**API Context:**
```javascript
// Different services with different database users
const serviceConnections = {
    'user-service': createConnection('user_service_db', 'user123'),
    'order-service': createConnection('order_service_db', 'order456'),
    'payment-service': createConnection('payment_service_db', 'payment789'),
    'notification-service': createConnection('notification_service_db', 'notify101')
};
```

**SQL Solution:**
```sql
-- Service-specific users with minimal permissions
CREATE USER user_service_db IDENTIFIED BY 'UserSvc!2024';
GRANT SELECT, INSERT, UPDATE ON Users TO user_service_db;
GRANT SELECT ON UserProfiles TO user_service_db;

CREATE USER order_service_db IDENTIFIED BY 'OrderSvc!2024';
GRANT SELECT, INSERT, UPDATE ON Orders TO order_service_db;
GRANT SELECT ON Products TO order_service_db;
GRANT SELECT ON Users TO order_service_db;  -- Only for order validation

CREATE USER payment_service_db IDENTIFIED BY 'PaySvc!2024';
GRANT SELECT, INSERT ON Payments TO payment_service_db;
GRANT SELECT ON Orders TO payment_service_db;  -- Only for payment processing

CREATE USER notification_service_db IDENTIFIED BY 'NotifySvc!2024';
GRANT SELECT ON Users TO notification_service_db;  -- Only for sending notifications
GRANT INSERT ON NotificationLog TO notification_service_db;
```

**Result:** Service isolation where compromised service can't access other services' data.

---

### 5. **ALTER USER** - Security Maintenance

**Intent:** Rotate credentials and update security settings programmatically.

**Problem Scenario:**
Automated password rotation for API service accounts as part of security compliance.

**API Context:**
```javascript
// API endpoint: POST /api/admin/rotate-credentials
app.post('/api/admin/rotate-credentials', requireSuperAdmin, async (req, res) => {
    const { serviceAccount } = req.body;
    
    const newPassword = generateSecurePassword();
    await rotateServiceAccountPassword(serviceAccount, newPassword);
    
    // Update connection pools with new credentials
    await updateConnectionPools(serviceAccount, newPassword);
});
```

**SQL Solution:**
```sql
-- Automated credential rotation
ALTER USER order_service_db IDENTIFIED BY 'NewSecurePass!2024';
ALTER USER payment_service_db IDENTIFIED BY 'AnotherNewPass!2024';

-- Update security settings
ALTER USER user_service_db 
    PASSWORD EXPIRE INTERVAL 90 DAY
    FAILED_LOGIN_ATTEMPTS 3
    PASSWORD_LOCK_TIME 30;
```

**Result:** Automated security maintenance through API-driven credential management.

---

### 6. **Views** - API Data Security Layer

**Intent:** Expose only necessary data through API endpoints while hiding sensitive information.

**Problem Scenario:**
Public API should show product information but hide internal cost data and supplier details.

**API Context:**
```javascript
// API endpoint: GET /api/public/products
app.get('/api/public/products', (req, res) => {
    // Query the view instead of the raw table
    const query = 'SELECT * FROM ProductPublicView WHERE active = 1';
    db.query(query, (err, results) => {
        res.json(results);
    });
});
```

**SQL Solution:**
```sql
-- Secure view for public API
CREATE VIEW ProductPublicView AS
SELECT 
    ProductId,
    ProductName,
    Category,
    Description,
    PublicPrice,
    ImageUrl,
    Rating,
    CASE WHEN Stock > 0 THEN 'In Stock' ELSE 'Out of Stock' END as Availability
FROM Products 
WHERE IsPublic = 1 AND IsDeleted = 0;

-- Internal view for staff API
CREATE VIEW ProductInternalView AS
SELECT 
    ProductId,
    ProductName,
    Category,
    CostPrice,
    PublicPrice,
    Margin,
    Stock,
    SupplierId,
    LastUpdated
FROM Products 
WHERE IsDeleted = 0;

-- Grant appropriate access
GRANT SELECT ON ProductPublicView TO public_api_user;
GRANT SELECT ON ProductInternalView TO staff_api_user;
REVOKE SELECT ON Products FROM public_api_user;  -- Force use of view
```

**Result:** API automatically serves filtered, secure data without application-level filtering complexity.

---

### 7. **Stored Procedures** - API Security Gateway

**Intent:** Encapsulate complex business logic and security checks in database procedures.

**Problem Scenario:**
Banking API needs to transfer money with multiple validation checks and audit logging.

**API Context:**
```javascript
// API endpoint: POST /api/transfer
app.post('/api/transfer', authenticateUser, async (req, res) => {
    const { fromAccount, toAccount, amount } = req.body;
    const userId = req.user.id;
    
    try {
        // Call stored procedure instead of multiple queries
        const result = await db.execute('CALL TransferFunds(?, ?, ?, ?, ?)', 
            [userId, fromAccount, toAccount, amount, req.ip]);
        
        if (result.success) {
            res.json({ message: 'Transfer completed', transactionId: result.transactionId });
        } else {
            res.status(400).json({ error: result.errorMessage });
        }
    } catch (error) {
        res.status(500).json({ error: 'Transfer failed' });
    }
});
```

**SQL Solution:**
```sql
DELIMITER //
CREATE PROCEDURE TransferFunds(
    IN p_UserId INT,
    IN p_FromAccount VARCHAR(20),
    IN p_ToAccount VARCHAR(20),
    IN p_Amount DECIMAL(15,2),
    IN p_IpAddress VARCHAR(45),
    OUT p_Success BOOLEAN,
    OUT p_TransactionId VARCHAR(50),
    OUT p_ErrorMessage VARCHAR(500)
)
BEGIN
    DECLARE v_FromBalance DECIMAL(15,2);
    DECLARE v_DailyLimit DECIMAL(15,2);
    DECLARE v_DailyTransferred DECIMAL(15,2);
    DECLARE EXIT HANDLER FOR SQLEXCEPTION 
    BEGIN
        ROLLBACK;
        SET p_Success = FALSE;
        SET p_ErrorMessage = 'Database error occurred';
    END;

    START TRANSACTION;
    
    -- Verify account ownership
    IF NOT EXISTS (SELECT 1 FROM Accounts WHERE AccountNumber = p_FromAccount AND UserId = p_UserId) THEN
        SET p_Success = FALSE;
        SET p_ErrorMessage = 'Source account not owned by user';
        ROLLBACK;
    END IF;
    
    -- Check balance
    SELECT Balance INTO v_FromBalance 
    FROM Accounts WHERE AccountNumber = p_FromAccount FOR UPDATE;
    
    IF v_FromBalance < p_Amount THEN
        SET p_Success = FALSE;
        SET p_ErrorMessage = 'Insufficient funds';
        ROLLBACK;
    END IF;
    
    -- Check daily transfer limit
    SELECT DailyTransferLimit INTO v_DailyLimit 
    FROM Users WHERE UserId = p_UserId;
    
    SELECT COALESCE(SUM(Amount), 0) INTO v_DailyTransferred
    FROM Transactions 
    WHERE FromAccount = p_FromAccount 
    AND DATE(TransactionDate) = CURDATE();
    
    IF (v_DailyTransferred + p_Amount) > v_DailyLimit THEN
        SET p_Success = FALSE;
        SET p_ErrorMessage = 'Daily transfer limit exceeded';
        ROLLBACK;
    END IF;
    
    -- Perform transfer
    SET p_TransactionId = CONCAT('TXN_', UUID());
    
    UPDATE Accounts SET Balance = Balance - p_Amount 
    WHERE AccountNumber = p_FromAccount;
    
    UPDATE Accounts SET Balance = Balance + p_Amount 
    WHERE AccountNumber = p_ToAccount;
    
    -- Log transaction
    INSERT INTO Transactions (TransactionId, UserId, FromAccount, ToAccount, Amount, IpAddress, TransactionDate)
    VALUES (p_TransactionId, p_UserId, p_FromAccount, p_ToAccount, p_Amount, p_IpAddress, NOW());
    
    -- Audit log
    INSERT INTO AuditLog (UserId, Action, Details, IpAddress, Timestamp)
    VALUES (p_UserId, 'TRANSFER', CONCAT('Transferred $', p_Amount, ' from ', p_FromAccount, ' to ', p_ToAccount), p_IpAddress, NOW());
    
    COMMIT;
    SET p_Success = TRUE;
    SET p_ErrorMessage = NULL;
END//
DELIMITER ;

-- Grant execute permission
GRANT EXECUTE ON PROCEDURE TransferFunds TO banking_api_user;
```

**Result:** Atomic, secure, audited transactions with business logic enforced at database level.

---

## ✅ Validation & Data Integrity Commands

### 1. **NOT NULL** - API Input Validation

**Intent:** Ensure critical data is always provided through API calls.

**Problem Scenario:**
User registration API must ensure essential fields are never empty.

**API Context:**
```javascript
// API endpoint: POST /api/users/register
app.post('/api/users/register', async (req, res) => {
    const { email, password, firstName, lastName } = req.body;
    
    try {
        // Database will enforce NOT NULL constraints
        const userId = await createUser({ email, password, firstName, lastName });
        res.status(201).json({ userId, message: 'User created successfully' });
    } catch (error) {
        if (error.code === 'ER_BAD_NULL_ERROR') {
            res.status(400).json({ error: 'Required fields missing' });
        }
    }
});
```

**SQL Solution:**
```sql
CREATE TABLE Users (
    UserId INT AUTO_INCREMENT PRIMARY KEY,
    Email VARCHAR(255) NOT NULL,
    PasswordHash VARCHAR(255) NOT NULL,
    FirstName VARCHAR(100) NOT NULL,
    LastName VARCHAR(100) NOT NULL,
    CreatedAt TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    IsActive BOOLEAN NOT NULL DEFAULT TRUE
);
```

**Result:** Database-level validation prevents incomplete user records from API submissions.

---

### 2. **UNIQUE** - Prevent API Data Duplication

**Intent:** Ensure unique identifiers and prevent duplicate resource creation.

**Problem Scenario:**
Product API should prevent creating products with duplicate SKUs or email registration with existing emails.

**API Context:**
```javascript
// API endpoint: POST /api/products
app.post('/api/products', authenticateUser, async (req, res) => {
    const { sku, name, price, category } = req.body;
    
    try {
        const productId = await createProduct({ sku, name, price, category });
        res.status(201).json({ productId, message: 'Product created' });
    } catch (error) {
        if (error.code === 'ER_DUP_ENTRY') {
            res.status(409).json({ error: 'Product with this SKU already exists' });
        }
    }
});
```

**SQL Solution:**
```sql
CREATE TABLE Products (
    ProductId INT AUTO_INCREMENT PRIMARY KEY,
    SKU VARCHAR(50) NOT NULL UNIQUE,
    Name VARCHAR(200) NOT NULL,
    Price DECIMAL(10,2) NOT NULL,
    Category VARCHAR(100),
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_sku (SKU),  -- Optimize unique constraint checking
    INDEX idx_category (Category)
);

CREATE TABLE Users (
    UserId INT AUTO_INCREMENT PRIMARY KEY,
    Email VARCHAR(255) NOT NULL UNIQUE,
    Username VARCHAR(50) UNIQUE,
    PasswordHash VARCHAR(255) NOT NULL
);
```

**Result:** Automatic duplicate prevention with clear API error responses.

---

### 3. **PRIMARY KEY** - API Resource Identification

**Intent:** Ensure every API resource has a unique, reliable identifier.

**Problem Scenario:**
REST API needs consistent resource identification for CRUD operations.

**API Context:**
```javascript
// API endpoints using primary keys
app.get('/api/orders/:orderId', async (req, res) => {
    const order = await getOrderById(req.params.orderId);
    if (!order) return res.status(404).json({ error: 'Order not found' });
    res.json(order);
});

app.put('/api/orders/:orderId', async (req, res) => {
    const updated = await updateOrder(req.params.orderId, req.body);
    res.json(updated);
});

app.delete('/api/orders/:orderId', async (req, res) => {
    await deleteOrder(req.params.orderId);
    res.status(204).send();
});
```

**SQL Solution:**
```sql
CREATE TABLE Orders (
    OrderId INT AUTO_INCREMENT PRIMARY KEY,  -- Reliable API resource identifier
    CustomerId INT NOT NULL,
    OrderDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    TotalAmount DECIMAL(12,2) NOT NULL,
    Status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') NOT NULL
);

CREATE TABLE OrderItems (
    OrderItemId INT AUTO_INCREMENT PRIMARY KEY,
    OrderId INT NOT NULL,
    ProductId INT NOT NULL,
    Quantity INT NOT NULL,
    Price DECIMAL(10,2) NOT NULL
);
```

**Result:** Reliable, consistent API resource identification and manipulation.

---

### 4. **FOREIGN KEY** - API Data Consistency

**Intent:** Maintain referential integrity across related API resources.

**Problem Scenario:**
Order API must ensure orders reference valid customers and products.

**API Context:**
```javascript
// API endpoint: POST /api/orders
app.post('/api/orders', authenticateUser, async (req, res) => {
    const { customerId, items } = req.body;
    
    try {
        // Foreign key constraints will validate references
        const orderId = await createOrder({ customerId, items });
        res.status(201).json({ orderId, message: 'Order created' });
    } catch (error) {
        if (error.code === 'ER_NO_REFERENCED_ROW_2') {
            res.status(400).json({ error: 'Invalid customer or product reference' });
        }
    }
});
```

**SQL Solution:**
```sql
CREATE TABLE Customers (
    CustomerId INT AUTO_INCREMENT PRIMARY KEY,
    Email VARCHAR(255) NOT NULL UNIQUE,
    FirstName VARCHAR(100) NOT NULL,
    LastName VARCHAR(100) NOT NULL
);

CREATE TABLE Products (
    ProductId INT AUTO_INCREMENT PRIMARY KEY,
    Name VARCHAR(200) NOT NULL,
    Price DECIMAL(10,2) NOT NULL,
    IsActive BOOLEAN DEFAULT TRUE
);

CREATE TABLE Orders (
    OrderId INT AUTO_INCREMENT PRIMARY KEY,
    CustomerId INT NOT NULL,
    OrderDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)
        ON UPDATE CASCADE
        ON DELETE RESTRICT  -- Prevent deleting customers with orders
);

CREATE TABLE OrderItems (
    OrderItemId INT AUTO_INCREMENT PRIMARY KEY,
    OrderId INT NOT NULL,
    ProductId INT NOT NULL,
    Quantity INT NOT NULL CHECK (Quantity > 0),
    
    FOREIGN KEY (OrderId) REFERENCES Orders(OrderId)
        ON UPDATE CASCADE
        ON DELETE CASCADE,  -- Delete items when order is deleted
    
    FOREIGN KEY (ProductId) REFERENCES Products(ProductId)
        ON UPDATE CASCADE
        ON DELETE RESTRICT  -- Prevent deleting products in active orders
);
```

**Result:** API operations maintain data consistency automatically through database constraints.

---

### 5. **CHECK** - API Business Rule Enforcement

**Intent:** Enforce business rules and data validation at the database level.

**Problem Scenario:**
E-commerce API must ensure valid prices, quantities, and business rules.

**API Context:**
```javascript
// API endpoint: POST /api/products
app.post('/api/products', authenticateUser, async (req, res) => {
    const { name, price, discountPercent, stockQuantity } = req.body;
    
    try {
        const productId = await createProduct({ name, price, discountPercent, stockQuantity });
        res.status(201).json({ productId });
    } catch (error) {
        if (error.code === 'ER_CHECK_CONSTRAINT_VIOLATED') {
            res.status(400).json({ error: 'Invalid product data - check price and discount values' });
        }
    }
});
```

**SQL Solution:**
```sql
CREATE TABLE Products (
    ProductId INT AUTO_INCREMENT PRIMARY KEY,
    Name VARCHAR(200) NOT NULL,
    Price DECIMAL(10,2) NOT NULL CHECK (Price > 0),
    DiscountPercent DECIMAL(5,2) DEFAULT 0 CHECK (DiscountPercent >= 0 AND DiscountPercent <= 100),
    StockQuantity INT NOT NULL DEFAULT 0 CHECK (StockQuantity >= 0),
    Rating DECIMAL(3,2) DEFAULT 0 CHECK (Rating >= 0 AND Rating <= 5),
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE Orders (
    OrderId INT AUTO_INCREMENT PRIMARY KEY,
    CustomerId INT NOT NULL,
    TotalAmount DECIMAL(12,2) NOT NULL CHECK (TotalAmount > 0),
    Status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') NOT NULL,
    OrderDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ShippingDate TIMESTAMP NULL,
    
    CHECK (ShippingDate IS NULL OR ShippingDate >= OrderDate)  -- Shipping can't be before order
);

CREATE TABLE Employees (
    EmployeeId INT AUTO_INCREMENT PRIMARY KEY,
    Email VARCHAR(255) NOT NULL UNIQUE,
    Salary DECIMAL(10,2) CHECK (Salary > 0),
    Age INT CHECK (Age >= 18 AND Age <= 100),
    Department VARCHAR(50) NOT NULL,
    HireDate DATE NOT NULL CHECK (HireDate <= CURDATE())
);
```

**Result:** Business rules enforced consistently across all API operations.

---

### 6. **DEFAULT** - API Data Consistency

**Intent:** Provide consistent default values for optional API fields.

**Problem Scenario:**
User registration API should set sensible defaults for optional fields.

**API Context:**
```javascript
// API endpoint: POST /api/users/register
app.post('/api/users/register', async (req, res) => {
    // Optional fields can be omitted, defaults will be applied
    const userData = {
        email: req.body.email,
        password: req.body.password,
        firstName: req.body.firstName,
        lastName: req.body.lastName,
        // phoneNumber, timezone, newsletter can be omitted
        phoneNumber: req.body.phoneNumber,
        timezone: req.body.timezone,
        newsletter: req.body.newsletter
    };
    
    const userId = await createUser(userData);
    res.status(201).json({ userId });
});
```

**SQL Solution:**
```sql
CREATE TABLE Users (
    UserId INT AUTO_INCREMENT PRIMARY KEY,
    Email VARCHAR(255) NOT NULL UNIQUE,
    PasswordHash VARCHAR(255) NOT NULL,
    FirstName VARCHAR(100) NOT NULL,
    LastName VARCHAR(100) NOT NULL,
    PhoneNumber VARCHAR(20) NULL,
    Timezone VARCHAR(50) DEFAULT 'UTC',
    IsActive BOOLEAN DEFAULT TRUE,
    NewsletterSubscribed BOOLEAN DEFAULT FALSE,
    EmailVerified BOOLEAN DEFAULT FALSE,
    LoginAttempts INT DEFAULT 0,
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    LastLoginAt TIMESTAMP NULL,
    ProfileCompleteness DECIMAL(5,2) DEFAULT 25.0  -- Based on required fields filled
);

CREATE TABLE Products (
    ProductId INT AUTO_INCREMENT PRIMARY KEY,
    Name VARCHAR(200) NOT NULL,
    Description TEXT NULL,
    Price DECIMAL(10,2) NOT NULL,
    IsActive BOOLEAN DEFAULT TRUE,
    IsDiscounted BOOLEAN DEFAULT FALSE,
    StockQuantity INT DEFAULT 0,
    ViewCount INT DEFAULT 0,
    Rating DECIMAL(3,2) DEFAULT 0.0,
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE Orders (
    OrderId INT AUTO_INCREMENT PRIMARY KEY,
    CustomerId INT NOT NULL,
    Status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    PaymentStatus ENUM('pending', 'paid', 'failed', 'refunded') DEFAULT 'pending',
    ShippingMethod VARCHAR(50) DEFAULT 'standard',
    OrderDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    EstimatedDelivery DATE NULL,
    Notes TEXT NULL
);
```

**Result:** Consistent API behavior with predictable default values reducing client-side complexity.

---

### 7. **TRIGGERS** - API Automation & Auditing

**Intent:** Automate business processes and maintain audit trails for API operations.

**Problem Scenario:**
E-commerce API needs automatic inventory updates, audit logging, and business process automation.

**API Context:**
```javascript
// API endpoint: POST /api/orders
app.post('/api/orders', authenticateUser, async (req, res) => {
    const { customerId, items } = req.body;
    
    try {
        // Triggers will automatically:
        // 1. Update inventory levels
        // 2. Log audit trail
        // 3. Update customer purchase history
        // 4. Send notifications
        const orderId = await createOrder({ customerId, items });
        
        res.status(201).json({ 
            orderId, 
            message: 'Order created successfully' 
        });
    } catch (error) {
        if (error.code === 'ER_SIGNAL_EXCEPTION') {
            res.status(400).json({ error: 'Insufficient inventory' });
        }
    }
});
```

**SQL Solution:**
```sql
-- Audit log table
CREATE TABLE AuditLog (
    LogId INT AUTO_INCREMENT PRIMARY KEY,
    TableName VARCHAR(50) NOT NULL,
    RecordId INT NOT NULL,
    Action ENUM('INSERT', 'UPDATE', 'DELETE') NOT NULL,
    UserId INT NULL,
    OldValues JSON NULL,
    NewValues JSON NULL,
    Timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    IpAddress VARCHAR(45) NULL
);

-- Customer statistics table
CREATE TABLE CustomerStats (
    CustomerId INT PRIMARY KEY,
    TotalOrders INT DEFAULT 0,
    TotalSpent DECIMAL(15,2) DEFAULT 0,
    LastOrderDate TIMESTAMP NULL,
    AverageOrderValue DECIMAL(10,2) DEFAULT 0,
    FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)
);

-- Inventory management trigger
DELIMITER //
CREATE TRIGGER trg_UpdateInventory
AFTER INSERT ON OrderItems
FOR EACH ROW
BEGIN
    DECLARE v_CurrentStock INT;
    
    -- Check current inventory
    SELECT StockQuantity INTO v_CurrentStock 
    FROM Products 
    WHERE ProductId = NEW.ProductId;
    
    -- Prevent overselling
    IF v_CurrentStock < NEW.Quantity THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Insufficient inventory for this product';
    END IF;
    
    -- Update inventory
    UPDATE Products 
    SET StockQuantity = StockQuantity - NEW.Quantity,
        LastSoldDate = CURRENT_TIMESTAMP
    WHERE ProductId = NEW.ProductId;
    
    -- Alert if low stock
    IF (v_CurrentStock - NEW.Quantity) <= 5 THEN
        INSERT INTO InventoryAlerts (ProductId, CurrentStock, AlertType, CreatedAt)
        VALUES (NEW.ProductId, (v_CurrentStock - NEW.Quantity), 'LOW_STOCK', NOW());
    END IF;
END//

-- Customer statistics update trigger
CREATE TRIGGER trg_UpdateCustomerStats
AFTER INSERT ON Orders
FOR EACH ROW
BEGIN
    INSERT INTO CustomerStats (CustomerId, TotalOrders, TotalSpent, LastOrderDate, AverageOrderValue)
    VALUES (NEW.CustomerId, 1, NEW.TotalAmount, NEW.OrderDate, NEW.TotalAmount)
    ON DUPLICATE KEY UPDATE
        TotalOrders = TotalOrders + 1,
        TotalSpent = TotalSpent + NEW.TotalAmount,
        LastOrderDate = NEW.OrderDate,
        AverageOrderValue = (TotalSpent + NEW.TotalAmount) / (TotalOrders + 1);
END//

-- Comprehensive audit trigger for user changes
CREATE TRIGGER trg_AuditUserChanges
AFTER UPDATE ON Users
FOR EACH ROW
BEGIN
    INSERT INTO AuditLog (TableName, RecordId, Action, UserId, OldValues, NewValues, Timestamp)
    VALUES (
        'Users',
        NEW.UserId,
        'UPDATE',
        NEW.UserId,
        JSON_OBJECT(
            'Email', OLD.Email,
            'FirstName', OLD.FirstName,
            'LastName', OLD.LastName,
            'IsActive', OLD.IsActive
        ),
        JSON_OBJECT(
            'Email', NEW.Email,
            'FirstName', NEW.FirstName,
            'LastName', NEW.LastName,
            'IsActive', NEW.IsActive
        ),
        NOW()
    );
    
    -- Log security-sensitive changes
    IF OLD.Email != NEW.Email OR OLD.PasswordHash != NEW.PasswordHash THEN
        INSERT INTO SecurityLog (UserId, Action, Details, Timestamp)
        VALUES (NEW.UserId, 'PROFILE_SECURITY_CHANGE', 
                CONCAT('Email or password changed for user ', NEW.UserId), NOW());
    END IF;
END//

-- Order status change automation
CREATE TRIGGER trg_OrderStatusChange
AFTER UPDATE ON Orders
FOR EACH ROW
BEGIN
    -- Automatically update shipping date when status changes to 'shipped'
    IF NEW.Status = 'shipped' AND OLD.Status != 'shipped' THEN
        UPDATE Orders SET ShippingDate = NOW() WHERE OrderId = NEW.OrderId;
        
        -- Create shipping notification
        INSERT INTO NotificationQueue (UserId, Type, Message, CreatedAt)
        SELECT CustomerId, 'ORDER_SHIPPED', 
               CONCAT('Your order #', NEW.OrderId, ' has been shipped!'), NOW()
        FROM Orders WHERE OrderId = NEW.OrderId;
    END IF;
    
    -- Handle cancellations - restore inventory
    IF NEW.Status = 'cancelled' AND OLD.Status != 'cancelled' THEN
        UPDATE Products p
        INNER JOIN OrderItems oi ON p.ProductId = oi.ProductId
        SET p.StockQuantity = p.StockQuantity + oi.Quantity
        WHERE oi.OrderId = NEW.OrderId;
    END IF;
END//

-- Product view tracking trigger
CREATE TRIGGER trg_TrackProductViews
AFTER INSERT ON ProductViews
FOR EACH ROW
BEGIN
    UPDATE Products 
    SET ViewCount = ViewCount + 1,
        LastViewedAt = NEW.ViewedAt
    WHERE ProductId = NEW.ProductId;
    
    -- Update trending products table
    INSERT INTO TrendingProducts (ProductId, ViewsToday, LastUpdated)
    VALUES (NEW.ProductId, 1, CURDATE())
    ON DUPLICATE KEY UPDATE
        ViewsToday = ViewsToday + 1,
        LastUpdated = CURDATE();
END//
DELIMITER ;
```

**Result:** Automated business processes, comprehensive audit trails, and real-time data consistency for all API operations.

---

## Summary

These SQL security and validation commands create a robust foundation for API development by:

**Security Benefits:**
- **Role-based access control** through GRANT/REVOKE/DENY
- **Service isolation** with dedicated database users
- **Data filtering** through secure views
- **Business logic encapsulation** with stored procedures

**Data Integrity Benefits:**
- **Input validation** with NOT NULL and CHECK constraints
- **Duplicate prevention** with UNIQUE constraints
- **Referential integrity** with FOREIGN KEY constraints
- **Consistent defaults** with DEFAULT values
- **Automated processes** with TRIGGERS

**API Development Benefits:**
- **Reduced application complexity** - database handles validation
- **Consistent error handling** - predictable constraint violations
- **Automatic auditing** - comprehensive logging without code changes
- **Performance optimization** - database-level operations are faster
- **Defense in depth** - multiple security layers protect data

---

## Advanced Integration Patterns

### Multi-Tenant API Security Pattern

**Scenario:** SaaS platform serving multiple organizations through a single API

```javascript
// API with tenant isolation
app.use('/api', (req, res, next) => {
    const tenantId = req.headers['x-tenant-id'];
    req.dbUser = `tenant_${tenantId}_user`;
    next();
});

app.get('/api/customers', async (req, res) => {
    const connection = await getConnectionForTenant(req.dbUser);
    // Tenant-specific view automatically filters data
    const customers = await connection.query('SELECT * FROM CustomerView');
    res.json(customers);
});
```

**SQL Implementation:**
```sql
-- Per-tenant database users
CREATE USER tenant_1001_user IDENTIFIED BY 'TenantPass1001!';
CREATE USER tenant_1002_user IDENTIFIED BY 'TenantPass1002!';

-- Tenant-isolated views
CREATE VIEW CustomerView AS
SELECT * FROM Customers 
WHERE TenantId = GET_TENANT_ID();  -- Custom function based on connection user

-- Row-level security for automatic tenant filtering
ALTER TABLE Customers ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON Customers
    FOR ALL TO tenant_1001_user
    USING (TenantId = 1001);

CREATE POLICY tenant_isolation ON Customers
    FOR ALL TO tenant_1002_user
    USING (TenantId = 1002);
```

### API Rate Limiting with Database Integration

**Scenario:** Implement API rate limiting using database constraints and triggers

```javascript
// API with database-backed rate limiting
app.use('/api', async (req, res, next) => {
    const apiKey = req.headers['x-api-key'];
    
    try {
        await db.execute('CALL CheckRateLimit(?)', [apiKey]);
        next();
    } catch (error) {
        if (error.message.includes('RATE_LIMIT_EXCEEDED')) {
            return res.status(429).json({ 
                error: 'Rate limit exceeded',
                retryAfter: 3600 
            });
        }
        next();
    }
});
```

**SQL Implementation:**
```sql
-- Rate limiting tables
CREATE TABLE ApiKeys (
    KeyId INT AUTO_INCREMENT PRIMARY KEY,
    ApiKey VARCHAR(64) UNIQUE NOT NULL,
    ClientName VARCHAR(100) NOT NULL,
    RequestsPerHour INT DEFAULT 1000,
    IsActive BOOLEAN DEFAULT TRUE
);

CREATE TABLE RateLimitTracking (
    KeyId INT,
    HourWindow DATETIME,
    RequestCount INT DEFAULT 0,
    PRIMARY KEY (KeyId, HourWindow),
    FOREIGN KEY (KeyId) REFERENCES ApiKeys(KeyId)
);

DELIMITER //
CREATE PROCEDURE CheckRateLimit(IN p_ApiKey VARCHAR(64))
BEGIN
    DECLARE v_KeyId INT;
    DECLARE v_Limit INT;
    DECLARE v_CurrentCount INT;
    DECLARE v_HourWindow DATETIME;
    
    SET v_HourWindow = DATE_FORMAT(NOW(), '%Y-%m-%d %H:00:00');
    
    -- Get API key details
    SELECT KeyId, RequestsPerHour INTO v_KeyId, v_Limit
    FROM ApiKeys 
    WHERE ApiKey = p_ApiKey AND IsActive = TRUE;
    
    IF v_KeyId IS NULL THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Invalid API key';
    END IF;
    
    -- Get current count for this hour
    SELECT COALESCE(RequestCount, 0) INTO v_CurrentCount
    FROM RateLimitTracking 
    WHERE KeyId = v_KeyId AND HourWindow = v_HourWindow;
    
    IF v_CurrentCount >= v_Limit THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'RATE_LIMIT_EXCEEDED';
    END IF;
    
    -- Increment counter
    INSERT INTO RateLimitTracking (KeyId, HourWindow, RequestCount)
    VALUES (v_KeyId, v_HourWindow, 1)
    ON DUPLICATE KEY UPDATE RequestCount = RequestCount + 1;
END//
DELIMITER ;
```

### Circuit Breaker Pattern with Database Health Checks

**Scenario:** Implement circuit breaker pattern for API resilience

```javascript
class DatabaseCircuitBreaker {
    constructor() {
        this.failureCount = 0;
        this.lastFailureTime = null;
        this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
        this.threshold = 5;
        this.timeout = 60000; // 1 minute
    }
    
    async execute(operation) {
        if (this.state === 'OPEN') {
            if (Date.now() - this.lastFailureTime > this.timeout) {
                this.state = 'HALF_OPEN';
            } else {
                throw new Error('Circuit breaker is OPEN');
            }
        }
        
        try {
            const result = await operation();
            this.onSuccess();
            return result;
        } catch (error) {
            this.onFailure();
            throw error;
        }
    }
}

// API with circuit breaker
const circuitBreaker = new DatabaseCircuitBreaker();

app.get('/api/products', async (req, res) => {
    try {
        const products = await circuitBreaker.execute(() => 
            db.query('SELECT * FROM Products WHERE IsActive = 1')
        );
        res.json(products);
    } catch (error) {
        // Fallback to cache or simplified response
        res.json({ error: 'Service temporarily unavailable' });
    }
});
```

### Event-Driven API Architecture with Database Triggers

**Scenario:** Implement event-driven architecture using database triggers

```javascript
// Event processing API
app.post('/api/orders', async (req, res) => {
    const orderId = await createOrder(req.body);
    
    // Database triggers will automatically:
    // 1. Publish events to event queue
    // 2. Update related systems
    // 3. Send notifications
    
    res.status(201).json({ orderId });
});

// Event consumer
setInterval(async () => {
    const events = await db.query('SELECT * FROM EventQueue WHERE processed = 0 LIMIT 100');
    
    for (const event of events) {
        await processEvent(event);
        await db.execute('UPDATE EventQueue SET processed = 1 WHERE id = ?', [event.id]);
    }
}, 5000);
```

**SQL Implementation:**
```sql
-- Event queue table
CREATE TABLE EventQueue (
    Id INT AUTO_INCREMENT PRIMARY KEY,
    EventType VARCHAR(50) NOT NULL,
    EntityType VARCHAR(50) NOT NULL,
    EntityId INT NOT NULL,
    EventData JSON NOT NULL,
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ProcessedAt TIMESTAMP NULL,
    Processed BOOLEAN DEFAULT FALSE,
    RetryCount INT DEFAULT 0,
    
    INDEX idx_unprocessed (Processed, CreatedAt),
    INDEX idx_entity (EntityType, EntityId)
);

-- Event publishing trigger
DELIMITER //
CREATE TRIGGER trg_PublishOrderEvents
AFTER INSERT ON Orders
FOR EACH ROW
BEGIN
    -- Publish order created event
    INSERT INTO EventQueue (EventType, EntityType, EntityId, EventData)
    VALUES (
        'ORDER_CREATED',
        'Order',
        NEW.OrderId,
        JSON_OBJECT(
            'orderId', NEW.OrderId,
            'customerId', NEW.CustomerId,
            'totalAmount', NEW.TotalAmount,
            'status', NEW.Status,
            'createdAt', NEW.OrderDate
        )
    );
END//

CREATE TRIGGER trg_PublishOrderStatusEvents
AFTER UPDATE ON Orders
FOR EACH ROW
BEGIN
    IF OLD.Status != NEW.Status THEN
        INSERT INTO EventQueue (EventType, EntityType, EntityId, EventData)
        VALUES (
            'ORDER_STATUS_CHANGED',
            'Order',
            NEW.OrderId,
            JSON_OBJECT(
                'orderId', NEW.OrderId,
                'customerId', NEW.CustomerId,
                'oldStatus', OLD.Status,
                'newStatus', NEW.Status,
                'changedAt', NOW()
            )
        );
    END IF;
END//

CREATE TRIGGER trg_PublishInventoryEvents
AFTER UPDATE ON Products
FOR EACH ROW
BEGIN
    IF OLD.StockQuantity != NEW.StockQuantity THEN
        INSERT INTO EventQueue (EventType, EntityType, EntityId, EventData)
        VALUES (
            'INVENTORY_UPDATED',
            'Product',
            NEW.ProductId,
            JSON_OBJECT(
                'productId', NEW.ProductId,
                'oldStock', OLD.StockQuantity,
                'newStock', NEW.StockQuantity,
                'updatedAt', NOW()
            )
        );
        
        -- Low stock alert
        IF NEW.StockQuantity <= 5 AND OLD.StockQuantity > 5 THEN
            INSERT INTO EventQueue (EventType, EntityType, EntityId, EventData)
            VALUES (
                'LOW_STOCK_ALERT',
                'Product',
                NEW.ProductId,
                JSON_OBJECT(
                    'productId', NEW.ProductId,
                    'productName', NEW.Name,
                    'currentStock', NEW.StockQuantity,
                    'alertLevel', 'LOW',
                    'alertAt', NOW()
                )
            );
        END IF;
    END IF;
END//
DELIMITER ;
```

### API Analytics and Monitoring with Database Integration

**Scenario:** Real-time API analytics and monitoring using database features

```javascript
// API middleware for analytics
app.use('/api', async (req, res, next) => {
    const startTime = Date.now();
    const originalSend = res.send;
    
    res.send = function(data) {
        const responseTime = Date.now() - startTime;
        
        // Log API call asynchronously
        setImmediate(() => {
            logApiCall({
                endpoint: req.originalUrl,
                method: req.method,
                statusCode: res.statusCode,
                responseTime,
                userId: req.user?.id,
                apiKey: req.headers['x-api-key'],
                userAgent: req.headers['user-agent'],
                ip: req.ip
            });
        });
        
        originalSend.call(this, data);
    };
    
    next();
});

// Real-time analytics endpoint
app.get('/api/admin/analytics', requireAdmin, async (req, res) => {
    const analytics = await db.query(`
        SELECT 
            endpoint,
            COUNT(*) as total_requests,
            AVG(response_time_ms) as avg_response_time,
            COUNT(CASE WHEN status_code >= 400 THEN 1 END) as error_count,
            COUNT(DISTINCT user_id) as unique_users
        FROM ApiCallLogs 
        WHERE created_at >= DATE_SUB(NOW(), INTERVAL 1 HOUR)
        GROUP BY endpoint
        ORDER BY total_requests DESC
    `);
    
    res.json(analytics);
});
```

**SQL Implementation:**
```sql
-- API analytics tables
CREATE TABLE ApiCallLogs (
    Id BIGINT AUTO_INCREMENT PRIMARY KEY,
    Endpoint VARCHAR(255) NOT NULL,
    Method ENUM('GET', 'POST', 'PUT', 'DELETE', 'PATCH') NOT NULL,
    StatusCode INT NOT NULL,
    ResponseTimeMs INT NOT NULL,
    UserId INT NULL,
    ApiKey VARCHAR(64) NULL,
    UserAgent TEXT NULL,
    IpAddress VARCHAR(45) NOT NULL,
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_endpoint_time (Endpoint, CreatedAt),
    INDEX idx_user_time (UserId, CreatedAt),
    INDEX idx_status_time (StatusCode, CreatedAt),
    INDEX idx_apikey_time (ApiKey, CreatedAt)
);

-- Real-time analytics views
CREATE VIEW HourlyApiStats AS
SELECT 
    DATE_FORMAT(CreatedAt, '%Y-%m-%d %H:00:00') as hour_window,
    Endpoint,
    COUNT(*) as request_count,
    AVG(ResponseTimeMs) as avg_response_time,
    MIN(ResponseTimeMs) as min_response_time,
    MAX(ResponseTimeMs) as max_response_time,
    COUNT(CASE WHEN StatusCode >= 400 THEN 1 END) as error_count,
    COUNT(CASE WHEN StatusCode >= 500 THEN 1 END) as server_error_count,
    COUNT(DISTINCT UserId) as unique_users,
    COUNT(DISTINCT IpAddress) as unique_ips
FROM ApiCallLogs 
WHERE CreatedAt >= DATE_SUB(NOW(), INTERVAL 24 HOUR)
GROUP BY hour_window, Endpoint;

-- Performance monitoring stored procedure
DELIMITER //
CREATE PROCEDURE GetPerformanceMetrics(
    IN p_TimeWindow INT,  -- Hours to look back
    IN p_Endpoint VARCHAR(255)  -- NULL for all endpoints
)
BEGIN
    SELECT 
        CASE 
            WHEN p_Endpoint IS NULL THEN 'ALL_ENDPOINTS'
            ELSE p_Endpoint
        END as endpoint,
        COUNT(*) as total_requests,
        AVG(ResponseTimeMs) as avg_response_time,
        PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY ResponseTimeMs) as p50_response_time,
        PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY ResponseTimeMs) as p95_response_time,
        PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY ResponseTimeMs) as p99_response_time,
        COUNT(CASE WHEN StatusCode >= 400 THEN 1 END) as error_count,
        (COUNT(CASE WHEN StatusCode >= 400 THEN 1 END) / COUNT(*)) * 100 as error_rate,
        COUNT(DISTINCT UserId) as unique_users,
        COUNT(DISTINCT IpAddress) as unique_ips,
        MAX(ResponseTimeMs) as max_response_time,
        MIN(ResponseTimeMs) as min_response_time
    FROM ApiCallLogs 
    WHERE CreatedAt >= DATE_SUB(NOW(), INTERVAL p_TimeWindow HOUR)
        AND (p_Endpoint IS NULL OR Endpoint = p_Endpoint)
    GROUP BY 
        CASE 
            WHEN p_Endpoint IS NULL THEN 'ALL_ENDPOINTS'
            ELSE Endpoint
        END;
END//

-- Automated alerting trigger
CREATE TRIGGER trg_ApiPerformanceAlert
AFTER INSERT ON ApiCallLogs
FOR EACH ROW
BEGIN
    DECLARE v_ErrorRate DECIMAL(5,2);
    DECLARE v_AvgResponseTime DECIMAL(10,2);
    
    -- Check error rate for last 5 minutes
    SELECT 
        (COUNT(CASE WHEN StatusCode >= 500 THEN 1 END) / COUNT(*)) * 100,
        AVG(ResponseTimeMs)
    INTO v_ErrorRate, v_AvgResponseTime
    FROM ApiCallLogs 
    WHERE Endpoint = NEW.Endpoint 
        AND CreatedAt >= DATE_SUB(NOW(), INTERVAL 5 MINUTE);
    
    -- Alert on high error rate
    IF v_ErrorRate > 10 THEN
        INSERT INTO AlertQueue (AlertType, Severity, Message, CreatedAt)
        VALUES (
            'HIGH_ERROR_RATE',
            'HIGH',
            CONCAT('High error rate detected for ', NEW.Endpoint, ': ', v_ErrorRate, '%'),
            NOW()
        );
    END IF;
    
    -- Alert on slow response times
    IF v_AvgResponseTime > 5000 THEN  -- 5 seconds
        INSERT INTO AlertQueue (AlertType, Severity, Message, CreatedAt)
        VALUES (
            'SLOW_RESPONSE',
            'MEDIUM',
            CONCAT('Slow response time for ', NEW.Endpoint, ': ', v_AvgResponseTime, 'ms'),
            NOW()
        );
    END IF;
END//
DELIMITER ;
```

### Data Archiving and Cleanup Automation

**Scenario:** Automated data lifecycle management for API logs and temporary data

```sql
-- Data archiving automation
DELIMITER //
CREATE EVENT evt_ArchiveOldApiLogs
ON SCHEDULE EVERY 1 DAY
STARTS '2024-01-01 02:00:00'
DO
BEGIN
    -- Archive logs older than 90 days
    INSERT INTO ApiCallLogsArchive 
    SELECT * FROM ApiCallLogs 
    WHERE CreatedAt < DATE_SUB(CURDATE(), INTERVAL 90 DAY);
    
    -- Delete archived logs
    DELETE FROM ApiCallLogs 
    WHERE CreatedAt < DATE_SUB(CURDATE(), INTERVAL 90 DAY);
    
    -- Clean up processed events older than 7 days
    DELETE FROM EventQueue 
    WHERE Processed = 1 
        AND ProcessedAt < DATE_SUB(NOW(), INTERVAL 7 DAY);
    
    -- Clean up expired sessions
    DELETE FROM UserSessions 
    WHERE ExpiresAt < NOW();
    
    -- Update statistics
    INSERT INTO MaintenanceLog (Task, RecordsProcessed, CompletedAt)
    VALUES ('DAILY_CLEANUP', ROW_COUNT(), NOW());
END//

-- Performance optimization event
CREATE EVENT evt_OptimizePerformance
ON SCHEDULE EVERY 1 WEEK
STARTS '2024-01-01 01:00:00'
DO
BEGIN
    -- Analyze and optimize frequently used tables
    ANALYZE TABLE ApiCallLogs, Orders, Products, Users;
    
    -- Rebuild indexes if needed
    OPTIMIZE TABLE ApiCallLogs, Orders, Products;
    
    -- Update table statistics
    UPDATE INFORMATION_SCHEMA.TABLES 
    SET TABLE_COMMENT = CONCAT('Last optimized: ', NOW())
    WHERE TABLE_SCHEMA = DATABASE()
        AND TABLE_NAME IN ('ApiCallLogs', 'Orders', 'Products', 'Users');
END//
DELIMITER ;

-- Enable event scheduler
SET GLOBAL event_scheduler = ON;
```

This comprehensive guide demonstrates how SQL security and validation commands create a robust, scalable foundation for API development with automated security, data integrity, performance monitoring, and maintenance capabilities built directly into the database layer.
