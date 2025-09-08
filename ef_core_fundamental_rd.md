# Fundamentals of Relational Databases

## Main Intent

The primary intent of relational databases is to **provide a structured, efficient, and reliable way to store, organize, and retrieve data**. They solve the fundamental challenges of data management by:

- **Eliminating data redundancy** through proper organization
- **Ensuring data integrity** through relationships and constraints
- **Enabling efficient data retrieval** through structured queries
- **Supporting concurrent access** by multiple users safely
- **Maintaining consistency** across related information

---

## Introduction

Relational databases are the backbone of most modern applications, providing a systematic approach to data management. They organize information into structured tables with well-defined relationships, making complex data operations reliable and efficient.

### Why Relational Databases Matter:
- **Structured Data Management** - Organize complex information systematically
- **Data Integrity** - Maintain accuracy and consistency across all operations
- **Scalability** - Handle growing amounts of data and users
- **Standardization** - Use SQL as a universal query language
- **ACID Properties** - Guarantee reliable transactions

---

## Structure of Relational Databases

### Tables, Rows, and Columns

Relational databases organize data into **tables** (also called relations), which consist of:
- **Rows (Records/Tuples)** - Individual data entries
- **Columns (Attributes/Fields)** - Specific data properties

#### Example: Library Management System

```sql
-- Books Table
CREATE TABLE Books (
    BookID INT PRIMARY KEY,
    Title VARCHAR(255) NOT NULL,
    ISBN VARCHAR(13) UNIQUE,
    PublicationYear INT,
    Genre VARCHAR(100),
    Price DECIMAL(10,2)
);

-- Sample Data
INSERT INTO Books VALUES 
(1, 'The Great Gatsby', '9780743273565', 1925, 'Fiction', 12.99),
(2, 'To Kill a Mockingbird', '9780061120084', 1960, 'Fiction', 14.99),
(3, 'Database Systems', '9780321197849', 2010, 'Technology', 89.99);
```

**Table Structure Breakdown:**
- **Table Name**: Books
- **Columns**: BookID, Title, ISBN, PublicationYear, Genre, Price
- **Rows**: Each book entry represents one complete record

---

## Keys and Schemas

### Primary Keys

Primary keys uniquely identify each record in a table, ensuring no duplicates exist.

```sql
-- Example: Student Table with Primary Key
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,        -- Primary Key
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(100) UNIQUE,
    EnrollmentDate DATE
);

-- This ensures each student has a unique identifier
INSERT INTO Students VALUES 
(1001, 'John', 'Smith', 'john.smith@email.com', '2023-09-01'),
(1002, 'Jane', 'Doe', 'jane.doe@email.com', '2023-09-01');
```

### Foreign Keys

Foreign keys create relationships between tables by referencing primary keys in other tables.

```sql
-- Enrollments Table with Foreign Key
CREATE TABLE Enrollments (
    EnrollmentID INT PRIMARY KEY,
    StudentID INT,                    -- Foreign Key
    CourseID INT,                     -- Foreign Key  
    EnrollmentDate DATE,
    Grade CHAR(2),
    
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
);
```

### Database Schema

A schema defines the structure and organization of the entire database:

```sql
-- Complete Library Schema Example
CREATE SCHEMA LibraryManagement;

-- Authors Table
CREATE TABLE Authors (
    AuthorID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    BirthYear INT,
    Country VARCHAR(50)
);

-- Publishers Table  
CREATE TABLE Publishers (
    PublisherID INT PRIMARY KEY,
    PublisherName VARCHAR(100) NOT NULL,
    Address VARCHAR(255),
    ContactEmail VARCHAR(100)
);

-- Books Table with Foreign Keys
CREATE TABLE Books (
    BookID INT PRIMARY KEY,
    Title VARCHAR(255) NOT NULL,
    AuthorID INT,
    PublisherID INT,
    ISBN VARCHAR(13) UNIQUE,
    PublicationYear INT,
    
    FOREIGN KEY (AuthorID) REFERENCES Authors(AuthorID),
    FOREIGN KEY (PublisherID) REFERENCES Publishers(PublisherID)
);
```

