# CRUD Operations with EF Core - Intention and Examples

## Intention

The primary intention of implementing CRUD operations with Entity Framework Core is to:

1. **Provide a standardized approach** to data manipulation across applications
2. **Simplify database interactions** by abstracting complex SQL operations into intuitive C# methods
3. **Ensure data consistency** through transaction management and change tracking
4. **Enable efficient data access patterns** that follow object-relational mapping principles
5. **Maintain separation of concerns** between business logic and data access logic

## Practical Examples

### Setup: Sample Entity and DbContext

```csharp
// Entity Model
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
}

// DbContext
public class ShopContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("your-connection-string");
    }
}
```

### CREATE Operations

**Intention**: Add new records to the database

```csharp
// Adding a single product
using var context = new ShopContext();

var newProduct = new Product
{
    Name = "Laptop",
    Price = 999.99m,
    Category = "Electronics"
};

context.Products.Add(newProduct);
await context.SaveChangesAsync();

// Adding multiple products
var products = new List<Product>
{
    new Product { Name = "Mouse", Price = 29.99m, Category = "Electronics" },
    new Product { Name = "Keyboard", Price = 79.99m, Category = "Electronics" }
};

context.Products.AddRange(products);
await context.SaveChangesAsync();
```

### READ Operations

**Intention**: Retrieve data from the database for display or processing

```csharp
using var context = new ShopContext();

// Find by primary key
var product = await context.Products.FindAsync(1);

// Get first matching record
var firstElectronics = await context.Products
    .FirstOrDefaultAsync(p => p.Category == "Electronics");

// Get all records
var allProducts = await context.Products.ToListAsync();

// Get filtered records
var expensiveProducts = await context.Products
    .Where(p => p.Price > 100)
    .OrderBy(p => p.Name)
    .ToListAsync();

// Check if record exists
var exists = await context.Products
    .AnyAsync(p => p.Name == "Laptop");
```

### UPDATE Operations

**Intention**: Modify existing records with new data

```csharp
using var context = new ShopContext();

// Update single record
var product = await context.Products.FindAsync(1);
if (product != null)
{
    product.Price = 899.99m;
    product.Name = "Gaming Laptop";
    
    context.Products.Update(product);
    await context.SaveChangesAsync();
}

// Bulk update using ExecuteUpdate (EF Core 7+)
await context.Products
    .Where(p => p.Category == "Electronics")
    .ExecuteUpdateAsync(p => p.SetProperty(x => x.Price, x => x.Price * 0.9m));

// Update multiple records
var productsToUpdate = await context.Products
    .Where(p => p.Category == "Electronics")
    .ToListAsync();

foreach (var item in productsToUpdate)
{
    item.Price *= 0.95m; // Apply 5% discount
}

context.Products.UpdateRange(productsToUpdate);
await context.SaveChangesAsync();
```

### DELETE Operations

**Intention**: Remove unwanted or obsolete records from the database

```csharp
using var context = new ShopContext();

// Delete single record by finding it first
var product = await context.Products.FindAsync(1);
if (product != null)
{
    context.Products.Remove(product);
    await context.SaveChangesAsync();
}

// Delete multiple records
var productsToDelete = await context.Products
    .Where(p => p.Price < 10)
    .ToListAsync();

context.Products.RemoveRange(productsToDelete);
await context.SaveChangesAsync();

// Bulk delete using ExecuteDelete (EF Core 7+)
await context.Products
    .Where(p => p.Category == "Discontinued")
    .ExecuteDeleteAsync();
```

## Best Practices and Patterns

### Service Layer Pattern

```csharp
public class ProductService
{
    private readonly ShopContext _context;
    
    public ProductService(ShopContext context)
    {
        _context = context;
    }
    
    public async Task<Product> CreateProductAsync(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return product;
    }
    
    public async Task<IEnumerable<Product>> GetAllProductsAsync()
    {
        return await _context.Products.ToListAsync();
    }
    
    public async Task<Product?> UpdateProductAsync(int id, Product updatedProduct)
    {
        var product = await _context.Products.FindAsync(id);
        if (product == null) return null;
        
        product.Name = updatedProduct.Name;
        product.Price = updatedProduct.Price;
        product.Category = updatedProduct.Category;
        
        await _context.SaveChangesAsync();
        return product;
    }
    
    public async Task<bool> DeleteProductAsync(int id)
    {
        var product = await _context.Products.FindAsync(id);
        if (product == null) return false;
        
        _context.Products.Remove(product);
        await _context.SaveChangesAsync();
        return true;
    }
}
```

### Error Handling Example

```csharp
public async Task<Product> CreateProductSafelyAsync(Product product)
{
    try
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return product;
    }
    catch (DbUpdateException ex)
    {
        // Handle database-specific errors
        throw new InvalidOperationException("Failed to create product", ex);
    }
    catch (Exception ex)
    {
        // Handle general errors
        throw new ApplicationException("An error occurred while creating the product", ex);
    }
}
```

## Key Benefits

1. **Type Safety**: Compile-time checking prevents common SQL errors
2. **IntelliSense Support**: IDE assistance for method discovery and parameters
3. **Change Tracking**: Automatic detection of entity modifications
4. **Transaction Management**: Built-in support for database transactions
5. **Performance Optimization**: Query optimization and caching capabilities
6. **Cross-Database Compatibility**: Same code works with different database providers
