# Data Modeling with Entity Framework Core

## Main Intent

The primary intent of data modeling with Entity Framework Core is to **bridge the gap between object-oriented programming and relational database design**. It enables developers to:

- **Define database structure using C# classes** instead of writing SQL DDL statements
- **Establish relationships between entities** using navigation properties
- **Apply constraints and validations** through attributes and Fluent API
- **Generate database schema automatically** from code definitions
- **Maintain database evolution** through migrations

This approach allows developers to think in terms of objects while EF Core handles the translation to relational database concepts.

---

## Introduction

Data modeling with Entity Framework Core transforms how developers approach database design. Instead of starting with database tables and then creating corresponding classes, EF Core enables a code-first approach where you define your domain models as C# classes, and the database schema is generated automatically.

### Benefits of EF Core Data Modeling:
- **Type Safety** - Compile-time checking of data models
- **IntelliSense Support** - IDE assistance with property names and types
- **Refactoring Support** - Easy renaming and restructuring of models
- **Version Control** - Track changes to your data model in source control
- **Automated Migrations** - Database schema updates handled automatically

---

## Step-by-Step Guide to Entity Modeling

### Step 1: Create Project Structure

```
MyHRSystem/
├── Models/
│   ├── Employee.cs
│   ├── Department.cs
│   ├── Position.cs
│   └── Project.cs
├── Data/
│   └── HRDbContext.cs
└── Program.cs
```

Create a well-organized folder structure:

```csharp
// Create Models folder in your project
// This will contain all your entity classes
```

### Step 2: Define Basic Entity Classes

#### Employee Entity
```csharp
// Models/Employee.cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace MyHRSystem.Models
{
    public class Employee
    {
        // Primary Key
        public int EmployeeID { get; set; }
        
        // Basic Properties
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public string Email { get; set; }
        public DateTime HireDate { get; set; }
        public decimal Salary { get; set; }
        public bool IsActive { get; set; }
        
        // Foreign Key
        public int DepartmentID { get; set; }
        
        // Navigation Properties (will be configured later)
        public Department Department { get; set; }
        public ICollection<Project> Projects { get; set; } = new List<Project>();
    }
}
```

#### Department Entity
```csharp
// Models/Department.cs
namespace MyHRSystem.Models
{
    public class Department
    {
        public int DepartmentID { get; set; }
        public string DepartmentName { get; set; }
        public string Location { get; set; }
        public decimal Budget { get; set; }
        public DateTime CreatedDate { get; set; }
        
        // Navigation Properties
        public ICollection<Employee> Employees { get; set; } = new List<Employee>();
    }
}
```

#### Project Entity
```csharp
// Models/Project.cs
namespace MyHRSystem.Models
{
    public class Project
    {
        public int ProjectID { get; set; }
        public string ProjectName { get; set; }
        public string Description { get; set; }
        public DateTime StartDate { get; set; }
        public DateTime? EndDate { get; set; }
        public decimal Budget { get; set; }
        public string Status { get; set; }
        
        // Many-to-Many relationship with Employees
        public ICollection<Employee> Employees { get; set; } = new List<Employee>();
    }
}
```

### Step 3: Add Properties with Data Types

#### Understanding EF Core Data Type Mapping

