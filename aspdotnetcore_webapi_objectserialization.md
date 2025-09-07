# Serialization in .NET - Complete Guide with ASP.NET Web API Examples

## Introduction

Serialization in .NET is a crucial process for converting complex objects into a format that can be stored, transmitted, and reconstructed effectively. This guide provides practical examples of implementing different serialization techniques in ASP.NET Web API applications.

## Table of Contents

1. [JSON Serialization](#json-serialization)
2. [XML Serialization](#xml-serialization)
3. [Binary Serialization](#binary-serialization)
4. [Custom Serialization Settings](#custom-serialization-settings)
5. [Performance Considerations](#performance-considerations)

---

## JSON Serialization

JSON serialization is the most common approach in modern web APIs due to its lightweight nature and excellent JavaScript compatibility.

### Sample Model

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public DateTime CreatedDate { get; set; }
    public List<string> Categories { get; set; } = new();
}
```

### 1. Automatic JSON Serialization (Default ASP.NET Core)

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    public ActionResult<Product> GetProduct(int id)
    {
        var product = new Product
        {
            Id = id,
            Name = "Sample Product",
            Price = 29.99m,
            CreatedDate = DateTime.UtcNow,
            Categories = new List<string> { "Electronics", "Gadgets" }
        };

        // ASP.NET Core automatically serializes to JSON
        return Ok(product);
    }

    [HttpPost]
    public ActionResult<Product> CreateProduct([FromBody] Product product)
    {
        // ASP.NET Core automatically deserializes from JSON
        product.Id = new Random().Next(1, 1000);
        product.CreatedDate = DateTime.UtcNow;
        
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }
}
```

### 2. Manual JSON Serialization

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsManualController : ControllerBase
{
    [HttpGet("manual/{id}")]
    public async Task<IActionResult> GetProductManual(int id)
    {
        var product = new Product
        {
            Id = id,
            Name = "Manual Serialization Product",
            Price = 49.99m,
            CreatedDate = DateTime.UtcNow,
            Categories = new List<string> { "Books", "Education" }
        };

        // Manual JSON serialization with custom options
        var options = new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
            WriteIndented = true,
            PropertyNameCaseInsensitive = true
        };

        var json = JsonSerializer.Serialize(product, options);
        
        return Ok(json);
    }

    [HttpPost("manual")]
    public async Task<IActionResult> CreateProductManual()
    {
        using var reader = new StreamReader(Request.Body);
        var json = await reader.ReadToEndAsync();

        var options = new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
            PropertyNameCaseInsensitive = true
        };

        var product = JsonSerializer.Deserialize<Product>(json, options);
        product.Id = new Random().Next(1, 1000);
        product.CreatedDate = DateTime.UtcNow;

        return CreatedAtAction(nameof(GetProductManual), new { id = product.Id }, product);
    }
}
```

---

## XML Serialization

XML serialization is useful for systems requiring structured data exchange and interoperability with legacy systems.

### Model with XML Attributes

```csharp
[XmlRoot("Product")]
public class ProductXml
{
    [XmlElement("ProductId")]
    public int Id { get; set; }

    [XmlElement("ProductName")]
    public string Name { get; set; }

    [XmlElement("ProductPrice")]
    public decimal Price { get; set; }

    [XmlElement("CreationDate")]
    public DateTime CreatedDate { get; set; }

    [XmlArray("CategoryList")]
    [XmlArrayItem("Category")]
    public List<string> Categories { get; set; } = new();
}
```

### XML Serialization Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsXmlController : ControllerBase
{
    [HttpGet("{id}")]
    [Produces("application/xml")]
    public IActionResult GetProductXml(int id)
    {
        var product = new ProductXml
        {
            Id = id,
            Name = "XML Serialized Product",
            Price = 39.99m,
            CreatedDate = DateTime.UtcNow,
            Categories = new List<string> { "Home", "Kitchen" }
        };

        return Ok(product);
    }

    [HttpGet("manual/{id}")]
    public IActionResult GetProductXmlManual(int id)
    {
        var product = new ProductXml
        {
            Id = id,
            Name = "Manual XML Product",
            Price = 59.99m,
            CreatedDate = DateTime.UtcNow,
            Categories = new List<string> { "Sports", "Outdoor" }
        };

        // Manual XML serialization
        var serializer = new XmlSerializer(typeof(ProductXml));
        using var stringWriter = new StringWriter();
        serializer.Serialize(stringWriter, product);
        var xml = stringWriter.ToString();

        return Content(xml, "application/xml");
    }

    [HttpPost]
    [Consumes("application/xml")]
    public IActionResult CreateProductXml([FromBody] ProductXml product)
    {
        product.Id = new Random().Next(1, 1000);
        product.CreatedDate = DateTime.UtcNow;

        return CreatedAtAction(nameof(GetProductXml), new { id = product.Id }, product);
    }

    [HttpPost("manual")]
    public async Task<IActionResult> CreateProductXmlManual()
    {
        using var reader = new StreamReader(Request.Body);
        var xml = await reader.ReadToEndAsync();

        // Manual XML deserialization
        var serializer = new XmlSerializer(typeof(ProductXml));
        using var stringReader = new StringReader(xml);
        var product = (ProductXml)serializer.Deserialize(stringReader);
        
        product.Id = new Random().Next(1, 1000);
        product.CreatedDate = DateTime.UtcNow;

        return CreatedAtAction(nameof(GetProductXml), new { id = product.Id }, product);
    }
}
```

---

## Binary Serialization

Binary serialization provides compact storage and is ideal for performance-critical scenarios or internal system communication.

### Binary Serialization Helper

```csharp
public static class BinarySerializationHelper
{
    public static byte[] Serialize<T>(T obj)
    {
        using var stream = new MemoryStream();
        using var writer = new BinaryWriter(stream);
        
        // Simple binary serialization for demonstration
        // In production, consider using more robust solutions like MessagePack or protobuf-net
        var json = JsonSerializer.Serialize(obj);
        var bytes = Encoding.UTF8.GetBytes(json);
        writer.Write(bytes.Length);
        writer.Write(bytes);
        
        return stream.ToArray();
    }

    public static T Deserialize<T>(byte[] data)
    {
        using var stream = new MemoryStream(data);
        using var reader = new BinaryReader(stream);
        
        var length = reader.ReadInt32();
        var bytes = reader.ReadBytes(length);
        var json = Encoding.UTF8.GetString(bytes);
        
        return JsonSerializer.Deserialize<T>(json);
    }
}
```

### Binary Serialization Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsBinaryController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetProductBinary(int id)
    {
        var product = new Product
        {
            Id = id,
            Name = "Binary Serialized Product",
            Price = 79.99m,
            CreatedDate = DateTime.UtcNow,
            Categories = new List<string> { "Technology", "Software" }
        };

        var binaryData = BinarySerializationHelper.Serialize(product);
        
        return File(binaryData, "application/octet-stream", $"product_{id}.bin");
    }

    [HttpPost]
    public async Task<IActionResult> CreateProductBinary(IFormFile file)
    {
        if (file == null || file.Length == 0)
            return BadRequest("No file uploaded");

        using var stream = new MemoryStream();
        await file.CopyToAsync(stream);
        var binaryData = stream.ToArray();

        var product = BinarySerializationHelper.Deserialize<Product>(binaryData);
        product.Id = new Random().Next(1, 1000);
        product.CreatedDate = DateTime.UtcNow;

        return Ok(product);
    }

    [HttpGet("cache/{id}")]
    public IActionResult GetProductFromCache(int id)
    {
        // Example: Using binary serialization for caching
        var product = new Product
        {
            Id = id,
            Name = "Cached Product",
            Price = 99.99m,
            CreatedDate = DateTime.UtcNow,
            Categories = new List<string> { "Cache", "Performance" }
        };

        var binaryData = BinarySerializationHelper.Serialize(product);
        
        // In real scenario, you'd store/retrieve from cache (Redis, Memory, etc.)
        var cachedProduct = BinarySerializationHelper.Deserialize<Product>(binaryData);

        return Ok(new { 
            Message = "Retrieved from binary cache",
            Product = cachedProduct,
            BinarySizeBytes = binaryData.Length
        });
    }
}
```

---

## Custom Serialization Settings

### Global Configuration in Program.cs

```csharp
using System.Text.Json;
using System.Text.Json.Serialization;

var builder = WebApplication.CreateBuilder(args);

// Configure JSON serialization globally
builder.Services.ConfigureHttpJsonOptions(options =>
{
    options.SerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
    options.SerializerOptions.WriteIndented = true;
    options.SerializerOptions.PropertyNameCaseInsensitive = true;
    options.SerializerOptions.Converters.Add(new JsonStringEnumConverter());
    options.SerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull;
});

// Configure MVC JSON options
builder.Services.Configure<JsonOptions>(options =>
{
    options.SerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
    options.SerializerOptions.PropertyNameCaseInsensitive = true;
    options.SerializerOptions.Converters.Add(new JsonStringEnumConverter());
});

// Add XML formatters
builder.Services.AddControllers(options =>
{
    options.RespectBrowserAcceptHeader = true;
    options.ReturnHttpNotAcceptable = true;
}).AddXmlSerializerFormatters();

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

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

### Custom JSON Converter Example

```csharp
public class CustomDateTimeConverter : JsonConverter<DateTime>
{
    private readonly string _dateFormat = "yyyy-MM-dd HH:mm:ss";

    public override DateTime Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        return DateTime.ParseExact(reader.GetString(), _dateFormat, CultureInfo.InvariantCulture);
    }

    public override void Write(Utf8JsonWriter writer, DateTime value, JsonSerializerOptions options)
    {
        writer.WriteStringValue(value.ToString(_dateFormat));
    }
}

// Usage in model
public class ProductWithCustomDateTime
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    [JsonConverter(typeof(CustomDateTimeConverter))]
    public DateTime CreatedDate { get; set; }
}
```

### Content Negotiation Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsNegotiationController : ControllerBase
{
    [HttpGet("{id}")]
    [Produces("application/json", "application/xml")]
    public ActionResult<Product> GetProduct(int id, [FromHeader(Name = "Accept")] string acceptHeader)
    {
        var product = new Product
        {
            Id = id,
            Name = "Content Negotiated Product",
            Price = 89.99m,
            CreatedDate = DateTime.UtcNow,
            Categories = new List<string> { "Flexible", "Negotiation" }
        };

        // The framework automatically handles content negotiation based on Accept header
        return Ok(product);
    }

    [HttpGet("format/{id}")]
    public IActionResult GetProductInFormat(int id, [FromQuery] string format = "json")
    {
        var product = new Product
        {
            Id = id,
            Name = "Format Specific Product",
            Price = 69.99m,
            CreatedDate = DateTime.UtcNow,
            Categories = new List<string> { "Format", "Specific" }
        };

        return format.ToLower() switch
        {
            "xml" => Ok(product), // Will be serialized as XML if XML formatters are configured
            "binary" => File(BinarySerializationHelper.Serialize(product), "application/octet-stream"),
            _ => Ok(product) // Default to JSON
        };
    }
}
```

---

## Performance Considerations

### Performance Comparison Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class PerformanceController : ControllerBase
{
    [HttpGet("compare/{iterations}")]
    public IActionResult CompareSerializationPerformance(int iterations = 1000)
    {
        var products = GenerateTestProducts(100);
        var results = new Dictionary<string, object>();

        // JSON Serialization Performance
        var jsonStopwatch = Stopwatch.StartNew();
        for (int i = 0; i < iterations; i++)
        {
            var json = JsonSerializer.Serialize(products);
            var deserialized = JsonSerializer.Deserialize<List<Product>>(json);
        }
        jsonStopwatch.Stop();

        // XML Serialization Performance
        var xmlStopwatch = Stopwatch.StartNew();
        var xmlSerializer = new XmlSerializer(typeof(List<Product>));
        for (int i = 0; i < iterations; i++)
        {
            using var stringWriter = new StringWriter();
            xmlSerializer.Serialize(stringWriter, products);
            var xml = stringWriter.ToString();
            
            using var stringReader = new StringReader(xml);
            var deserialized = (List<Product>)xmlSerializer.Deserialize(stringReader);
        }
        xmlStopwatch.Stop();

        // Binary Serialization Performance
        var binaryStopwatch = Stopwatch.StartNew();
        for (int i = 0; i < iterations; i++)
        {
            var binary = BinarySerializationHelper.Serialize(products);
            var deserialized = BinarySerializationHelper.Deserialize<List<Product>>(binary);
        }
        binaryStopwatch.Stop();

        results.Add("JSON", new { 
            ElapsedMs = jsonStopwatch.ElapsedMilliseconds,
            SizeBytes = Encoding.UTF8.GetBytes(JsonSerializer.Serialize(products)).Length
        });
        
        results.Add("XML", new { 
            ElapsedMs = xmlStopwatch.ElapsedMilliseconds,
            SizeBytes = GetXmlSize(products)
        });
        
        results.Add("Binary", new { 
            ElapsedMs = binaryStopwatch.ElapsedMilliseconds,
            SizeBytes = BinarySerializationHelper.Serialize(products).Length
        });

        return Ok(new { 
            Iterations = iterations,
            ProductCount = products.Count,
            Results = results
        });
    }

    private List<Product> GenerateTestProducts(int count)
    {
        return Enumerable.Range(1, count)
            .Select(i => new Product
            {
                Id = i,
                Name = $"Product {i}",
                Price = new Random().Next(1, 1000),
                CreatedDate = DateTime.UtcNow.AddDays(-new Random().Next(1, 365)),
                Categories = new List<string> { $"Category{i % 5}", $"SubCategory{i % 3}" }
            })
            .ToList();
    }

    private int GetXmlSize(List<Product> products)
    {
        var serializer = new XmlSerializer(typeof(List<Product>));
        using var stringWriter = new StringWriter();
        serializer.Serialize(stringWriter, products);
        return Encoding.UTF8.GetBytes(stringWriter.ToString()).Length;
    }
}
```

---

## Conclusion

This guide demonstrates practical implementation of different serialization techniques in ASP.NET Web API:

- **JSON Serialization**: Best for web APIs and modern applications
- **XML Serialization**: Ideal for enterprise systems and interoperability
- **Binary Serialization**: Optimal for performance-critical internal communications

Choose the appropriate serialization method based on your specific requirements for performance, interoperability, and data format preferences.

### Key Takeaways

1. JSON is the default and most widely used format in modern web APIs
2. XML provides better interoperability with legacy systems
3. Binary serialization offers the best performance for internal communications
4. Content negotiation allows clients to choose their preferred format
5. Custom serialization settings provide fine-grained control over data formatting

### Best Practices

- Use automatic serialization when possible to reduce code complexity
- Configure global serialization settings for consistency
- Implement proper error handling for serialization/deserialization operations
- Consider performance implications when choosing serialization methods
- Use appropriate HTTP content types for different serialization formats
