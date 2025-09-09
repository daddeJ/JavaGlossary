# Retrieving Data with SELECT Statements

## Introduction

SQL's **SELECT** statement is a fundamental tool for accessing specific data from databases. It enables users to pull, filter, and organize records based on defined criteria.

## Using SELECT Statements for Data Retrieval

### Basic SELECT Syntax

**Intent**: Retrieve specific columns or all columns from a database table to view and analyze data.

**Syntax**:
```sql
SELECT column_name(s) FROM table_name;
SELECT * FROM table_name;
```

**Examples**:
```sql
-- Retrieve specific columns
SELECT FirstName, LastName, Email FROM Employees;

-- Retrieve all columns
SELECT * FROM Employees;

-- Retrieve a single column
SELECT Department FROM Employees;
```

**Use Case**: When you need to view employee contact information or get a complete overview of all employee data.

## Filtering Data with WHERE

### WHERE Clause

**Intent**: Filter rows that meet specific conditions to retrieve only relevant data instead of the entire table.

**Syntax**:
```sql
SELECT column_name(s) FROM table_name WHERE condition;
```

**Examples**:
```sql
-- Single condition
SELECT * FROM Employees WHERE Department = 'HR';

-- Numeric comparison
SELECT FirstName, LastName, Salary FROM Employees WHERE Salary > 50000;

-- Multiple conditions with AND
SELECT * FROM Employees WHERE Department = 'IT' AND Salary > 60000;

-- Multiple conditions with OR
SELECT * FROM Employees WHERE Department = 'HR' OR Department = 'Finance';

-- Date filtering
SELECT * FROM Employees WHERE HireDate >= '2020-01-01';

-- Text pattern matching
SELECT * FROM Employees WHERE FirstName LIKE 'J%';
```

**Use Case**: Find all HR employees, identify high-earning staff, or locate employees hired after a specific date.

## Sorting Results with ORDER BY

### ORDER BY Clause

**Intent**: Organize query results in ascending or descending order to make data easier to interpret and analyze.

**Syntax**:
```sql
SELECT column_name(s) FROM table_name ORDER BY column_name ASC/DESC;
```

**Examples**:
```sql
-- Sort by salary (highest first)
SELECT * FROM Employees ORDER BY Salary DESC;

-- Sort by last name (alphabetical)
SELECT FirstName, LastName FROM Employees ORDER BY LastName ASC;

-- Multiple column sorting
SELECT * FROM Employees ORDER BY Department ASC, Salary DESC;

-- Sort with WHERE clause
SELECT * FROM Employees WHERE Department = 'IT' ORDER BY HireDate DESC;
```

**Use Case**: Identify top earners, alphabetize employee lists, or find the most recently hired employees in each department.

## Removing Duplicates with DISTINCT

### DISTINCT Keyword

**Intent**: Eliminate duplicate values from query results to show only unique entries.

**Syntax**:
```sql
SELECT DISTINCT column_name(s) FROM table_name;
```

**Examples**:
```sql
-- Get unique departments
SELECT DISTINCT Department FROM Employees;

-- Get unique job titles
SELECT DISTINCT JobTitle FROM Employees;

-- Get unique combinations
SELECT DISTINCT Department, JobTitle FROM Employees;

-- Count unique values
SELECT COUNT(DISTINCT Department) AS UniqueDepartments FROM Employees;
```

**Use Case**: Create a list of all departments in the company, identify different job roles, or count how many unique categories exist.

## Limiting the Number of Rows Returned

### TOP/LIMIT Clause

**Intent**: Restrict the number of rows returned to focus on the most relevant records, such as top performers or recent entries.

**Syntax**:
```sql
-- SQL Server/MS Access
SELECT TOP number column_name(s) FROM table_name;

-- MySQL/PostgreSQL
SELECT column_name(s) FROM table_name LIMIT number;
```

**Examples**:
```sql
-- Top 5 highest paid employees (SQL Server)
SELECT TOP 5 * FROM Employees ORDER BY Salary DESC;

-- Top 5 highest paid employees (MySQL)
SELECT * FROM Employees ORDER BY Salary DESC LIMIT 5;

-- Most recently hired employees
SELECT TOP 10 FirstName, LastName, HireDate 
FROM Employees 
ORDER BY HireDate DESC;

-- First 3 employees alphabetically
SELECT TOP 3 * FROM Employees ORDER BY LastName ASC;
```

**Use Case**: Find the top 5 sales performers, get the 10 most recent hires, or sample a small portion of data for testing.

## Combining Multiple Clauses

### Complete Query Examples

**Intent**: Combine multiple SQL clauses to create powerful, targeted queries that filter, sort, and limit data simultaneously.

**Examples**:
```sql
-- Get top 5 IT employees by salary
SELECT TOP 5 FirstName, LastName, Salary 
FROM Employees 
WHERE Department = 'IT' 
ORDER BY Salary DESC;

-- Get unique departments with employee count, sorted by count
SELECT DISTINCT Department, COUNT(*) as EmployeeCount
FROM Employees 
GROUP BY Department 
ORDER BY EmployeeCount DESC;

-- Get recent hires in specific departments
SELECT FirstName, LastName, Department, HireDate
FROM Employees 
WHERE Department IN ('IT', 'HR', 'Finance') 
  AND HireDate >= '2023-01-01'
ORDER BY HireDate DESC;
```

## Sample Employee Table Reference

For context, here's what a typical Employees table might look like:

| EmployeeID | FirstName | LastName | Department | JobTitle | Salary | HireDate |
|------------|-----------|----------|------------|----------|--------|----------|
| 1 | John | Smith | IT | Developer | 75000 | 2022-03-15 |
| 2 | Jane | Doe | HR | Manager | 80000 | 2021-06-20 |
| 3 | Mike | Johnson | Finance | Analyst | 65000 | 2023-01-10 |
| 4 | Sarah | Wilson | IT | Senior Developer | 90000 | 2020-11-05 |
| 5 | Tom | Brown | HR | Specialist | 55000 | 2023-08-12 |

## Best Practices

1. **Always use specific column names** instead of SELECT * when possible for better performance
2. **Add appropriate WHERE clauses** to limit data retrieval and improve query speed
3. **Use ORDER BY** to make results more readable and meaningful
4. **Apply DISTINCT carefully** as it can impact performance on large datasets
5. **Combine clauses logically** - WHERE before ORDER BY, ORDER BY before LIMIT/TOP

## Conclusion

The **SELECT** statement combined with **WHERE**, **ORDER BY**, **DISTINCT**, and **TOP/LIMIT** keywords provides a robust framework for customized data retrieval. Mastering these commands enables efficient querying, supporting deeper data insights and more effective decision-making in database management and analysis.