```csharp
// Models/Employee.cs - Enhanced with proper data types
public class Employee
{
    // Integer types
    public int EmployeeID { get; set; }           // SQL: INT
    public long NationalID { get; set; }          // SQL: BIGINT
    
    // String types
    public string FirstName { get; set; }         // SQL: NVARCHAR(MAX) by default
    public string LastName { get; set; }
    public string Email { get; set; }
    public char Gender { get; set; }              // SQL: NCHAR(1)
    
    // Date and Time types
    public DateTime HireDate { get; set; }        // SQL: DATETIME2
    public DateOnly BirthDate { get; set; }       // SQL: DATE
    public TimeOnly WorkStartTime { get; set; }   // SQL: TIME
    
    // Decimal types
    public decimal Salary { get; set; }           // SQL: DECIMAL(18,2)
    public double HoursWorked { get; set; }       // SQL: FLOAT
    public float Rating { get; set; }             // SQL: REAL
    
    // Boolean type
    public bool IsActive { get; set; }            // SQL: BIT
    
    // Nullable types
    public DateTime? TerminationDate { get; set; } // SQL: DATETIME2 (nullable)
    public int? ManagerID { get; set; }           // SQL: INT (nullable)
    
    // Binary data
    public byte[] ProfilePicture { get; set; }    // SQL: VARBINARY(MAX)
    
    // GUID type
    public Guid ExternalID { get; set; }          // SQL: UNIQUEIDENTIFIER
    
    // Enum (will be stored as integer by default)
    public EmployeeStatus Status { get; set; }
    
    // Navigation Properties
    public int DepartmentID { get; set; }
    public Department Department { get; set; }
    public Employee Manager { get; set; }
    public ICollection<Employee> DirectReports { get; set; } = new List<Employee>();
}

public enum EmployeeStatus
{
    Active = 1,
    Inactive = 2,
    OnLeave = 3,
    Terminated = 4
}
```

### Step 4: Establish Relationships

#### One-to-Many Relationships

```csharp
// Department to Employees (One-to-Many)
public class Department
{
    public int DepartmentID { get; set; }
    public string DepartmentName { get; set; }
    
    // One Department has Many Employees
    public ICollection<Employee> Employees { get; set; } = new List<Employee>();
}

public class Employee  
{
    public int EmployeeID { get; set; }
    public string FirstName { get; set; }
    
    // Foreign Key
    public int DepartmentID { get; set; }
    
    // Navigation Property - Many Employees belong to One Department
    public Department Department { get; set; }
}
```

#### Self-Referencing Relationships

```csharp
// Employee to Manager (Self-Referencing One-to-Many)
public class Employee
{
    public int EmployeeID { get; set; }
    public string FirstName { get; set; }
    
    // Self-referencing Foreign Key
    public int? ManagerID { get; set; }
    
    // Navigation Properties
    public Employee Manager { get; set; }                          // One Manager
    public ICollection<Employee> DirectReports { get; set; }       // Many Direct Reports
}
```

#### One-to-One Relationships

```csharp
// Employee to EmployeeProfile (One-to-One)
public class Employee
{
    public int EmployeeID { get; set; }
    public string FirstName { get; set; }
    
    // One-to-One Navigation
    public EmployeeProfile Profile { get; set; }
}

public class EmployeeProfile
{
    public int EmployeeProfileID { get; set; }
    public int EmployeeID { get; set; }         // Foreign Key (also serves as alternate key)
    public string Bio { get; set; }
    public string LinkedIn { get; set; }
    public byte[] Resume { get; set; }
    
    // Navigation Property
    public Employee Employee { get; set; }
}
```

#### Many-to-Many Relationships

```csharp
// Modern EF Core approach (no junction entity needed)
public class Employee
{
    public int EmployeeID { get; set; }
    public string FirstName { get; set; }
    
    // Many-to-Many: Employees work on many Projects
    public ICollection<Project> Projects { get; set; } = new List<Project>();
}

public class Project
{
    public int ProjectID { get; set; }
    public string ProjectName { get; set; }
    
    // Many-to-Many: Projects have many Employees
    public ICollection<Employee> Employees { get; set; } = new List<Employee>();
}

// If you need additional data in the relationship, create explicit junction entity
public class ProjectAssignment
{
    public int EmployeeID { get; set; }
    public int ProjectID { get; set; }
    public DateTime AssignedDate { get; set; }
    public string Role { get; set; }
    public decimal HourlyRate { get; set; }
    
    // Navigation Properties
    public Employee Employee { get; set; }
    public Project Project { get; set; }
}
```

### Step 5: Configure with Data Annotations

