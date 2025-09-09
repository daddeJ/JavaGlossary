# SQL Data Manipulation Guide

## Introduction
This comprehensive guide covers essential SQL commands for manipulating database records, including inserting, updating, and deleting data. Each section includes syntax explanations, practical examples, and best practices to ensure accurate and safe data management.

---

## 1. Inserting Data with INSERT Statements

### Purpose & Intention
The INSERT statement is used to add new records to database tables. This is fundamental for populating databases with initial data or adding new entries as your application grows.

### Basic INSERT Syntax
```sql
INSERT INTO tableName (columnA, columnB, columnC) 
VALUES ('valueA', 'valueB', 'valueC');
```

### Single Record Insert

**Intention**: Add one new customer to the database
```sql
-- Sample: Adding a new customer
INSERT INTO Customer (CustomerID, FirstName, LastName, Email, Phone) 
VALUES (101, 'John', 'Doe', 'john.doe@email.com', '555-0123');
```

**Intention**: Add a new product to inventory
```sql
-- Sample: Adding a new product
INSERT INTO Products (ProductID, ProductName, Category, Price, StockQuantity) 
VALUES (201, 'Wireless Headphones', 'Electronics', 89.99, 50);
```

### Multiple Record Insert (Batch Insert)

**Intention**: Add multiple customers efficiently in a single operation
```sql
-- Sample: Batch insert multiple customers
INSERT INTO Customer (CustomerID, FirstName, LastName, Email, Phone) 
VALUES 
    (102, 'Jane', 'Smith', 'jane.smith@email.com', '555-0124'),
    (103, 'Mike', 'Johnson', 'mike.j@email.com', '555-0125'),
    (104, 'Sarah', 'Wilson', 'sarah.w@email.com', '555-0126');
```

**Intention**: Populate a music catalog with multiple songs
```sql
-- Sample: Adding multiple songs to music catalog
INSERT INTO MusicCatalog (SongID, Title, Artist, Genre, Duration, ReleaseYear) 
VALUES 
    (1, 'Bohemian Rhapsody', 'Queen', 'Rock', 355, 1975),
    (2, 'Shape of You', 'Ed Sheeran', 'Pop', 233, 2017),
    (3, 'Billie Jean', 'Michael Jackson', 'Pop', 294, 1983);
```

### Working with Foreign Key Constraints

**Intention**: Add orders that reference existing customers
```sql
-- Sample: Insert order with foreign key reference
-- First, verify customer exists
SELECT CustomerID FROM Customer WHERE CustomerID = 101;

-- Then insert the order
INSERT INTO Orders (OrderID, CustomerID, OrderDate, TotalAmount) 
VALUES (501, 101, '2024-09-09', 299.99);
```

---

## 2. Updating Data with UPDATE Statements

### Purpose & Intention
The UPDATE statement modifies existing records in a table. This is crucial for maintaining current information, correcting errors, or bulk updating records based on business rules.

### Basic UPDATE Syntax
```sql
UPDATE tableName 
SET columnA = 'newValue', columnB = 'anotherValue' 
WHERE condition;
```

### Single Record Update

**Intention**: Update a specific customer's contact information
```sql
-- Sample: Update customer email
UPDATE Customer 
SET Email = 'john.doe.new@email.com', Phone = '555-9999' 
WHERE CustomerID = 101;
```

**Intention**: Correct a product's price
```sql
-- Sample: Update product price
UPDATE Products 
SET Price = 79.99 
WHERE ProductID = 201;
```

### Multiple Record Updates

**Intention**: Apply a discount to all electronics products
```sql
-- Sample: Bulk price reduction for electronics
UPDATE Products 
SET Price = Price * 0.9  -- 10% discount
WHERE Category = 'Electronics';
```

**Intention**: Mark popular songs based on play count
```sql
-- Sample: Update popularity status for viral songs
UPDATE MusicCatalog 
SET Popularity = 'Viral', LastUpdated = CURRENT_TIMESTAMP 
WHERE WeeklyListeners > 1000000;
```

### Conditional Updates

**Intention**: Update stock status based on quantity levels
```sql
-- Sample: Update stock status based on quantity
UPDATE Products 
SET StockStatus = CASE 
    WHEN StockQuantity = 0 THEN 'Out of Stock'
    WHEN StockQuantity < 10 THEN 'Low Stock'
    ELSE 'In Stock'
END;
```

---

## 3. Deleting Data with DELETE Statements

