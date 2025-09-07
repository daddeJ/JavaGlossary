# ASP.NET Core Error Handling and Logging Best Practices

## Introduction

Error handling and logging are fundamental aspects of building robust, maintainable ASP.NET Core applications. This guide provides comprehensive best practices with practical examples to help you implement effective error management and logging strategies.

## Error Handling Best Practices

### 1. Use Try-Catch Blocks for Known Exceptions

Handle specific exceptions gracefully to prevent application crashes and provide meaningful feedback to users.

```csharp
[HttpGet("{id}")]
public async Task<IActionResult> GetUser(int id)
{
    try
    {
        var user = await _userService.GetUserByIdAsync(id);
        if (user == null)
        {
            return NotFound($"User with ID {id} not found.");
        }
        return Ok(user);
    }
    catch (ArgumentException ex)
    {
        _logger.LogWarning("Invalid user ID provided: {UserId}. Error: {Error}", id, ex.Message);
        return BadRequest("Invalid user ID provided.");
    }
    catch (SqlException ex)
    {
        _logger.LogError(ex, "Database error occurred while fetching user {UserId}", id);
        return StatusCode(500, "A database error occurred. Please try again later.");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Unexpected error occurred while fetching user {UserId}", id);
        return StatusCode(500, "An unexpected error occurred.");
    }
}
```

### 2. Implement Global Exception Handling Middleware

Create centralized error handling to catch unhandled exceptions across your application.

```csharp
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionMiddleware> _logger;

    public GlobalExceptionMiddleware(RequestDelegate next, ILogger<GlobalExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An unhandled exception occurred. Request: {Method} {Path}", 
                context.Request.Method, context.Request.Path);
            
            await HandleExceptionAsync(context, ex);
        }
    }

    private static async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/json";
        
        var response = new ErrorResponse();

        switch (exception)
        {
            case ValidationException validationEx:
                response.StatusCode = (int)HttpStatusCode.BadRequest;
                response.Message = "Validation failed";
                response.Details = validationEx.Message;
                break;
            
            case NotFoundException notFoundEx:
                response.StatusCode = (int)HttpStatusCode.NotFound;
                response.Message = "Resource not found";
                response.Details = notFoundEx.Message;
                break;
            
            case UnauthorizedAccessException:
                response.StatusCode = (int)HttpStatusCode.Unauthorized;
                response.Message = "Unauthorized access";
                break;
            
            default:
                response.StatusCode = (int)HttpStatusCode.InternalServerError;
                response.Message = "An internal server error occurred";
                break;
        }

        context.Response.StatusCode = response.StatusCode;
        
        var jsonResponse = JsonSerializer.Serialize(response, new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        });
        
        await context.Response.WriteAsync(jsonResponse);
    }
}

public class ErrorResponse
{
    public int StatusCode { get; set; }
    public string Message { get; set; }
    public string Details { get; set; }
    public DateTime Timestamp { get; set; } = DateTime.UtcNow;
}
```

### 3. Register Global Exception Middleware

Add the middleware to your application pipeline in `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddLogging();

var app = builder.Build();

// Configure pipeline
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseMiddleware<GlobalExceptionMiddleware>();
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseRouting();
app.MapControllers();

app.Run();
```

### 4. Create Custom Exception Types

Define specific exception types for better error categorization:

```csharp
public class NotFoundException : Exception
{
    public NotFoundException(string message) : base(message) { }
    public NotFoundException(string message, Exception innerException) : base(message, innerException) { }
}

public class ValidationException : Exception
{
    public ValidationException(string message) : base(message) { }
    public ValidationException(string message, Exception innerException) : base(message, innerException) { }
}

public class BusinessLogicException : Exception
{
    public BusinessLogicException(string message) : base(message) { }
    public BusinessLogicException(string message, Exception innerException) : base(message, innerException) { }
}
```

## Logging Best Practices

### 1. Configure Logging Levels in appsettings.json

Set up different log levels for various environments:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore.Database.Command": "Warning",
      "System.Net.Http.HttpClient": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