Data annotations provide a declarative way to configure your entities directly on the properties.

```csharp
// Models/Employee.cs - With Data Annotations
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

public class Employee
{
    [Key]                                    // Explicit primary key
    public int EmployeeID { get; set; }
    
    [Required]                               // NOT NULL constraint
    [StringLength(50)]                       // VARCHAR(50)
    public string FirstName { get; set; }
    
    [Required]
    [StringLength(50)]
    public string LastName { get; set; }
    
    [Required]
    [StringLength(100)]
    [EmailAddress]                           // Email format validation
    public string Email { get; set; }
    
    [Column(TypeName = "decimal(10,2)")]     // Explicit SQL type
    public decimal Salary { get; set; }
    
    [DataType(DataType.Date)]                // Date only (no time)
    public DateTime HireDate { get; set; }
    
    [StringLength(15)]
    [Phone]                                  // Phone format validation
    public string PhoneNumber { get; set; }
    
    [Range(18, 65)]                         // Age validation
    public int Age { get; set; }
    
    // Foreign Key with explicit naming
    [ForeignKey("Department")]
    public int DepartmentID { get; set; }
    
    // Navigation Property
    public Department Department { get; set; }
    
    // Computed property (not mapped to database)
    [NotMapped]
    public string FullName => $"{FirstName} {LastName}";
    
    // Index configuration
    [Index(nameof(Email), IsUnique = true)]  // Unique index on Email
    public class Employee { }                 // This goes on the class level
}
```

#### Department with Annotations

```csharp
// Models/Department.cs - With Data Annotations
[Table("Departments")]                       // Custom table name
[Index(nameof(DepartmentName), IsUnique = true)]
public class Department
{
    [Key]
    public int DepartmentID { get; set; }
    
    [Required]
    [StringLength(100)]
    public string DepartmentName { get; set; }
    
    [StringLength(200)]
    public string Location { get; set; }
    
    [Column(TypeName = "decimal(12,2)")]
    [Range(0, double.MaxValue, ErrorMessage = "Budget must be positive")]
    public decimal Budget { get; set; }
    
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]  // Auto-generated
    public DateTime CreatedDate { get; set; }
    
    // Navigation Property
    public ICollection<Employee> Employees { get; set; } = new List<Employee>();
}
```

### Step 6: Use Fluent API for Advanced Configuration

The Fluent API provides more powerful and flexible configuration options through the `OnModelCreating` method.

