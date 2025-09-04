# Dependency Injection in ASP.NET Core Web API - Complete Guide

## Introduction

Dependency Injection (DI) is a core design pattern in ASP.NET Core, enabling applications to achieve greater modularity, flexibility, and ease of maintenance. By allowing dependencies to be injected externally, DI minimizes direct dependencies between classes and external services, fostering a more adaptable and resilient codebase.

## Key Concepts of Dependency Injection

### Dependencies and Services
A dependency is any external service or object a class relies on to function. In ASP.NET Core, DI manages these dependencies by supplying required services to classes from an external source, which promotes flexibility in code structure and enables smooth integration of various services.

### Tight vs. Loose Coupling
- **Tightly coupled systems**: Components are interdependent, making code more challenging to maintain
- **Loosely coupled systems**: DI promotes loose coupling by injecting dependencies from external sources, reducing the risk of system-wide changes due to updates in a single component

## Benefits of Using Dependency Injection

- **Improved flexibility**: Swap out implementations easily without modifying core code
- **Enhanced performance**: Efficiently manages service instances, optimizing performance
- **Simplified maintenance**: Updates to services impact only areas where they're explicitly injected
- **Better testability**: Enables easy mocking and unit testing

## Service Lifetimes in ASP.NET Core

ASP.NET Core provides three service lifetimes:

1. **Transient**: Created each time they're requested
2. **Scoped**: Created once per request
3. **Singleton**: Created once for the application lifetime

## Practical Examples

### 1. Basic Service Interface and Implementation

First, let's create a simple service interface:

```csharp
// Interfaces/IUserService.cs
public interface IUserService
{
    Task<User> GetUserByIdAsync(int id);
    Task<IEnumerable<User>> GetAllUsersAsync();
    Task<User> CreateUserAsync(User user);
    Task<bool> DeleteUserAsync(int id);
}
```

```csharp
// Models/User.cs
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

### 2. Service Implementation

```csharp
// Services/UserService.cs
public class UserService : IUserService
{
    private readonly ILogger<UserService> _logger;
    private readonly List<User> _users; // In-memory storage for demo

    public UserService(ILogger<UserService> logger)
    {
        _logger = logger;
        _users = new List<User>
        {
            new User { Id = 1, Name = "John Doe", Email = "john@example.com", CreatedAt = DateTime.UtcNow },
            new User { Id = 2, Name = "Jane Smith", Email = "jane@example.com", CreatedAt = DateTime.UtcNow }
        };
    }

    public async Task<User> GetUserByIdAsync(int id)
    {
        _logger.LogInformation("Getting user with ID: {UserId}", id);
        
        var user = _users.FirstOrDefault(u => u.Id == id);
        if (user == null)
        {
            _logger.LogWarning("User with ID {UserId} not found", id);
        }
        
        return await Task.FromResult(user);
    }

    public async Task<IEnumerable<User>> GetAllUsersAsync()
    {
        _logger.LogInformation("Getting all users");
        return await Task.FromResult(_users);
    }

    public async Task<User> CreateUserAsync(User user)
    {
        _logger.LogInformation("Creating new user: {UserName}", user.Name);
        
        user.Id = _users.Max(u => u.Id) + 1;
        user.CreatedAt = DateTime.UtcNow;
        _users.Add(user);
        
        return await Task.FromResult(user);
    }

