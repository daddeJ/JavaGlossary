# Entity Framework Core (EF Core) Guide

## Main Intent

The primary intent of Entity Framework Core is to **bridge the gap between object-oriented programming and relational databases**. It solves the fundamental mismatch between how we think about data in code (as objects) versus how databases store data (in tables).

### Core Problems EF Core Solves:
- **Eliminates boilerplate SQL code** - No need to write repetitive CRUD operations
- **Provides type safety** - Compile-time checking instead of runtime SQL errors
- **Reduces complexity** - Work with familiar C# objects instead of raw SQL
- **Improves maintainability** - Database schema changes are managed through migrations
- **Enhances productivity** - Focus on business logic rather than data access code

---

## Key Concepts

### Object-Relational Mapping (ORM)
ORM is a programming technique that creates a bridge between object-oriented programming languages and relational databases. It allows developers to work with database records as if they were regular objects in their code.

### Entity Framework Core Features
- **LINQ Integration** - Write database queries using C# syntax
- **Database Migrations** - Version control for your database schema
- **Change Tracking** - Automatically detect and persist object changes
- **Multi-database Support** - Works with SQL Server, PostgreSQL, SQLite, MySQL, and more

---

## Practical Examples

### 1. Setting Up EF Core

First, install the necessary NuGet packages:

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

### 2. Creating Entity Models

```csharp
// Product.cs
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
    public DateTime CreatedDate { get; set; }
    
    // Navigation property
    public List<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
}

// Order.cs
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public string CustomerName { get; set; }
    public decimal TotalAmount { get; set; }
    
    // Navigation property
    public List<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
}

// OrderItem.cs
public class OrderItem
{
    public int Id { get; set; }
    public int ProductId { get; set; }
    public int OrderId { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
    
    // Navigation properties
    public Product Product { get; set; }
    public Order Order { get; set; }
}
```

### 3. Creating the DbContext

```csharp
// ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }
    
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure entity relationships and constraints
        modelBuilder.Entity<Product>()
            .Property(p => p.Price)
            .HasColumnType("decimal(18,2)");
            
        modelBuilder.Entity<OrderItem>()
            .HasOne(oi => oi.Product)
            .WithMany(p => p.OrderItems)
            .HasForeignKey(oi => oi.ProductId);
            
        modelBuilder.Entity<OrderItem>()
            .HasOne(oi => oi.Order)
            .WithMany(o => o.OrderItems)
            .HasForeignKey(oi => oi.OrderId);
    }
}
```

### 4. Configuration in Program.cs (ASP.NET Core)

```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add EF Core service
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();
```

### 5. Database Operations with LINQ

#### Create (Insert) Operations
```csharp
public class ProductService
{
    private readonly ApplicationDbContext _context;
    
    public ProductService(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<Product> CreateProductAsync(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return product;
    }
}
```

#### Read (Query) Operations
```csharp
public async Task<List<Product>> GetExpensiveProductsAsync()
{
    return await _context.Products
        .Where(p => p.Price > 100)
        .OrderByDescending(p => p.Price)
        .ToListAsync();
}

public async Task<Product?> GetProductWithOrdersAsync(int productId)
{
    return await _context.Products
        .Include(p => p.OrderItems)
            .ThenInclude(oi => oi.Order)
        .FirstOrDefaultAsync(p => p.Id == productId);
}
```

#### Update Operations
```csharp
public async Task UpdateProductPriceAsync(int productId, decimal newPrice)
{
    var product = await _context.Products.FindAsync(productId);
    if (product != null)
    {
        product.Price = newPrice;
        await _context.SaveChangesAsync(); // EF Core tracks changes automatically
    }
}
```

#### Delete Operations
```csharp
public async Task DeleteProductAsync(int productId)
{
    var product = await _context.Products.FindAsync(productId);
    if (product != null)
    {
        _context.Products.Remove(product);
        await _context.SaveChangesAsync();
    }
}
```

### 6. Database Migrations

Create and apply migrations to manage database schema changes:

```bash
# Create a new migration
dotnet ef migrations add InitialCreate

# Update the database
dotnet ef database update

# Add a new migration for schema changes
dotnet ef migrations add AddProductCategory

# Update database with new migration
dotnet ef database update
```

---

## Without EF Core vs With EF Core

### Without EF Core (Raw SQL)
```csharp
// Tedious, error-prone SQL code
public async Task<List<Product>> GetProductsAsync()
{
    var products = new List<Product>();
    using var connection = new SqlConnection(connectionString);
    await connection.OpenAsync();
    
    var command = new SqlCommand("SELECT Id, Name, Price, Category FROM Products WHERE Price > @price", connection);
    command.Parameters.AddWithValue("@price", 100);
    
    using var reader = await command.ExecuteReaderAsync();
    while (await reader.ReadAsync())
    {
        products.Add(new Product
        {
            Id = reader.GetInt32("Id"),
            Name = reader.GetString("Name"),
            Price = reader.GetDecimal("Price"),
            Category = reader.GetString("Category")
        });
    }
    
    return products;
}
```

### With EF Core (Clean and Simple)
```csharp
// Clean, type-safe, maintainable code
public async Task<List<Product>> GetProductsAsync()
{
    return await _context.Products
        .Where(p => p.Price > 100)
        .ToListAsync();
}
```

---

## Advanced Features

### 1. Complex LINQ Queries
```csharp
// Get top 5 customers by total order value
var topCustomers = await _context.Orders
    .GroupBy(o => o.CustomerName)
    .Select(g => new 
    {
        CustomerName = g.Key,
        TotalOrders = g.Count(),
        TotalValue = g.Sum(o => o.TotalAmount)
    })
    .OrderByDescending(x => x.TotalValue)
    .Take(5)
    .ToListAsync();
```

### 2. Raw SQL When Needed
```csharp
// Execute raw SQL for complex scenarios
var productStats = await _context.Products
    .FromSqlRaw("SELECT * FROM Products WHERE Price > {0}", 100)
    .ToListAsync();
```

### 3. Stored Procedure Execution
```csharp
var salesReport = await _context.Database
    .SqlQueryRaw<SalesReportDto>("EXEC GetSalesReport @StartDate, @EndDate", 
        new SqlParameter("@StartDate", startDate),
        new SqlParameter("@EndDate", endDate))
    .ToListAsync();
```

---

## Benefits Summary

1. **Developer Productivity** - Write less code, focus on business logic
2. **Type Safety** - Compile-time error checking
3. **Maintainability** - Easier to refactor and update
4. **Database Agnostic** - Switch between different database providers
5. **Performance** - Built-in optimization and caching mechanisms
6. **Migration Management** - Version control for database schema
7. **Testing** - Easy to mock and unit test data access

---

## Best Practices

1. **Use async/await** for database operations
2. **Dispose DbContext properly** or use dependency injection
3. **Use Include() for eager loading** navigation properties
4. **Implement repository pattern** for better testability
5. **Use migrations** for schema changes instead of manual SQL scripts
6. **Monitor and optimize queries** using logging and profiling tools

Entity Framework Core transforms database development from a tedious, error-prone process into a productive, maintainable experience that lets developers focus on what matters most: building great applications.