```csharp
// Data/HRDbContext.cs
using Microsoft.EntityFrameworkCore;
using MyHRSystem.Models;

public class HRDbContext : DbContext
{
    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }
    public DbSet<Project> Projects { get; set; }
    public DbSet<EmployeeProfile> EmployeeProfiles { get; set; }
    
    public HRDbContext(DbContextOptions<HRDbContext> options) : base(options)
    {
    }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Employee Configuration
        ConfigureEmployee(modelBuilder);
        
        // Department Configuration  
        ConfigureDepartment(modelBuilder);
        
        // Project Configuration
        ConfigureProject(modelBuilder);
        
        // Relationship Configuration
        ConfigureRelationships(modelBuilder);
        
        // Seed Data
        SeedData(modelBuilder);
        
        base.OnModelCreating(modelBuilder);
    }
    
    private void ConfigureEmployee(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Employee>(entity =>
        {
            // Table configuration
            entity.ToTable("Employees");
            
            // Primary Key
            entity.HasKey(e => e.EmployeeID);
            
            // Property configurations
            entity.Property(e => e.FirstName)
                .IsRequired()
                .HasMaxLength(50)
                .IsUnicode(true);
                
            entity.Property(e => e.LastName)
                .IsRequired()
                .HasMaxLength(50);
                
            entity.Property(e => e.Email)
                .IsRequired()
                .HasMaxLength(100);
                
            entity.Property(e => e.Salary)
                .HasColumnType("decimal(10,2)")
                .HasDefaultValue(0);
                
            entity.Property(e => e.HireDate)
                .HasDefaultValueSql("GETDATE()");
                
            entity.Property(e => e.IsActive)
                .HasDefaultValue(true);
                
            // Computed column
            entity.Property(e => e.FullName)
                .HasComputedColumnSql("[FirstName] + ' ' + [LastName]");
                
            // Indexes
            entity.HasIndex(e => e.Email)
                .IsUnique()
                .HasDatabaseName("IX_Employee_Email");
                
            entity.HasIndex(e => new { e.FirstName, e.LastName })
                .HasDatabaseName("IX_Employee_Name");
                
            // Constraints
            entity.HasCheckConstraint("CK_Employee_Salary", "Salary >= 0");
            entity.HasCheckConstraint("CK_Employee_Age", "Age >= 18 AND Age <= 65");
        });
    }
    
    private void ConfigureDepartment(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Department>(entity =>
        {
            entity.HasKey(d => d.DepartmentID);
            
            entity.Property(d => d.DepartmentName)
                .IsRequired()
                .HasMaxLength(100);
                
            entity.Property(d => d.Budget)
                .HasColumnType("decimal(12,2)")
                .HasDefaultValue(0);
                
            entity.HasIndex(d => d.DepartmentName)
                .IsUnique();
        });
    }
    
    private void ConfigureProject(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Project>(entity =>
        {
            entity.HasKey(p => p.ProjectID);
            
            entity.Property(p => p.ProjectName)
                .IsRequired()
                .HasMaxLength(200);
                
            entity.Property(p => p.Status)
                .HasMaxLength(20)
                .HasDefaultValue("Planning");
                
            entity.Property(p => p.Budget)
                .HasColumnType("decimal(15,2)");
        });
    }
    
    private void ConfigureRelationships(ModelBuilder modelBuilder)
    {
        // Employee to Department (One-to-Many)
        modelBuilder.Entity<Employee>()
            .HasOne(e => e.Department)
            .WithMany(d => d.Employees)
            .HasForeignKey(e => e.DepartmentID)
            .OnDelete(DeleteBehavior.Restrict); // Prevent cascading deletes
            
        // Employee to Manager (Self-Referencing)
        modelBuilder.Entity<Employee>()
            .HasOne(e => e.Manager)
            .WithMany(e => e.DirectReports)
            .HasForeignKey(e => e.ManagerID)
            .OnDelete(DeleteBehavior.Restrict);
            
        // Employee to EmployeeProfile (One-to-One)
        modelBuilder.Entity<Employee>()
            .HasOne(e => e.Profile)
            .WithOne(p => p.Employee)
            .HasForeignKey<EmployeeProfile>(p => p.EmployeeID)
            .OnDelete(DeleteBehavior.Cascade);
            
        // Employee to Project (Many-to-Many)
        modelBuilder.Entity<Employee>()
            .HasMany(e => e.Projects)
            .WithMany(p => p.Employees)
            .UsingEntity<Dictionary<string, object>>(
                "EmployeeProject",
                j => j.HasOne<Project>().WithMany().HasForeignKey("ProjectID"),
                j => j.HasOne<Employee>().WithMany().HasForeignKey("EmployeeID"),
                j =>
                {
                    j.HasKey("EmployeeID", "ProjectID");
                    j.Property<DateTime>("AssignedDate").HasDefaultValueSql("GETDATE()");
                });
    }
    
    private void SeedData(ModelBuilder modelBuilder)
    {
        // Seed Departments
        modelBuilder.Entity<Department>().HasData(
            new Department { DepartmentID = 1, DepartmentName = "Engineering", Location = "Building A", Budget = 1000000, CreatedDate = DateTime.Now },
            new Department { DepartmentID = 2, DepartmentName = "Marketing", Location = "Building B", Budget = 500000, CreatedDate = DateTime.Now },
            new Department { DepartmentID = 3, DepartmentName = "Human Resources", Location = "Building C", Budget = 300000, CreatedDate = DateTime.Now }
        );
        
        // Seed Employees
        modelBuilder.Entity<Employee>().HasData(
            new Employee 
            { 
                EmployeeID = 1, 
                FirstName = "John", 
                LastName = "Doe", 
                Email = "john.doe@company.com",
                HireDate = new DateTime(2020, 1, 15),
                Salary = 75000,
                IsActive = true,
                DepartmentID = 1,
                Age = 30
            },
            new Employee 
            { 
                EmployeeID = 2, 
                FirstName = "Jane", 
                LastName = "Smith", 
                Email = "jane.smith@company.com",
                HireDate = new DateTime(2019, 3, 20),
                Salary = 68000,
                IsActive = true,
                DepartmentID = 2,
                ManagerID = 1,
                Age = 28
            }
        );
    }
}
```