**Development environment (appsettings.Development.json):**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Information",
      "Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
  }
}
```

**Production environment (appsettings.Production.json):**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "Microsoft.AspNetCore": "Error",
      "Microsoft.EntityFrameworkCore.Database.Command": "Error"
    }
  }
}
```

### 2. Use Structured Logging with Parameters

Always use structured logging with parameters instead of string interpolation:

```csharp
public class UserService
{
    private readonly ILogger<UserService> _logger;
    private readonly IUserRepository _userRepository;

    public UserService(ILogger<UserService> logger, IUserRepository userRepository)
    {
        _logger = logger;
        _userRepository = userRepository;
    }

    public async Task<User> CreateUserAsync(CreateUserRequest request)
    {
        _logger.LogInformation("Creating user with email: {Email}", request.Email);

        try
        {
            // Validate request
            if (string.IsNullOrEmpty(request.Email))
            {
                _logger.LogWarning("User creation failed: Email is required");
                throw new ValidationException("Email is required");
            }

            // Check if user exists
            var existingUser = await _userRepository.GetByEmailAsync(request.Email);
            if (existingUser != null)
            {
                _logger.LogWarning("User creation failed: User with email {Email} already exists", request.Email);
                throw new ValidationException($"User with email {request.Email} already exists");
            }

            var user = new User
            {
                Email = request.Email,
                Name = request.Name,
                CreatedAt = DateTime.UtcNow
            };

            var createdUser = await _userRepository.CreateAsync(user);
            
            _logger.LogInformation("User created successfully with ID: {UserId} and Email: {Email}", 
                createdUser.Id, createdUser.Email);

            return createdUser;
        }
        catch (Exception ex) when (!(ex is ValidationException))
        {
            _logger.LogError(ex, "Failed to create user with email: {Email}", request.Email);
            throw;
        }
    }
}
```

### 3. Implement Log Scopes for Request Tracking

Use log scopes to group related log entries:

```csharp
[HttpPost]
public async Task<IActionResult> CreateUser([FromBody] CreateUserRequest request)
{
    using var scope = _logger.BeginScope("CreateUser-{RequestId}", Guid.NewGuid());
    
    _logger.LogInformation("Starting user creation process");
    
    try
    {
        var user = await _userService.CreateUserAsync(request);
        
        _logger.LogInformation("User creation completed successfully");
        return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
    }
    catch (ValidationException ex)
    {
        _logger.LogWarning("User creation failed due to validation: {Error}", ex.Message);
        return BadRequest(ex.Message);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "User creation failed unexpectedly");
        return StatusCode(500, "An error occurred while creating the user");
    }
}
```

### 4. Configure Advanced Logging with Serilog

Add Serilog for enhanced logging capabilities:

```csharp
// Program.cs
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Configure Serilog
builder.Host.UseSerilog((context, configuration) =>
    configuration
        .ReadFrom.Configuration(context.Configuration)
        .Enrich.FromLogContext()
        .Enrich.WithMachineName()
        .Enrich.WithThreadId()
        .WriteTo.Console()
        .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)
        .WriteTo.Seq("http://localhost:5341") // Optional: Seq for log analysis
);

// Add services
builder.Services.AddControllers();

var app = builder.Build();

// Add request logging middleware
app.UseSerilogRequestLogging(options =>
{
    options.MessageTemplate = "HTTP {RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.0000} ms";
    options.GetLevel = (httpContext, elapsed, ex) => ex != null 
        ? LogEventLevel.Error 
        : httpContext.Response.StatusCode > 499 
            ? LogEventLevel.Error 
            : LogEventLevel.Information;
    
    options.EnrichDiagnosticContext = (diagnosticContext, httpContext) =>
    {
        diagnosticContext.Set("RequestHost", httpContext.Request.Host.Value);
        diagnosticContext.Set("RequestScheme", httpContext.Request.Scheme);
        diagnosticContext.Set("UserAgent", httpContext.Request.Headers["User-Agent"].ToString());
    };
});

app.UseMiddleware<GlobalExceptionMiddleware>();
app.UseRouting();
app.MapControllers();

app.Run();
```