### Purpose & Intention
The DELETE statement removes records from a table. This is used for data cleanup, removing obsolete records, or maintaining data quality by eliminating unwanted entries.

### Basic DELETE Syntax
```sql
DELETE FROM tableName 
WHERE condition;
```

### Single Record Deletion

**Intention**: Remove a specific discontinued product
```sql
-- Sample: Delete a discontinued product
DELETE FROM Products 
WHERE ProductID = 201 AND StockQuantity = 0;
```

**Intention**: Remove a canceled order
```sql
-- Sample: Delete canceled order
DELETE FROM Orders 
WHERE OrderID = 501 AND OrderStatus = 'Canceled';
```

### Multiple Record Deletion

**Intention**: Clean up old, inactive customer records
```sql
-- Sample: Remove customers who haven't ordered in 2+ years
DELETE FROM Customer 
WHERE CustomerID NOT IN (
    SELECT DISTINCT CustomerID 
    FROM Orders 
    WHERE OrderDate > DATE_SUB(CURRENT_DATE, INTERVAL 2 YEAR)
);
```

**Intention**: Remove unpopular songs from catalog
```sql
-- Sample: Delete songs with very low play counts
DELETE FROM MusicCatalog 
WHERE WeeklyListeners < 100 AND ReleaseYear < 2010;
```

### Complete Table Cleanup

**Intention**: Clear all records while preserving table structure
```sql
-- Sample: Clear entire temporary table
DELETE FROM TempProcessingData;
-- This removes all rows but keeps the table structure intact
```

---

## 4. Best Practices & Safety Guidelines

### Pre-Execution Verification

**Intention**: Always verify which records will be affected before executing destructive operations

```sql
-- Before UPDATE: Check what will be changed
SELECT CustomerID, FirstName, LastName, Email 
FROM Customer 
WHERE CustomerID = 101;

-- Before DELETE: Verify what will be removed
SELECT * FROM Products 
WHERE Category = 'Discontinued' AND StockQuantity = 0;
```

### Transaction Safety

**Intention**: Use transactions for critical operations to allow rollback if needed

```sql
-- Sample: Safe update with transaction
BEGIN TRANSACTION;

UPDATE Products 
SET Price = Price * 1.1 
WHERE Category = 'Premium';

-- Verify the changes look correct
SELECT * FROM Products WHERE Category = 'Premium';

-- If satisfied, commit; otherwise, rollback
COMMIT;
-- OR: ROLLBACK;
```

### Backup Strategy

**Intention**: Create backups before major data modifications

```sql
-- Sample: Create backup table before bulk operations
CREATE TABLE Customer_Backup AS 
SELECT * FROM Customer;

-- Perform your operations
UPDATE Customer SET Status = 'Active' WHERE LastOrderDate > '2024-01-01';

-- If something goes wrong, restore from backup
-- DROP TABLE Customer;
-- ALTER TABLE Customer_Backup RENAME TO Customer;
```

---

## 5. Common Patterns & Use Cases

### Data Migration Pattern

**Intention**: Move data from staging to production tables
```sql
-- Sample: Migrate validated data
INSERT INTO Customer (CustomerID, FirstName, LastName, Email)
SELECT CustomerID, FirstName, LastName, Email
FROM Customer_Staging 
WHERE ValidationStatus = 'Approved';
```

### Data Synchronization Pattern

**Intention**: Keep related tables in sync
```sql
-- Sample: Update customer total spent
UPDATE Customer 
SET TotalSpent = (
    SELECT COALESCE(SUM(TotalAmount), 0)
    FROM Orders 
    WHERE Orders.CustomerID = Customer.CustomerID
);
```

### Audit Trail Pattern

**Intention**: Maintain history before making changes
```sql
-- Sample: Archive before deletion
INSERT INTO Customer_Archive 
SELECT *, CURRENT_TIMESTAMP as ArchivedDate 
FROM Customer 
WHERE Status = 'Inactive';

DELETE FROM Customer 
WHERE Status = 'Inactive';
```

---

## Conclusion

Mastering SQL data manipulation commands—INSERT, UPDATE, and DELETE—is essential for effective database management. By following the patterns and best practices outlined in this guide, you can:

- **Insert data** efficiently and maintain referential integrity
- **Update records** safely with proper conditions and verification
- **Delete data** responsibly while preserving important information
- **Maintain data quality** through proper validation and backup strategies

Remember: Always test your queries on a copy of your data first, use transactions for critical operations, and maintain regular backups to ensure data safety and integrity.