### Step 7: Configure DbContext in Program.cs

```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;
using MyHRSystem.Data;

var builder = WebApplication.CreateBuilder(args);

// Add Entity Framework
builder.Services.AddDbContext<HRDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Add other services
builder.Services.AddControllers();

var app = builder.Build();

// Configure middleware
app.UseRouting();
app.MapControllers();

app.Run();
```

### Step 8: Generate and Apply Migrations

```bash
# Create initial migration
dotnet ef migrations add InitialCreate

# Apply migration to database
dotnet ef database update

# Add new migration for changes
dotnet ef migrations add AddEmployeeProfiles

# Update database with new migration
dotnet ef database update
```

---

## Data Annotations vs Fluent API Comparison

| Configuration | Data Annotations | Fluent API |
|---------------|------------------|------------|
| Primary Key | `[Key]` | `HasKey()` |
| Required Field | `[Required]` | `IsRequired()` |
| String Length | `[StringLength(50)]` | `HasMaxLength(50)` |
| Column Type | `[Column(TypeName = "decimal(18,2)")]` | `HasColumnType("decimal(18,2)")` |
| Foreign Key | `[ForeignKey("Department")]` | `HasForeignKey()` |
| Table Name | `[Table("Employees")]` | `ToTable("Employees")` |
| Index | `[Index(nameof(Email), IsUnique = true)]` | `HasIndex().IsUnique()` |
| Relationships | Limited support | Full relationship configuration |
| Computed Columns | Not supported | `HasComputedColumnSql()` |
| Check Constraints | Not supported | `HasCheckConstraint()` |
| Default Values | Not supported | `HasDefaultValue()` |

---

## Advanced Configuration Examples

### Custom Value Conversions

```csharp
// Convert enum to string in database
modelBuilder.Entity<Employee>()
    .Property(e => e.Status)
    .HasConversion<string>();

// Custom value converter for encrypted data
modelBuilder.Entity<Employee>()
    .Property(e => e.SSN)
    .HasConversion(
        v => EncryptionHelper.Encrypt(v),
        v => EncryptionHelper.Decrypt(v));
```

### Owned Types

```csharp
// Address as owned type (stored in same table)
public class Employee
{
    public int EmployeeID { get; set; }
    public string Name { get; set; }
    public Address HomeAddress { get; set; }
}

public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public string ZipCode { get; set; }
}

// Configuration
modelBuilder.Entity<Employee>()
    .OwnsOne(e => e.HomeAddress, address =>
    {
        address.Property(a => a.Street).HasColumnName("HomeStreet");
        address.Property(a => a.City).HasColumnName("HomeCity");
        address.Property(a => a.ZipCode).HasColumnName("HomeZipCode");
    });
```

### Global Query Filters

```csharp
// Apply soft delete filter globally
modelBuilder.Entity<Employee>()
    .HasQueryFilter(e => e.IsActive);

// This filter is automatically applied to all queries
// SELECT * FROM Employees WHERE IsActive = 1
```

---

## Best Practices