**Serilog configuration in appsettings.json:**
```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning",
        "Microsoft.Hosting.Lifetime": "Information"
      }
    },
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss} [{Level:u3}] {Message:lj} {Properties:j}{NewLine}{Exception}"
        }
      },
      {
        "Name": "File",
        "Args": {
          "path": "logs/log-.txt",
          "rollingInterval": "Day",
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss} [{Level:u3}] {Message:lj} {Properties:j}{NewLine}{Exception}"
        }
      }
    ]
  }
}
```

### 5. Performance Logging

Monitor and log performance metrics:

```csharp
public class PerformanceLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<PerformanceLoggingMiddleware> _logger;

    public PerformanceLoggingMiddleware(RequestDelegate next, ILogger<PerformanceLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();
        
        await _next(context);
        
        stopwatch.Stop();
        
        var elapsedMs = stopwatch.ElapsedMilliseconds;
        
        if (elapsedMs > 1000) // Log slow requests
        {
            _logger.LogWarning("Slow request detected: {Method} {Path} took {ElapsedMs}ms", 
                context.Request.Method, 
                context.Request.Path, 
                elapsedMs);
        }
        else
        {
            _logger.LogInformation("Request completed: {Method} {Path} in {ElapsedMs}ms", 
                context.Request.Method, 
                context.Request.Path, 
                elapsedMs);
        }
    }
}
```

## Additional Best Practices

### 1. Health Checks with Logging

Implement health checks to monitor application status:

```csharp
public class DatabaseHealthCheck : IHealthCheck
{
    private readonly IDbContext _dbContext;
    private readonly ILogger<DatabaseHealthCheck> _logger;

    public DatabaseHealthCheck(IDbContext dbContext, ILogger<DatabaseHealthCheck> logger)
    {
        _dbContext = dbContext;
        _logger = logger;
    }

    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context, 
        CancellationToken cancellationToken = default)
    {
        try
        {
            await _dbContext.Database.CanConnectAsync(cancellationToken);
            _logger.LogDebug("Database health check passed");
            return HealthCheckResult.Healthy("Database is healthy");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Database health check failed");
            return HealthCheckResult.Unhealthy("Database is unhealthy", ex);
        }
    }
}
```

### 2. Secure Logging Practices

Avoid logging sensitive information:

```csharp
public class SecureLoggingService
{
    private readonly ILogger<SecureLoggingService> _logger;

    public SecureLoggingService(ILogger<SecureLoggingService> logger)
    {
        _logger = logger;
    }

    public void LogUserAction(string userId, string action, object data = null)
    {
        // ❌ Don't log sensitive data
        // _logger.LogInformation("User {UserId} performed {Action} with data: {Data}", 
        //     userId, action, JsonSerializer.Serialize(data));

        // ✅ Log only necessary information
        _logger.LogInformation("User {UserId} performed {Action}", userId, action);
        
        // If you need to log data, sanitize it first
        if (data != null)
        {
            var sanitizedData = SanitizeForLogging(data);
            _logger.LogDebug("Action details: {SanitizedData}", sanitizedData);
        }
    }

    private object SanitizeForLogging(object data)
    {
        // Remove or mask sensitive fields
        var json = JsonSerializer.Serialize(data);
        var sanitized = json
            .Replace("\"password\":\"[^\"]*\"", "\"password\":\"***\"")
            .Replace("\"creditCard\":\"[^\"]*\"", "\"creditCard\":\"***\"");
        
        return JsonSerializer.Deserialize<object>(sanitized);
    }
}
```

## Conclusion

Implementing robust error handling and logging practices in ASP.NET Core applications ensures:

- **Reliability**: Applications handle errors gracefully without crashing
- **Maintainability**: Comprehensive logs make debugging and troubleshooting easier
- **Performance Monitoring**: Track application performance and identify bottlenecks
- **Security**: Proper error handling prevents information leakage
- **User Experience**: Users receive meaningful error messages instead of technical details

Remember to:
- Use structured logging with parameters
- Implement both local try-catch blocks and global exception handling
- Configure appropriate log levels for different environments
- Monitor performance and set up alerts for critical issues
- Keep sensitive information out of logs
- Regularly review and analyze logs to improve application quality

By following these best practices, you'll build more robust, maintainable, and monitorable ASP.NET Core applications.
