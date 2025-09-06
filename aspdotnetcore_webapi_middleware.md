# ASP.NET Core Middleware - Table of Contents & Categories

## Overview
This document organizes ASP.NET Core middleware components by their primary purpose and provides a comprehensive table of contents for easy navigation and understanding of the request pipeline architecture.

---

## Table of Contents

### 1. [Security & Protection Middleware](#1-security--protection-middleware)
- [Authentication Middleware](#authentication-middleware)
- [Authorization Middleware](#authorization-middleware) 
- [CORS Middleware](#cors-middleware)
- [Cookie Policy Middleware](#cookie-policy-middleware)
- [HTTPS & Transport Security](#https--transport-security)

### 2. [Request Processing & Routing](#2-request-processing--routing)
- [Routing Middleware](#routing-middleware)
- [Endpoint Mapping](#endpoint-mapping)
- [Static File Serving](#static-file-serving)

### 3. [Performance & Optimization](#3-performance--optimization)
- [Caching Middleware](#caching-middleware)
- [Compression Middleware](#compression-middleware)
- [Response Optimization](#response-optimization)

### 4. [Communication & Real-time](#4-communication--real-time)
- [WebSocket Support](#websocket-support)
- [Session Management](#session-management)

### 5. [Error Handling & Diagnostics](#5-error-handling--diagnostics)
- [Exception Handling](#exception-handling)
- [Status Code Pages](#status-code-pages)
- [Health Checks](#health-checks)
- [Development Tools](#development-tools)

### 6. [Infrastructure & Configuration](#6-infrastructure--configuration)
- [Proxy & Load Balancer Support](#proxy--load-balancer-support)
- [Localization & Internationalization](#localization--internationalization)
- [API Documentation](#api-documentation)

### 7. [Custom & Extensibility](#7-custom--extensibility)
- [Custom Middleware](#custom-middleware)
- [Middleware Pipeline Order](#middleware-pipeline-order)

---

## 1. Security & Protection Middleware

### Authentication Middleware
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseAuthentication` | Identifies and validates users | JWT tokens, cookies, OAuth providers |

### Authorization Middleware
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseAuthorization` | Enforces access control policies | Role-based, policy-based, claim-based authorization |

### CORS Middleware
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseCors` | Manages cross-origin resource sharing | API access from web browsers, SPA applications |

### Cookie Policy Middleware
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseCookiePolicy` | Handles cookie consent and compliance | GDPR compliance, privacy regulations |

### HTTPS & Transport Security
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseHttpsRedirection` | Redirects HTTP requests to HTTPS | Enforcing secure connections |
| `UseHsts` | Enforces HTTPS-only communication | Browser security, preventing downgrade attacks |

---

## 2. Request Processing & Routing

### Routing Middleware
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseRouting` | Matches requests to endpoints | URL pattern matching, route resolution |

### Endpoint Mapping
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `MapControllers` | Maps MVC controller actions | Traditional MVC applications |
| `MapGet/Post/Put/Delete` | Maps minimal API endpoints | Lightweight APIs, microservices |
| `UseEndpoints` | Configures endpoint execution | Custom endpoint routing |

### Static File Serving
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseStaticFiles` | Serves static content (CSS, JS, images) | Web applications, asset delivery |
| `UseDefaultFiles` | Serves default documents (index.html) | SPA applications, static websites |
| `UseFileServer` | Combined static files + directory browsing | File management applications |

---

## 3. Performance & Optimization

### Caching Middleware
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseResponseCaching` | Caches HTTP responses | API response caching, performance optimization |

### Compression Middleware
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseResponseCompression` | Compresses responses (gzip, brotli) | Bandwidth optimization, faster loading |

### Response Optimization
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| Response headers | Optimizes response metadata | Cache control, security headers |

---

## 4. Communication & Real-time

### WebSocket Support
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseWebSockets` | Enables WebSocket connections | Real-time chat, live updates, gaming |

### Session Management
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseSession` | Server-side session state | User preferences, shopping carts, temporary data |

---

## 5. Error Handling & Diagnostics

### Exception Handling
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseDeveloperExceptionPage` | Detailed error pages for development | Debugging, development environment |
| `UseExceptionHandler` | Global exception handling for production | User-friendly error pages, error logging |

### Status Code Pages
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseStatusCodePages` | Custom responses for HTTP status codes | 404 pages, error handling |

### Health Checks
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseHealthChecks` | Application health monitoring | Load balancer checks, monitoring systems |

### Development Tools
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| Error pages | Development-time diagnostics | Debugging, troubleshooting |

---

## 6. Infrastructure & Configuration

### Proxy & Load Balancer Support
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseForwardedHeaders` | Handles proxy headers | Reverse proxy deployments, load balancers |

### Localization & Internationalization
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseRequestLocalization` | Culture and language support | Multi-language applications, global apps |

### API Documentation
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseSwagger` | Generates OpenAPI specifications | API documentation generation |
| `UseSwaggerUI` | Interactive API documentation | Developer tools, API testing |

---

## 7. Custom & Extensibility

### Custom Middleware
| Middleware | Purpose | Primary Use Case |
|------------|---------|------------------|
| `UseMiddleware<T>` | Custom application logic | Logging, metrics, business logic |

---

## Middleware Categories by Purpose

### 🔐 Security-First Middleware
**Purpose:** Protect application from threats and ensure secure communication
- `UseHttpsRedirection`
- `UseHsts` 
- `UseAuthentication`
- `UseAuthorization`
- `UseCors`
- `UseCookiePolicy`

### 🚀 Performance-Focused Middleware
**Purpose:** Optimize response times and resource usage
- `UseResponseCompression`
- `UseResponseCaching`
- `UseStaticFiles`
- `UseDefaultFiles`

### 🛣️ Request Routing Middleware
**Purpose:** Direct requests to appropriate handlers
- `UseRouting`
- `MapControllers`
- `MapGet/Post/Put/Delete`
- `UseEndpoints`

### 🔧 Diagnostics & Monitoring Middleware
**Purpose:** Monitor application health and handle errors
- `UseDeveloperExceptionPage`
- `UseExceptionHandler`
- `UseStatusCodePages`
- `UseHealthChecks`

### 🌐 Communication Middleware
**Purpose:** Enable different communication patterns
- `UseWebSockets`
- `UseSession`

### ⚙️ Infrastructure Middleware  
**Purpose:** Handle deployment and operational concerns
- `UseForwardedHeaders`
- `UseRequestLocalization`
- `UseSwagger`
- `UseSwaggerUI`

---

## Recommended Pipeline Order

The middleware pipeline should follow this general order for optimal functionality:

```
1. Exception Handling (Development/Production)
2. Security Headers (HSTS, HTTPS Redirection)  
3. Static Files (Early for performance)
4. Forwarded Headers (Proxy support)
5. Routing
6. CORS (After routing, before authentication)
7. Authentication 
8. Authorization
9. Custom Business Logic Middleware
10. Session Management
11. Response Optimization (Caching, Compression)
12. Endpoint Execution
```

---

## Selection Guide by Application Type

### 🌐 Web API Applications
**Essential Middleware:**
- `UseRouting` + `MapControllers`
- `UseAuthentication` + `UseAuthorization`
- `UseCors`
- `UseResponseCompression`
- `UseSwagger` + `UseSwaggerUI`

### 📱 Single Page Applications (SPA)
**Essential Middleware:**
- `UseStaticFiles` + `UseDefaultFiles`
- `UseRouting`
- `UseCors`
- `UseAuthentication` + `UseAuthorization`

### 🌍 Multi-tenant Applications
**Essential Middleware:**
- `UseRequestLocalization`
- `UseAuthentication` + `UseAuthorization`
- `UseSession`
- Custom tenant resolution middleware

### 🔄 Real-time Applications
**Essential Middleware:**
- `UseWebSockets`
- `UseAuthentication` + `UseAuthorization`
- `UseCors`
- Custom connection management middleware

### 📊 Microservices
**Essential Middleware:**
- `UseHealthChecks`
- `UseAuthentication` + `UseAuthorization`
- `UseForwardedHeaders`
- `UseResponseCompression`
- Custom service discovery middleware

This categorization helps developers understand the purpose of each middleware component and select the appropriate ones based on their application requirements and architecture.

# ASP.NET Core Middleware Guide

## Introduction

Middleware in ASP.NET Core forms the request pipeline that handles HTTP requests and responses. Each middleware component can perform operations before and after the next component in the pipeline. This guide covers the most common middleware components, their purposes, and implementation examples.

---

## 🔹 Security & Authentication

### `UseAuthentication`
**Purpose:** Enables authentication mechanisms (cookies, JWT, OAuth, etc.) to identify users.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add authentication services
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = "your-issuer",
            ValidAudience = "your-audience",
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("your-secret-key"))
        };
    });

var app = builder.Build();

// Enable authentication middleware
app.UseAuthentication(); // Must come before UseAuthorization
app.UseAuthorization();

app.MapControllers();
app.Run();
```

### `UseAuthorization`
**Purpose:** Enforces authorization policies to determine if authenticated users can access specific resources.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add authorization services with policies
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));
    options.AddPolicy("MinimumAge", policy => policy.RequireClaim("Age", "18", "19", "20"));
});

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization(); // Enforces the authorization policies

// Example protected endpoint
app.MapGet("/admin", () => "Admin area")
   .RequireAuthorization("AdminOnly");

app.Run();
```

### `UseCookiePolicy`
**Purpose:** Manages cookie consent and policy compliance, especially for GDPR requirements.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.Configure<CookiePolicyOptions>(options =>
{
    options.CheckConsentNeeded = context => true;
    options.MinimumSameSitePolicy = SameSiteMode.None;
    options.ConsentCookie.Name = "cookie-consent";
    options.ConsentCookie.Expiration = TimeSpan.FromDays(365);
});

var app = builder.Build();

app.UseCookiePolicy(); // Apply cookie policy

app.MapGet("/", () => "Check browser for cookie consent banner");
app.Run();
```

### `UseCors`
**Purpose:** Configures Cross-Origin Resource Sharing to allow web pages from different domains to access your API.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add CORS services
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowSpecificOrigin", policy =>
    {
        policy.WithOrigins("https://localhost:3000", "https://myapp.com")
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials();
    });
    
    options.AddDefaultPolicy(policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

var app = builder.Build();

app.UseCors("AllowSpecificOrigin"); // Apply specific CORS policy
// or app.UseCors(); for default policy

app.MapControllers();
app.Run();
```

### `UseHsts`
**Purpose:** Adds HTTP Strict Transport Security header to force browsers to use HTTPS only.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    // Configure HSTS (HTTP Strict Transport Security)
    app.UseHsts(); // Adds Strict-Transport-Security header
}

app.UseHttpsRedirection();

app.MapGet("/", () => "This will force HTTPS in production");
app.Run();
```

### `UseHttpsRedirection`
**Purpose:** Automatically redirects all HTTP requests to HTTPS for security.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Configure HTTPS redirection options
builder.Services.AddHttpsRedirection(options =>
{
    options.RedirectStatusCode = StatusCodes.Status307TemporaryRedirect;
    options.HttpsPort = 5001;
});

var app = builder.Build();

app.UseHttpsRedirection(); // Redirect HTTP to HTTPS

app.MapGet("/", () => "This endpoint will redirect to HTTPS if accessed via HTTP");
app.Run();
```

---

## 🔹 Request/Response Handling

### `UseRouting`
**Purpose:** Matches HTTP requests to endpoints based on URL patterns.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();

var app = builder.Build();

app.UseRouting(); // Enable endpoint routing

// Define endpoints after UseRouting
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
    endpoints.MapGet("/custom", () => "Custom endpoint");
    endpoints.MapPost("/api/data", (DataModel data) => Results.Ok(data));
});

app.Run();
```

### `MapControllers` / `MapGet` / `UseEndpoints`
**Purpose:** Execute endpoint handlers for controllers, minimal APIs, and Razor Pages.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();

var app = builder.Build();

app.UseRouting();

// Map different types of endpoints
app.MapControllers(); // Map controller endpoints

// Minimal API endpoints
app.MapGet("/hello", () => "Hello World!");
app.MapPost("/users", (User user) => Results.Created($"/users/{user.Id}", user));
app.MapPut("/users/{id}", (int id, User user) => Results.Ok(user));
app.MapDelete("/users/{id}", (int id) => Results.NoContent());

app.Run();
```

### `UseStaticFiles`
**Purpose:** Serves static files (HTML, CSS, JS, images) from the wwwroot directory.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

// Serve static files from wwwroot
app.UseStaticFiles();

// Serve static files from custom directory
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "CustomFiles")),
    RequestPath = "/files"
});

app.MapGet("/", () => "Static files are served from /wwwroot and /files");
app.Run();
```

### `UseDefaultFiles`
**Purpose:** Serves default documents like `index.html` when accessing a directory.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

// Configure default files
app.UseDefaultFiles(new DefaultFilesOptions
{
    DefaultFileNames = new List<string> { "index.html", "default.html", "home.html" }
});

app.UseStaticFiles(); // Must come after UseDefaultFiles

app.Run();
```

### `UseFileServer`
**Purpose:** Combines static files, default files, and directory browsing functionality.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

// Enable file server with directory browsing
app.UseFileServer(new FileServerOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "Files")),
    RequestPath = "/browse",
    EnableDirectoryBrowsing = true
});

app.MapGet("/", () => "Browse files at /browse");
app.Run();
```

### `UseWebSockets`
**Purpose:** Enables WebSocket support for real-time communication.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

// Configure WebSocket options
app.UseWebSockets(new WebSocketOptions
{
    KeepAliveInterval = TimeSpan.FromMinutes(2)
});

app.Use(async (context, next) =>
{
    if (context.Request.Path == "/ws")
    {
        if (context.WebSockets.IsWebSocketRequest)
        {
            var webSocket = await context.WebSockets.AcceptWebSocketAsync();
            await HandleWebSocket(webSocket);
        }
        else
        {
            context.Response.StatusCode = 400;
        }
    }
    else
    {
        await next();
    }
});

app.MapGet("/", () => "WebSocket endpoint available at /ws");
app.Run();

async Task HandleWebSocket(WebSocket webSocket)
{
    var buffer = new byte[1024 * 4];
    while (webSocket.State == WebSocketState.Open)
    {
        var result = await webSocket.ReceiveAsync(buffer, CancellationToken.None);
        if (result.MessageType == WebSocketMessageType.Text)
        {
            var message = Encoding.UTF8.GetString(buffer, 0, result.Count);
            var response = Encoding.UTF8.GetBytes($"Echo: {message}");
            await webSocket.SendAsync(response, WebSocketMessageType.Text, true, CancellationToken.None);
        }
    }
}
```

### `UseResponseCaching`
**Purpose:** Adds response caching capabilities to improve performance.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddResponseCaching(options =>
{
    options.MaximumBodySize = 1024 * 1024; // 1MB
    options.UseCaseSensitivePaths = true;
});

var app = builder.Build();

app.UseResponseCaching();

app.MapGet("/cached-data", () => 
{
    return Results.Ok(new { Data = "This response can be cached", Timestamp = DateTime.Now });
}).WithMetadata(new ResponseCacheAttribute { Duration = 300 }); // Cache for 5 minutes

app.Run();
```

### `UseResponseCompression`
**Purpose:** Compresses responses using algorithms like gzip and brotli to reduce bandwidth.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
    options.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(
        new[] { "application/json" });
});

var app = builder.Build();

app.UseResponseCompression(); // Compress responses

app.MapGet("/large-data", () => 
{
    var largeObject = new { 
        Data = string.Join("", Enumerable.Repeat("Large data content ", 1000))
    };
    return Results.Ok(largeObject);
});

app.Run();
```

### `UseSession`
**Purpose:** Enables session state management for storing user data server-side.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add session services
builder.Services.AddDistributedMemoryCache();
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(30);
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true;
    options.Cookie.Name = "MyApp.Session";
});

var app = builder.Build();

app.UseSession(); // Enable session middleware

app.MapGet("/set-session", (HttpContext context) =>
{
    context.Session.SetString("Username", "JohnDoe");
    context.Session.SetInt32("UserId", 123);
    return "Session data set";
});

app.MapGet("/get-session", (HttpContext context) =>
{
    var username = context.Session.GetString("Username");
    var userId = context.Session.GetInt32("UserId");
    return $"Username: {username}, UserId: {userId}";
});

app.Run();
```

---

## 🔹 Error Handling & Diagnostics

### `UseDeveloperExceptionPage`
**Purpose:** Shows detailed error pages with stack traces during development.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage(); // Show detailed errors in development
}

app.MapGet("/error", () => 
{
    throw new InvalidOperationException("This is a test exception");
});

app.Run();
```

### `UseExceptionHandler`
**Purpose:** Provides global exception handling middleware for production environments.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/error");
}

app.MapGet("/error", () => 
{
    return Results.Problem("An error occurred while processing your request.");
});

app.MapGet("/throw", () => 
{
    throw new Exception("Something went wrong!");
});

app.Run();
```

### `UseStatusCodePages`
**Purpose:** Returns custom responses for HTTP status codes like 404, 500.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

// Configure status code pages
app.UseStatusCodePages(async context =>
{
    var response = context.HttpContext.Response;
    var statusCode = response.StatusCode;
    
    response.ContentType = "text/plain";
    await response.WriteAsync($"Status Code: {statusCode} - {GetStatusDescription(statusCode)}");
});

// Or use status code pages with redirect
// app.UseStatusCodePagesWithRedirects("/error/{0}");

app.MapGet("/", () => "Home page");
// 404 will be handled by status code pages middleware

app.Run();

string GetStatusDescription(int statusCode) => statusCode switch
{
    404 => "Page not found",
    500 => "Internal server error",
    403 => "Forbidden",
    401 => "Unauthorized",
    _ => "Unknown status"
};
```

### `UseHealthChecks`
**Purpose:** Exposes health check endpoints to monitor application status.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add health checks
builder.Services.AddHealthChecks()
    .AddCheck("database", () => 
    {
        // Check database connection
        var isHealthy = true; // Your database check logic
        return isHealthy ? HealthCheckResult.Healthy("Database is running") 
                        : HealthCheckResult.Unhealthy("Database is down");
    })
    .AddCheck("external-api", async () =>
    {
        // Check external API
        using var client = new HttpClient();
        var response = await client.GetAsync("https://api.example.com/health");
        return response.IsSuccessStatusCode ? HealthCheckResult.Healthy("External API is running")
                                           : HealthCheckResult.Unhealthy("External API is down");
    });

var app = builder.Build();

// Configure health check endpoint
app.UseHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        context.Response.ContentType = "application/json";
        var response = new
        {
            Status = report.Status.ToString(),
            Checks = report.Entries.Select(x => new
            {
                Name = x.Key,
                Status = x.Value.Status.ToString(),
                Description = x.Value.Description,
                Duration = x.Value.Duration.ToString()
            }),
            Duration = report.TotalDuration.ToString()
        };
        await context.Response.WriteAsync(JsonSerializer.Serialize(response));
    }
});

app.MapGet("/", () => "Check health at /health");
app.Run();
```

---

## 🔹 Miscellaneous

### `UseMiddleware<T>`
**Purpose:** Custom middleware injection point for application-specific logic.

**Example:**
```csharp
// Custom middleware class
public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestTimingMiddleware> _logger;

    public RequestTimingMiddleware(RequestDelegate next, ILogger<RequestTimingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();
        
        await _next(context);
        
        stopwatch.Stop();
        _logger.LogInformation("Request {Method} {Path} took {ElapsedMs}ms", 
            context.Request.Method, 
            context.Request.Path, 
            stopwatch.ElapsedMilliseconds);
    }
}

var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

// Register custom middleware
app.UseMiddleware<RequestTimingMiddleware>();

app.MapGet("/", () => "Request timing will be logged");
app.Run();
```

### `UseForwardedHeaders`
**Purpose:** Handles proxy headers like `X-Forwarded-For` when behind a reverse proxy.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;
    options.KnownNetworks.Clear();
    options.KnownProxies.Clear();
});

var app = builder.Build();

app.UseForwardedHeaders(); // Handle forwarded headers from proxy

app.MapGet("/client-info", (HttpContext context) =>
{
    return new
    {
        RemoteIpAddress = context.Connection.RemoteIpAddress?.ToString(),
        Scheme = context.Request.Scheme,
        Host = context.Request.Host.ToString()
    };
});

app.Run();
```

### `UseRequestLocalization`
**Purpose:** Supports internationalization with culture and language settings.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.Configure<RequestLocalizationOptions>(options =>
{
    var supportedCultures = new[] { "en-US", "es-ES", "fr-FR", "de-DE" };
    options.SetDefaultCulture("en-US")
           .AddSupportedCultures(supportedCultures)
           .AddSupportedUICultures(supportedCultures);
    
    options.ApplyCurrentCultureToResponseHeaders = true;
});

var app = builder.Build();

app.UseRequestLocalization(); // Enable localization

app.MapGet("/culture", () =>
{
    var culture = CultureInfo.CurrentCulture;
    var uiCulture = CultureInfo.CurrentUICulture;
    
    return new
    {
        Culture = culture.Name,
        UICulture = uiCulture.Name,
        DateFormat = DateTime.Now.ToString("D", culture),
        NumberFormat = (12345.67).ToString("C", culture)
    };
});

app.Run();
```

### `UseSwagger` / `UseSwaggerUI`
**Purpose:** Generates and serves API documentation using OpenAPI specifications.

**Example:**
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo 
    { 
        Title = "My API", 
        Version = "v1",
        Description = "A sample API with Swagger documentation"
    });
    
    // Include XML comments if available
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    if (File.Exists(xmlPath))
    {
        c.IncludeXmlComments(xmlPath);
    }
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger(); // Generate OpenAPI specification
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
        c.RoutePrefix = "swagger"; // Access UI at /swagger
    });
}

app.MapControllers();

// Example API endpoint
app.MapGet("/api/products", () => 
{
    return new[] { 
        new { Id = 1, Name = "Product 1" }, 
        new { Id = 2, Name = "Product 2" } 
    };
})
.WithName("GetProducts")
.WithSummary("Get all products")
.WithDescription("Retrieves a list of all available products");

app.Run();
```

---

## Middleware Order Matters

The order of middleware registration is crucial. Here's the recommended order:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services...
builder.Services.AddAuthentication();
builder.Services.AddAuthorization();
builder.Services.AddControllers();

var app = builder.Build();

// 1. Exception handling (first)
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
    app.UseSwaggerUI();
}
else
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

// 2. HTTPS and security
app.UseHttpsRedirection();
app.UseForwardedHeaders();

// 3. Static files (early for performance)
app.UseStaticFiles();

// 4. Routing
app.UseRouting();

// 5. CORS (after routing, before auth)
app.UseCors();

// 6. Authentication & Authorization
app.UseAuthentication();
app.UseAuthorization();

// 7. Custom middleware
app.UseMiddleware<RequestTimingMiddleware>();

// 8. Session (late in pipeline)
app.UseSession();

// 9. Endpoints (last)
app.MapControllers();

app.Run();
```

This guide provides a comprehensive overview of ASP.NET Core middleware with practical examples for each component. Each middleware serves a specific purpose in the request pipeline and should be ordered correctly for optimal functionality.