---

## Database Relationships

### One-to-One Relationship

Each record in one table corresponds to exactly one record in another table.

```sql
-- Employee and Employee_Details (One-to-One)
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Department VARCHAR(50)
);

CREATE TABLE EmployeeDetails (
    EmployeeID INT PRIMARY KEY,
    SSN VARCHAR(11) UNIQUE,
    HomeAddress VARCHAR(255),
    PersonalPhone VARCHAR(15),
    
    FOREIGN KEY (EmployeeID) REFERENCES Employees(EmployeeID)
);
```

**Real-world example**: Each employee has exactly one set of personal details.

### One-to-Many Relationship

One record in a table can be related to multiple records in another table.

```sql
-- Department to Employees (One-to-Many)
CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY,
    DepartmentName VARCHAR(100),
    ManagerID INT
);

CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    DepartmentID INT,
    
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
);

-- Sample Data showing One-to-Many
INSERT INTO Departments VALUES (1, 'Engineering', 101);
INSERT INTO Employees VALUES 
(101, 'Alice', 'Johnson', 1),  -- Engineering
(102, 'Bob', 'Wilson', 1),     -- Engineering  
(103, 'Carol', 'Davis', 1);    -- Engineering
```

**Real-world example**: One department has many employees.

### Many-to-Many Relationship

Multiple records in one table can be related to multiple records in another table, managed through a junction table.

```sql
-- Students and Courses (Many-to-Many)
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    StudentName VARCHAR(100)
);

CREATE TABLE Courses (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(100),
    Credits INT
);

-- Junction Table
CREATE TABLE StudentCourses (
    StudentID INT,
    CourseID INT,
    EnrollmentDate DATE,
    Grade CHAR(2),
    
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
);

-- Sample Data
INSERT INTO Students VALUES (1, 'John Smith'), (2, 'Jane Doe');
INSERT INTO Courses VALUES (101, 'Mathematics'), (102, 'Physics');

-- Many-to-Many relationships
INSERT INTO StudentCourses VALUES 
(1, 101, '2023-09-01', 'A'),  -- John takes Math
(1, 102, '2023-09-01', 'B'),  -- John takes Physics
(2, 101, '2023-09-01', 'A'),  -- Jane takes Math
(2, 102, '2023-09-01', 'A');  -- Jane takes Physics
```

---

## Normalization

Normalization eliminates redundancy and ensures data integrity through structured refinement.

### Before Normalization (Problematic Design)

```sql
-- Unnormalized Table (BAD EXAMPLE)
CREATE TABLE StudentCourseInfo (
    StudentID INT,
    StudentName VARCHAR(100),
    StudentEmail VARCHAR(100),
    CourseID INT,
    CourseName VARCHAR(100),
    InstructorName VARCHAR(100),
    InstructorEmail VARCHAR(100),
    Grade CHAR(2)
);

-- Problems:
-- 1. Student info repeated for each course
-- 2. Instructor info repeated for each student in course
-- 3. Update anomalies - changing instructor name requires multiple updates
```

### First Normal Form (1NF)

Eliminate repeating groups and ensure atomic values.

```sql
-- 1NF: Atomic values, no repeating groups
CREATE TABLE StudentCourses_1NF (
    StudentID INT,
    StudentName VARCHAR(100),
    StudentEmail VARCHAR(100),
    CourseID INT,
    CourseName VARCHAR(100),
    InstructorName VARCHAR(100),
    Grade CHAR(2)
);

-- Each column contains only one value (atomic)
-- No comma-separated lists or multiple values in single field
```

### Second Normal Form (2NF)

Remove partial dependencies - all non-key attributes must depend on the entire primary key.

```sql
-- 2NF: Separate tables to eliminate partial dependencies
CREATE TABLE Students_2NF (
    StudentID INT PRIMARY KEY,
    StudentName VARCHAR(100),
    StudentEmail VARCHAR(100)
);

CREATE TABLE Courses_2NF (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(100),
    InstructorName VARCHAR(100)
);

CREATE TABLE Enrollments_2NF (
    StudentID INT,
    CourseID INT,
    Grade CHAR(2),
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Students_2NF(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Courses_2NF(CourseID)
);
```