    public async Task<bool> DeleteUserAsync(int id)
    {
        _logger.LogInformation("Deleting user with ID: {UserId}", id);
        
        var user = _users.FirstOrDefault(u => u.Id == id);
        if (user != null)
        {
            _users.Remove(user);
            return await Task.FromResult(true);
        }
        
        return await Task.FromResult(false);
    }
}
```

### 3. Controller with Dependency Injection

```csharp
// Controllers/UsersController.cs
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly ILogger<UsersController> _logger;

    // Constructor injection
    public UsersController(IUserService userService, ILogger<UsersController> logger)
    {
        _userService = userService;
        _logger = logger;
    }

    [HttpGet]
    public async Task<ActionResult<IEnumerable<User>>> GetUsers()
    {
        try
        {
            var users = await _userService.GetAllUsersAsync();
            return Ok(users);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error occurred while getting users");
            return StatusCode(500, "Internal server error");
        }
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<User>> GetUser(int id)
    {
        try
        {
            var user = await _userService.GetUserByIdAsync(id);
            
            if (user == null)
            {
                return NotFound($"User with ID {id} not found");
            }
            
            return Ok(user);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error occurred while getting user {UserId}", id);
            return StatusCode(500, "Internal server error");
        }
    }

    [HttpPost]
    public async Task<ActionResult<User>> CreateUser([FromBody] User user)
    {
        try
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            var createdUser = await _userService.CreateUserAsync(user);
            return CreatedAtAction(nameof(GetUser), new { id = createdUser.Id }, createdUser);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error occurred while creating user");
            return StatusCode(500, "Internal server error");
        }
    }

    [HttpDelete("{id}")]
    public async Task<ActionResult> DeleteUser(int id)
    {
        try
        {
            var deleted = await _userService.DeleteUserAsync(id);
            
            if (!deleted)
            {
                return NotFound($"User with ID {id} not found");
            }
            
            return NoContent();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error occurred while deleting user {UserId}", id);
            return StatusCode(500, "Internal server error");
        }
    }
}
```

### 4. Service Registration in Program.cs

```csharp
// Program.cs
using Microsoft.OpenApi.Models;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();

// Register custom services with different lifetimes
builder.Services.AddScoped<IUserService, UserService>(); // Scoped lifetime
// builder.Services.AddTransient<IUserService, UserService>(); // Transient lifetime
// builder.Services.AddSingleton<IUserService, UserService>(); // Singleton lifetime

// Add Swagger/OpenAPI
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo 
    { 
        Title = "User Management API", 
        Version = "v1",
        Description = "A simple example ASP.NET Core Web API with Dependency Injection"
    });
});

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

## Advanced Examples

### 1. Multiple Service Implementations

```csharp
// Services/DatabaseUserService.cs
public class DatabaseUserService : IUserService
{
    private readonly ILogger<DatabaseUserService> _logger;
    
    public DatabaseUserService(ILogger<DatabaseUserService> logger)
    {
        _logger = logger;
    }
    
    public async Task<User> GetUserByIdAsync(int id)
    {
        _logger.LogInformation("Getting user from database with ID: {UserId}", id);
        // Database implementation here
        throw new NotImplementedException("Database implementation");
    }
    
    // ... other methods
}

// Register different implementations based on configuration
if (builder.Configuration.GetValue<bool>("UseInMemoryStorage"))
{
    builder.Services.AddScoped<IUserService, UserService>();
}
else
{
    builder.Services.AddScoped<IUserService, DatabaseUserService>();
}
```

### 2. Service Factory Pattern

```csharp
// Interfaces/IUserServiceFactory.cs
public interface IUserServiceFactory
{
    IUserService CreateUserService(string serviceType);
}

// Services/UserServiceFactory.cs
public class UserServiceFactory : IUserServiceFactory
{
    private readonly IServiceProvider _serviceProvider;

    public UserServiceFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public IUserService CreateUserService(string serviceType)
    {
        return serviceType.ToLower() switch
        {
            "memory" => _serviceProvider.GetRequiredService<UserService>(),
            "database" => _serviceProvider.GetRequiredService<DatabaseUserService>(),
            _ => throw new ArgumentException($"Unknown service type: {serviceType}")
        };
    }
}
```

## Dependency Injection in Unit Testing

### 1. Unit Test Setup

