# Advanced Features of Web API with ASP.NET Core in .NET 6

Web APIs form the backbone of numerous web and mobile applications and internet-enabled devices, from refrigerators to smartphones. ASP.NET Core provides a robust framework for crafting powerful APIs.

## Table of Contents
1. [Advanced Data Retrieval](#advanced-data-retrieval)
2. [API Versioning](#api-versioning)
3. [Security Measures](#security-measures)
4. [Prerequisites](#prerequisites)
5. [Sample Application](#sample-application)

## Advanced Data Retrieval <a name="advanced-data-retrieval"></a>

This section will focus on enhancing your data retrieval techniques by adding features like pagination and filtering. These functionalities are key in managing large amounts of data efficiently.

## API Versioning <a name="api-versioning"></a>

API versioning ensures backward compatibility and prevents breaking changes from affecting existing clients. We will implement several approaches to versioning APIs, with the help of NuGet packages that can save us significant time.

## Security Measures <a name="security-measures"></a>

This section will delve into various security-related topics, such as enforcing HTTPS, enabling cross-domain JavaScript requests to our API (CORS), and securing the API with tokens.

## Prerequisites <a name="prerequisites"></a>

The prerequisites for this course include:

- Some .NET experience, particularly with .NET 6 and ASP.NET Core 6.
- Familiarity with C#, the primary language for this guide.
- Basic experience with Web APIs, such as creating projects based on ASP.NET Core Web API, implementing API endpoints that support different HTTP methods, and handling HTTP status codes.

## Sample Application <a name="sample-application"></a>

Our sample API revolves around a simple model: an online shop a the fictional company. It includes products, each with properties like name, description, and price, categorized under different categories.

The data storage is handled by Entity Framework using an in-memory database. This means no extra requirements for databases like SQL Server. However, keep in mind that the data is regenerated each time we restart the application.

Here's the basic structure of our `Product` class:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public Category Category { get; set; }
}
```

We also have a `ProductsController` that provides full CRUD (Create, Read, Update, Delete) functionality, including getting all products, getting a specific product, and deleting multiple products.

```csharp
[HttpGet]
public async Task<ActionResult<IEnumerable<Product>>> GetProducts()

[HttpGet("{id}")]
public async Task<ActionResult<Product>> GetProduct(int id)

[HttpPost]
public async Task<ActionResult<Product>> PostProduct(Product product)

[HttpPut("{id}")]
public async Task<IActionResult> PutProduct(int id, Product product)

[HttpDelete("{id}")]
public async Task<IActionResult> DeleteProduct(int id)

[HttpDelete]
public async Task<IActionResult> DeleteProducts(IEnumerable<int> ids)
```

Upon running the application, you can access the Swagger user interface for the API in your web browser via your specified server (for example, localhost:7233/swagger/index.html).