### Third Normal Form (3NF)

Remove transitive dependencies - non-key attributes should not depend on other non-key attributes.

```sql
-- 3NF: Remove transitive dependencies
CREATE TABLE Students_3NF (
    StudentID INT PRIMARY KEY,
    StudentName VARCHAR(100),
    StudentEmail VARCHAR(100)
);

CREATE TABLE Instructors_3NF (
    InstructorID INT PRIMARY KEY,
    InstructorName VARCHAR(100),
    InstructorEmail VARCHAR(100)
);

CREATE TABLE Courses_3NF (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(100),
    InstructorID INT,
    FOREIGN KEY (InstructorID) REFERENCES Instructors_3NF(InstructorID)
);

CREATE TABLE Enrollments_3NF (
    StudentID INT,
    CourseID INT,
    Grade CHAR(2),
    EnrollmentDate DATE,
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Students_3NF(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Courses_3NF(CourseID)
);
```

**Benefits of Normalization:**
- **Eliminates redundancy** - Each piece of information stored once
- **Prevents update anomalies** - Change data in one place only
- **Maintains consistency** - No conflicting information
- **Saves storage space** - Less duplicate data

---

## Constraints

Constraints enforce business rules and maintain data quality.

### NOT NULL Constraint

Prevents empty values in critical fields.

```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(100) NOT NULL,    -- Required field
    Price DECIMAL(10,2) NOT NULL,         -- Price must be specified
    CategoryID INT NOT NULL,              -- Must belong to a category
    Description TEXT                      -- Optional field
);

-- This will fail:
-- INSERT INTO Products (ProductID) VALUES (1);  -- Error: NOT NULL constraint
```

### UNIQUE Constraint

Ensures no duplicate values in specified columns.

```sql
CREATE TABLE Users (
    UserID INT PRIMARY KEY,
    Username VARCHAR(50) NOT NULL UNIQUE,  -- No duplicate usernames
    Email VARCHAR(100) NOT NULL UNIQUE,    -- No duplicate emails
    FirstName VARCHAR(50),
    LastName VARCHAR(50)
);

-- This will fail if username already exists:
-- INSERT INTO Users VALUES (2, 'existing_user', 'new@email.com', 'John', 'Doe');
```

### CHECK Constraint

Validates data against specific conditions.

```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    Age INT CHECK (Age >= 18 AND Age <= 65),        -- Age validation
    Salary DECIMAL(10,2) CHECK (Salary > 0),        -- Positive salary
    HireDate DATE CHECK (HireDate >= '2000-01-01'), -- Reasonable hire date
    Status VARCHAR(20) CHECK (Status IN ('Active', 'Inactive', 'Terminated'))
);

-- Valid insert
INSERT INTO Employees VALUES (1, 'John', 25, 50000.00, '2023-01-15', 'Active');

-- This will fail:
-- INSERT INTO Employees VALUES (2, 'Jane', 17, -1000, '1999-12-31', 'Unknown');
```

### DEFAULT Constraint

Provides default values when no value is specified.

```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    OrderDate DATE DEFAULT CURRENT_DATE,           -- Default to today
    Status VARCHAR(20) DEFAULT 'Pending',          -- Default status
    Priority INT DEFAULT 1,                        -- Default priority
    ShippingCost DECIMAL(8,2) DEFAULT 0.00        -- Default shipping cost
);

-- Insert with defaults
INSERT INTO Orders (OrderID, CustomerID) VALUES (1, 100);
-- Results in: OrderID=1, CustomerID=100, OrderDate=today, Status='Pending', Priority=1, ShippingCost=0.00
```

---

## Real-World Example: E-Commerce Database

Here's a comprehensive example showing all concepts together:

```sql
-- Complete E-Commerce Database Schema

-- Customers Table
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(100) NOT NULL UNIQUE,
    RegistrationDate DATE DEFAULT CURRENT_DATE,
    Status VARCHAR(20) DEFAULT 'Active' CHECK (Status IN ('Active', 'Inactive'))
);

-- Categories Table
CREATE TABLE Categories (
    CategoryID INT PRIMARY KEY,
    CategoryName VARCHAR(100) NOT NULL UNIQUE,
    Description TEXT
);

-- Products Table
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(255) NOT NULL,
    CategoryID INT NOT NULL,
    Price DECIMAL(10,2) NOT NULL CHECK (Price > 0),
    StockQuantity INT DEFAULT 0 CHECK (StockQuantity >= 0),
    CreatedDate DATE DEFAULT CURRENT_DATE,
    
    FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID)
);

-- Orders Table
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    OrderDate DATETIME DEFAULT CURRENT_TIMESTAMP,
    TotalAmount DECIMAL(12,2) NOT NULL CHECK (TotalAmount >= 0),
    Status VARCHAR(20) DEFAULT 'Pending' CHECK (Status IN ('Pending', 'Shipped', 'Delivered', 'Cancelled')),
    
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

-- OrderItems Table (Junction table for Many-to-Many between Orders and Products)
CREATE TABLE OrderItems (
    OrderID INT,
    ProductID INT,
    Quantity INT NOT NULL CHECK (Quantity > 0),
    UnitPrice DECIMAL(10,2) NOT NULL CHECK (UnitPrice > 0),
    
    PRIMARY KEY (OrderID, ProductID),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);

-- Sample Data
INSERT INTO Categories VALUES (1, 'Electronics', 'Electronic devices and accessories');
INSERT INTO Categories VALUES (2, 'Books', 'Physical and digital books');

INSERT INTO Products VALUES 
(1, 'Laptop Computer', 1, 999.99, 50, '2024-01-01'),
(2, 'Programming Guide', 2, 29.99, 100, '2024-01-01');

INSERT INTO Customers VALUES 
(1, 'John', 'Smith', 'john@email.com', '2024-01-15', 'Active');

INSERT INTO Orders VALUES 
(1, 1, '2024-02-01 10:30:00', 1029.98, 'Delivered');

INSERT INTO OrderItems VALUES 
(1, 1, 1, 999.99),  -- 1 Laptop
(1, 2, 1, 29.99);   -- 1 Book
```

---

## Query Examples

### Retrieving Related Data

```sql
-- Get all orders with customer information
SELECT 
    o.OrderID,
    c.FirstName + ' ' + c.LastName AS CustomerName,
    o.OrderDate,
    o.TotalAmount,
    o.Status
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
WHERE o.Status = 'Delivered';

-- Get order details with product information
SELECT 
    o.OrderID,
    c.FirstName + ' ' + c.LastName AS Customer,
    p.ProductName,
    oi.Quantity,
    oi.UnitPrice,
    (oi.Quantity * oi.UnitPrice) AS LineTotal
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID  
JOIN OrderItems oi ON o.OrderID = oi.OrderID
JOIN Products p ON oi.ProductID = p.ProductID
ORDER BY o.OrderID, p.ProductName;
```

---

## Benefits of Relational Database Design

1. **Data Integrity** - Constraints ensure data quality and consistency
2. **Efficient Storage** - Normalization eliminates redundancy
3. **Flexible Queries** - SQL enables complex data retrieval
4. **Scalability** - Proper design handles growing data volumes
5. **Concurrent Access** - Multiple users can safely access data simultaneously
6. **Data Security** - Role-based access control and encryption
7. **Backup and Recovery** - Built-in mechanisms for data protection

---

## Best Practices

1. **Design before implementation** - Plan your schema carefully
2. **Use meaningful names** - Clear table and column names
3. **Implement proper constraints** - Enforce business rules at database level
4. **Normalize appropriately** - Usually aim for 3NF, denormalize only when necessary
5. **Index strategically** - Improve query performance on frequently searched columns
6. **Document your schema** - Maintain clear documentation of relationships and constraints

Relational databases provide the foundation for reliable, scalable data management. Understanding these fundamentals enables developers to build robust applications that maintain data integrity while supporting complex business requirements.
