# Understanding and Implementing API Versioning in C# and .NET Core

In software development, it's inevitable that your APIs will evolve over time. As new features get added and old ones get deprecated, there may be changes that are not backwards compatible. That's where API versioning comes in. In this guide, we'll take a closer look at different approaches to API versioning in ASP.NET Core and explore a helpful library from Microsoft that makes the implementation process more efficient.

## Table of Contents

- [Versioning Options](#versioning-options)
- [Using the Microsoft Versioning Library](#using-the-microsoft-versioning-library)
- [Implementing URL Versioning](#implementing-url-versioning)
- [Implementing Header Versioning](#implementing-header-versioning)
- [Implementing Query String Versioning](#implementing-query-string-versioning)
- [Fixing the API Documentation with Swagger](#fixing-api-documentation-with-swagger)

## Versioning Options <a name="versioning-options"></a>

There are several ways to implement API versioning, but three of the most common methods include:

1. **Using a specific HTTP header**: In this approach, the API version is specified within an HTTP header. We can use a custom header such as `X-API-Version`, though the name is flexible and can be something different. It's common practice to prefix custom headers with 'X' to denote that they're non-standard.

    ```http
    GET /products HTTP/1.1
    Host: api.example.com
    X-API-Version: 1.0
    ```

2. **Including the version in the URL path**: In this method, the version number is added directly to the URL, often as a prefix to the path. While this is a commonly used approach, it does go against the principle of having a single URI for a specific set of data, as set out in the REST architectural style.

    ```http
    GET /v1.0/products HTTP/1.1
    Host: api.example.com
    ```

3. **Appending the version as a query string value**: Although less common, this approach involves adding the version number as a query string parameter in the URL. This method requires caution to avoid clashes with existing query string parameters.

    ```http
    GET /products?api-version=1.0 HTTP/1.1
    Host: api.example.com
    ```

## Using the Microsoft Versioning Library <a name="using-the-microsoft-versioning-library"></a>

Rather than writing our own code to handle API versioning, we can take advantage of an existing library that does most of the heavy lifting for us. The `Microsoft.AspNetCore.Mvc.Versioning` NuGet package is designed to help with API versioning.

To add this package to your project, follow the steps below:

1. Open the NuGet Package Manager in Visual Studio and search for "versioning".
2. From the search results, select `Microsoft.AspNetCore.Mvc.Versioning`. As of this writing, the latest version is `5.0.0`. Even though it was designed for .NET 5, it also works with .NET 6, with a few minor exceptions. 
3. Click 'Install' to add it to your project.

If you're using Visual Studio Code or you prefer using the command line, you can also use the .NET CLI to add the package:

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Versioning
```

Be sure to run this command in the directory where your project file (`.csproj`) is located. 

In the following sections, we will explore how to implement different versioning strategies using this package.

## Implementing URL Versioning <a name="implementing-url-versioning"></a>

To implement URL versioning, we will utilize the `ApiVersion` attribute from the `Microsoft.AspNetCore.Mvc.Versioning` library in our products controller. We will also need to make some necessary configurations in the `Program.cs` file.

### Step 1: Add ApiVersion Attribute

First, we will modify the `ProductsController` class by adding the `ApiVersion` attribute to it. This attribute will specify the version of the API that this controller should respond to.

```csharp
[ApiController]
[ApiVersion("1.0")]
[Route("v{version:apiVersion}/products")]
public class ProductsV1Controller : ControllerBase
{
    // controller methods go here
}
```
The `ApiVersion` attribute is used to specify the version that this controller responds to, in this case, version `1.0`. The `Route` attribute is set up such that it prefixes the path with the version number.

### Step 2: Configure Versioning in Program.cs

Before we can utilize the versioning library, we have to configure it in the `Program.cs` file right after the `AddControllers` call:

```csharp
var builder = WebApplication.CreateBuilder(args);
// other builder configurations

builder.Services.AddControllers();
builder.Services.AddApiVersioning(options => 
{
    options.ReportApiVersions = true;
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
});
```
In the configuration options:

* `ReportApiVersions` is set to `true` so that the HTTP response will contain the version number of the API. This can be useful for backward compatibility.
* `DefaultApiVersion` sets the default version to `1.0` when no specific API version is specified in the HTTP request.
* `AssumeDefaultVersionWhenUnspecified` is set to `true`, indicating that the default version should be used when no version is specified in the request.

### Step 3: Implementing a Second Version

We can create another controller for version `2.0` of the API. In this example, we will make a simple change to simulate version evolution: version `2.0` will only return available products.

```csharp
[ApiController]
[ApiVersion("2.0")]
[Route("v{version:apiVersion}/products")]
public class ProductsV2Controller : ControllerBase
{
    // controller methods go here
}
```

In the method responsible for returning the products in `ProductsV2Controller`, we can modify the logic so that it only returns products that are available:

```csharp
public IEnumerable<Product> GetAllProducts()
{
    return _productService.GetAllProducts().Where(p => p.IsAvailable);
}
```
Now, when we make requests to our API using Postman or another HTTP client, we can specify the version in the URL:

```http
GET /v1.0/products HTTP/1.1
Host: api.example.com
```
This will return all products, whereas:

```http
GET /v2.0/products HTTP/1.1
Host: api.example.com
```
This will return only the products that are available, demonstrating that the correct controller and version are being used.

## Implementing Header Versioning <a name="implementing-header-versioning"></a>

While URL versioning is a viable approach, sometimes it may be preferred to use HTTP headers for specifying the API version. The `Microsoft.AspNetCore.Mvc.Versioning` library we installed earlier also supports this approach with some additional configurations.

### Step 1: Adjust the Controller

Start by adjusting the controllers to remove the version from the `Route` attribute since the version will be specified in the HTTP header instead. You still need to specify the `ApiVersion` attribute to determine the version that the controller corresponds to.

```csharp
[ApiController]
[ApiVersion("1.0")]
[Route("products")]
public class ProductsV1Controller : ControllerBase
{
    // controller methods go here
}
```
Repeat the same process for `ProductsV2Controller`:

```csharp
[ApiController]
[ApiVersion("2.0")]
[Route("products")]
public class ProductsV2Controller : ControllerBase
{
    // controller methods go here
}
```
These controllers are set to respond to versions `1.0` and `2.0` respectively.

### Step 2: Configure the Versioning in Program.cs

After adjusting the controllers, you need to configure the `Microsoft.AspNetCore.Mvc.Versioning` library to use header versioning.

```csharp
var builder = WebApplication.CreateBuilder(args);
// other builder configurations

builder.Services.AddControllers();
builder.Services.AddApiVersioning(options => 
{
    options.ReportApiVersions = true;
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ApiVersionReader = new HeaderApiVersionReader("X-API-Version");
});
```
In this configuration, the `HeaderApiVersionReader` is instantiated with the name of the HTTP header that will carry the version information, in this case, "X-API-Version".

### Step 3: Testing the Configuration

Now, when making a request to the API, you can specify the version by adding an `X-API-Version` header with the desired version number as the value. For example:

```http
GET /products HTTP/1.1
Host: api.example.com
X-API-Version: 1.0
```
This request will be handled by the `ProductsV1Controller`, while:

```http
GET /products HTTP/1.1
Host: api.example.com
X-API-Version: 2.0
```
This request will be handled by the `ProductsV2Controller`. In this way, you can control the version of the API to be used by specifying it in the HTTP header of the request.

## Implementing Query String Versioning <a name="implementing-query-string-versioning"></a>

The final versioning approach we'll discuss involves using query strings to specify the API version. This approach is rather simple and works out of the box with our previously installed `Microsoft.AspNetCore.Mvc.Versioning` library.

### Step 1: Adjust the Program.cs

In your `Program.cs` file, you only need to modify the configuration of the `AddApiVersioning` method. Specifically, you should remove the `HeaderApiVersionReader` and the framework will default to query string versioning.

```csharp
var builder = WebApplication.CreateBuilder(args);
// other builder configurations

builder.Services.AddControllers();
builder.Services.AddApiVersioning(options => 
{
    options.ReportApiVersions = true;
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
});
```

### Step 2: Testing the Configuration

Now, to specify the API version in a request, you add a `api-version` query parameter to the URL with the desired version number as the value. For example:

```http
GET /products?api-version=1.0 HTTP/1.1
Host: api.example.com
```

This request will be handled by the `ProductsV1Controller`. On the other hand:

```http
GET /products?api-version=2.0 HTTP/1.1
Host: api.example.com
```

This request will be handled by the `ProductsV2Controller`.

### Step 3: Customizing the Query Parameter

If you'd like to customize the name of the query parameter, you can do that by adding a `QueryStringApiVersionReader` to your `AddApiVersioning` configuration:

```csharp
builder.Services.AddApiVersioning(options => 
{
    options.ReportApiVersions = true;
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ApiVersionReader = new QueryStringApiVersionReader("hps-api-version");
});
```

Now the API version can be specified by using the `hps-api-version` query parameter in the URL:

```http
GET /products?hps-api-version=2.0 HTTP/1.1
Host: api.example.com
```

This request will be handled by the `ProductsV2Controller`. 

In summary, the query string versioning approach gives you flexibility and simplicity in controlling the version of the API that your application uses.

## Fixing the API Documentation with Swagger <a name="fixing-api-documentation-with-swagger"></a>

If you're using Swagger UI to document your API, you might encounter issues when working with multiple API versions. The root of this problem lies in the fact that Swagger doesn't automatically differentiate between different versions of the same route, leading to confusion and potential errors.

To fix this, we need to install the `Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer` NuGet package, which helps Swagger distinguish between different API versions.

### Step 1: Install the Required NuGet Package

In your NuGet package manager, search for `Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer`. Install this package into your project. If you're using .NET CLI, you can use the command:

```shell
dotnet add package Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer
```

### Step 2: Update the Configuration

Once the package is installed, go to your `Program.cs` file and configure `AddVersionedApiExplorer`. This method provides options specific to how Swagger interacts with API versions.

In `Program.cs`, add this to your `builder.Services` configurations:

```csharp
builder.Services.AddVersionedApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});
```

This configuration allows Swagger to correctly group different versions of the API and display them properly in the Swagger UI.

### Step 3: Testing the Fix

Now, if you navigate to your Swagger UI, you should be able to see your different API versions listed under their respective controller names, like `ProductsV`. You can try out your endpoints by providing parameters, including the version header. The Swagger UI should be able to correctly execute and display responses for each API version.

Please note that this fix is a workaround and not a comprehensive solution for versioning with Swagger. Some versions might not appear automatically in the UI. However, this fix allows the Swagger UI to work without error messages for the first version of your API. Over time, this issue is likely to be addressed by updates to the `Microsoft.AspNetCore.Mvc.Versioning` NuGet package.