```csharp
// Tests/UserControllerTests.cs
using Microsoft.Extensions.Logging;
using Microsoft.AspNetCore.Mvc;
using Moq;
using Xunit;

public class UsersControllerTests
{
    private readonly Mock<IUserService> _mockUserService;
    private readonly Mock<ILogger<UsersController>> _mockLogger;
    private readonly UsersController _controller;

    public UsersControllerTests()
    {
        _mockUserService = new Mock<IUserService>();
        _mockLogger = new Mock<ILogger<UsersController>>();
        _controller = new UsersController(_mockUserService.Object, _mockLogger.Object);
    }

    [Fact]
    public async Task GetUsers_ReturnsOkResult_WithListOfUsers()
    {
        // Arrange
        var expectedUsers = new List<User>
        {
            new User { Id = 1, Name = "Test User 1", Email = "test1@example.com" },
            new User { Id = 2, Name = "Test User 2", Email = "test2@example.com" }
        };
        
        _mockUserService.Setup(s => s.GetAllUsersAsync())
                       .ReturnsAsync(expectedUsers);

        // Act
        var result = await _controller.GetUsers();

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result.Result);
        var returnedUsers = Assert.IsAssignableFrom<IEnumerable<User>>(okResult.Value);
        Assert.Equal(2, returnedUsers.Count());
    }

    [Fact]
    public async Task GetUser_WithValidId_ReturnsOkResult()
    {
        // Arrange
        var userId = 1;
        var expectedUser = new User { Id = userId, Name = "Test User", Email = "test@example.com" };
        
        _mockUserService.Setup(s => s.GetUserByIdAsync(userId))
                       .ReturnsAsync(expectedUser);

        // Act
        var result = await _controller.GetUser(userId);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result.Result);
        var returnedUser = Assert.IsType<User>(okResult.Value);
        Assert.Equal(expectedUser.Id, returnedUser.Id);
        Assert.Equal(expectedUser.Name, returnedUser.Name);
    }

    [Fact]
    public async Task GetUser_WithInvalidId_ReturnsNotFound()
    {
        // Arrange
        var userId = 999;
        _mockUserService.Setup(s => s.GetUserByIdAsync(userId))
                       .ReturnsAsync((User)null);

        // Act
        var result = await _controller.GetUser(userId);

        // Assert
        Assert.IsType<NotFoundObjectResult>(result.Result);
    }
}
```

### 2. Integration Testing with TestServer

```csharp
// Tests/IntegrationTests.cs
public class UsersControllerIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;

    public UsersControllerIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = _factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Override services for testing
                services.AddScoped<IUserService, UserService>();
            });
        }).CreateClient();
    }

    [Fact]
    public async Task Get_Users_ReturnsSuccessAndCorrectContentType()
    {
        // Act
        var response = await _client.GetAsync("/api/users");

        // Assert
        response.EnsureSuccessStatusCode();
        Assert.Equal("application/json; charset=utf-8",
            response.Content.Headers.ContentType.ToString());
    }
}
```

## Best Practices

1. **Use interfaces**: Always define interfaces for your services to enable loose coupling
2. **Choose appropriate lifetimes**: 
   - Use `Scoped` for services that maintain state within a request
   - Use `Transient` for lightweight, stateless services
   - Use `Singleton` for expensive-to-create, thread-safe services
3. **Avoid service locator pattern**: Don't inject `IServiceProvider` directly
4. **Register dependencies in the right order**: Dependencies should be registered before the services that depend on them
5. **Use configuration objects**: Instead of injecting `IConfiguration` directly, use strongly-typed configuration objects

## Conclusion

Dependency Injection in ASP.NET Core is essential for developing flexible, maintainable, high-performance applications. DI facilitates loose coupling and modularity, allowing applications to adapt to new requirements seamlessly. Additionally, DI supports robust testing practices through mock services, making it easier to build reliable, testable code that can evolve efficiently as application needs change.

The examples provided demonstrate how to implement DI in real-world scenarios, from basic service registration to advanced patterns and comprehensive testing strategies.