### 1. Entity Design
- **Use meaningful names** for entities and properties
- **Follow C# naming conventions** (PascalCase for classes and properties)
- **Keep entities focused** - each entity should represent a single concept
- **Use nullable types appropriately** for optional fields

### 2. Navigation Properties
- **Initialize collections** in constructors or with property initializers
- **Use ICollection<T>** for one-to-many relationships
- **Be consistent** with navigation property naming

### 3. Configuration Strategy
- **Use Data Annotations** for simple configurations
- **Use Fluent API** for complex scenarios and relationships
- **Organize configuration** using separate configuration classes for large models
- **Document complex configurations** with comments

### 4. Performance Considerations
- **Use indexes strategically** on frequently queried columns
- **Consider composite indexes** for multi-column queries
- **Avoid over-indexing** as it impacts insert/update performance
- **Use appropriate data types** to minimize storage

---

## Complete Working Example

Here's a comprehensive example showing all concepts together:

```csharp
// Models/CompleteExample.cs
public class Company
{
    public int CompanyID { get; set; }
    [Required, StringLength(100)]
    public string Name { get; set; }
    public Address HeadquartersAddress { get; set; }
    public ICollection<Department> Departments { get; set; } = new List<Department>();
}

public class Department
{
    public int DepartmentID { get; set; }
    [Required, StringLength(100)]
    public string Name { get; set; }
    public int CompanyID { get; set; }
    public Company Company { get; set; }
    public ICollection<Employee> Employees { get; set; } = new List<Employee>();
}

public class Employee
{
    public int EmployeeID { get; set; }
    [Required, StringLength(50)]
    public string FirstName { get; set; }
    [Required, StringLength(50)]
    public string LastName { get; set; }
    [Required, EmailAddress]
    public string Email { get; set; }
    public DateTime HireDate { get; set; }
    [Column(TypeName = "decimal(10,2)")]
    public decimal Salary { get; set; }
    public EmployeeStatus Status { get; set; }
    
    // Relationships
    public int DepartmentID { get; set; }
    public Department Department { get; set; }
    public int? ManagerID { get; set; }
    public Employee Manager { get; set; }
    public ICollection<Employee> DirectReports { get; set; } = new List<Employee>();
    public EmployeeProfile Profile { get; set; }
    public ICollection<ProjectAssignment> ProjectAssignments { get; set; } = new List<ProjectAssignment>();
}

public class EmployeeProfile
{
    public int EmployeeProfileID { get; set; }
    public int EmployeeID { get; set; }
    public Employee Employee { get; set; }
    public string Biography { get; set; }
    public string Skills { get; set; }
    public byte[] Resume { get; set; }
}

public class Project
{
    public int ProjectID { get; set; }
    [Required, StringLength(200)]
    public string Name { get; set; }
    public DateTime StartDate { get; set; }
    public DateTime? EndDate { get; set; }
    [Column(TypeName = "decimal(15,2)")]
    public decimal Budget { get; set; }
    public ProjectStatus Status { get; set; }
    
    public ICollection<ProjectAssignment> ProjectAssignments { get; set; } = new List<ProjectAssignment>();
}

public class ProjectAssignment
{
    public int EmployeeID { get; set; }
    public Employee Employee { get; set; }
    public int ProjectID { get; set; }
    public Project Project { get; set; }
    public DateTime AssignedDate { get; set; }
    public string Role { get; set; }
    [Column(TypeName = "decimal(8,2)")]
    public decimal HourlyRate { get; set; }
}

[Owned]
public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public string State { get; set; }
    public string ZipCode { get; set; }
    public string Country { get; set; }
}

public enum EmployeeStatus
{
    Active,
    Inactive,
    OnLeave,
    Terminated
}

public enum ProjectStatus
{
    Planning,
    Active,
    OnHold,
    Completed,
    Cancelled
}
```

This comprehensive guide demonstrates how Entity Framework Core transforms database design from a table-centric approach to an object-oriented methodology, making data modeling more intuitive and maintainable for .NET developers.
