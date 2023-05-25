# Advanced Data Retrieval in ASP.NET Core

In many cases, APIs are required to handle the retrieval of large volumes of data. When dealing with a large number of products, returning all products at once might not be efficient or feasible. This tutorial walks you through the process of implementing advanced data retrieval mechanisms in a .NET Core API using C#. We will focus specifically on pagination, filtering and sorting. 

## Table of Contents
- [Implementing Pagination](#implementing-pagination)
  - [Adding QueryParameters Model](#adding-queryparameters-model)
  - [Updating the GetAllProducts Method](#updating-the-getallproducts-method)
  - [Testing the Pagination](#testing-the-pagination)
- [Implementing Filtering](#implementing-filtering)
  - [Adding ProductQueryParameters Model](#adding-productqueryparameters-model)
  - [Updating the GetAllProducts Method for Filtering](#updating-the-getallproducts-method-for-filtering)
  - [Testing the Filtering](#testing-the-filtering)
- [Implementing Searching](#implementing-searching)
  - [Updating the ProductQueryParameters Model for Searching](#updating-the-productqueryparameters-model-for-searching)
  - [Updating the GetAllProducts Method for Searching](#updating-the-getallproducts-method-for-searching)
  - [Testing the Searching](#testing-the-searching)
- [Adding Advanced Searching](#adding-advanced-searching)
  - [Updating the ProductQueryParameters Model for Advanced Searching](#updating-the-productqueryparameters-model-for-advanced-searching)
  - [Updating the GetAllProducts Method for Advanced Searching](#updating-the-getallproducts-method-for-advanced-searching)
  - [Testing the Advanced Searching](#testing-the-advanced-searching)
- [Implementing Sorting](#implementing-sorting)
  - [Updating the QueryParameters Model for Sorting](#updating-the-queryparameters-model-for-sorting)
  - [Adding IQueryableExtensions Class for Sorting](#adding-iqueryableextensions-class-for-sorting)
  - [Updating the GetAllProducts Method for Sorting](#updating-the-getallproducts-method-for-sorting)
  - [Testing the Sorting](#testing-the-sorting)

## Implementing Pagination <a name="implementing-pagination"></a>

In this section, we will update the API to accept two query string parameters: `size` and `page`. The `size` parameter determines the number of items on each page, and the `page` parameter identifies the specific page to return. 

This requires a certain mathematical understanding. If we consider an arbitrary page number and start at page one, this translates to: if we're on page five, we skip the first four pages; on page two, we skip one page. In code, this means we skip `(page - 1) * size` items. This will bring us to the start of the selected page.

We only return items that fit on one page, and the `Take` method will handle cases where there are fewer items left than the page size.

### Adding QueryParameters Model <a name="adding-queryparameters-model"></a>

Let's start by adding a model class for the parameters. This approach gives us the flexibility to add more parameters later, as we continue to enhance our API.

```csharp
public class QueryParameters
{
    private const int _maxSize = 100;
    private int _size = 50;

    public int Page { get; set; } = 1;

    public int Size
    {
        get
        {
            return _size;
        }
        set
        {
            _size = (value > _maxSize) ? _maxSize : value;
        }
    }
}
```
This model has two public properties, `Page` and `Size`, which correspond to our two query parameters. The `Page` property is initialized to one, and the `Size` property is limited to a maximum size of 100 to prevent returning an excessively large number of items. 

### Updating the GetAllProducts Method <a name="updating-the-getallproducts-method"></a>

Next, we update the `GetAllProducts` method in our products controller. Previously, this method returned all products. Now, we need to modify it to return only a subset of products based on the `QueryParameters` provided in the API call. 

```csharp
[HttpGet]
public async Task<IActionResult> GetAllProducts([FromQuery] QueryParameters queryParameters)
{
    IQueryable<Product> products = _context.Products;

    products = products
        .Skip(queryParameters.Size * (queryParameters.Page - 1))
        .Take(queryParameters.Size);

    return Ok(await products.ToListAsync());
}
```
Here, we use the `Skip` and `Take` methods to implement pagination. We calculate the number of items to skip based on the page number and size, and then take the number of items equivalent to the page size.

### Testing the Pagination <a name="testing-the-pagination"></a>

Now it's time to test our API. Let's assume that our products API endpoint is `/Products`.

When we access the `/Products` endpoint without any parameters, it returns products with a default page size of 50, starting from the first page.

However, if we set `size=25` (e.g., `/Products?size=25`), the last product in our list will have an ID of 25. This shows that we've set the page size successfully. 

If we are interested in the second page, we can set `page=2` (e.g., `/Products?size=25&page=2`). This will start at the 26th item, but end at the 33rd item since we only have 8 items left. 

Thus, our API now supports pagination, with a default page number of one and a page size of 50. Both of these values can be changed by setting the appropriate query string parameters.

## Implementing Filtering <a name="implementing-filtering"></a>

Filtering refers to the application of rules to limit the number of items returned. These rules can be based on a variety of factors, including price intervals, product categories, or other product-specific attributes.

### Adding ProductQueryParameters Model <a name="adding-productqueryparameters-model"></a>

We'll begin by adding a new class to handle our product-specific query parameters. This class will inherit from our existing `QueryParameters` class, so it will still include the `Page` and `Size` properties. We will then add properties to filter by `MinPrice` and `MaxPrice`.

```csharp
public class ProductQueryParameters : QueryParameters
{
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
}
```

This class includes two new properties, `MinPrice` and `MaxPrice`, which are both nullable decimals. This design allows the client to optionally set either or both of these values when making a request to the API.

### Updating the GetAllProducts Method for Filtering <a name="updating-the-getallproducts-method-for-filtering"></a>

Next, we will update the `GetAllProducts` method in the products controller to apply the filter parameters. This involves adding a `WHERE` clause to our LINQ query for each filter parameter that is set.

```csharp
[HttpGet]
public async Task<IActionResult> GetAllProducts([FromQuery] ProductQueryParameters queryParameters)
{
    IQueryable<Product> products = _context.Products;

    if (queryParameters.MinPrice != null)
    {
        products = products.Where(
            p => p.Price >= queryParameters.MinPrice.Value);
    }

    if (queryParameters.MaxPrice != null)
    {
        products = products.Where(
            p => p.Price <= queryParameters.MaxPrice.Value);
    }

    products = products
        .Skip(queryParameters.Size * (queryParameters.Page - 1))
        .Take(queryParameters.Size);

    return Ok(await products.ToListAsync());
}
```

In this updated method, we first check if the `MinPrice` and `MaxPrice` query parameters are set, and if so, apply the appropriate filter. We then apply the pagination as before.

### Testing the Filtering <a name="testing-the-filtering"></a>

Finally, we can test our API to ensure that the filtering is working as expected. If we access the `/Products` endpoint with the query parameters `minPrice=20` and `maxPrice=50` (i.e., `/Products?minPrice=20&maxPrice=50`), the API should return all products priced between $20 and $50, inclusive. 

By allowing both pagination and filtering, our API has become much more flexible and efficient. Future additions to the filtering capabilities could include other product-specific parameters, following a similar pattern to what we've implemented here.

## Implementing Searching <a name="implementing-searching"></a>

Searching allows users to find specific items based on their properties. In our case, we will implement a simple search function that allows users to search for products by name or SKU.

## Updating the ProductQueryParameters Model for Searching <a name="updating-the-productqueryparameters-model-for-searching"></a>

First, we need to amend the `ProductQueryParameters` model to accommodate the search parameters. We will add two properties: `Name` and `Sku`. 

```csharp
public class ProductQueryParameters : QueryParameters
{
    public string Name { get; set; } = string.Empty;
    public string Sku { get; set; } = string.Empty;
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
}
```

The `Name` property will be used to perform a substring match on the product name, while the `Sku` property will be used to perform an exact match on the product SKU.

### Updating the GetAllProducts Method for Searching <a name="updating-the-getallproducts-method-for-searching"></a>

Next, we need to update the `GetAllProducts` method to use the new search parameters.

```csharp
[HttpGet]
public async Task<IActionResult> GetAllProducts([FromQuery] ProductQueryParameters queryParameters)
{
    IQueryable<Product> products = _context.Products;

    if (!string.IsNullOrEmpty(queryParameters.Sku))
    {
        products = products.Where(p => p.Sku == queryParameters.Sku);
    }

    if (!string.IsNullOrEmpty(queryParameters.Name))
    {
        products = products.Where(p => p.Name.ToLower().Contains(queryParameters.Name.ToLower()));
    }

    if (queryParameters.MinPrice != null)
    {
        products = products.Where(p => p.Price >= queryParameters.MinPrice.Value);
    }

    if (queryParameters.MaxPrice != null)
    {
        products = products.Where(p => p.Price <= queryParameters.MaxPrice.Value);
    }

    products = products
        .Skip(queryParameters.Size * (queryParameters.Page - 1))
        .Take(queryParameters.Size);

    return Ok(await products.ToListAsync());
}
```

Here, we add two `WHERE` clauses to our LINQ query to filter products based on the `Name` and `Sku` query parameters. The SKU search is case sensitive, while the name search is case insensitive and also allows for substring matches.

### Testing the Searching <a name="testing-the-searching"></a>

Finally, we can test our search functionality by calling the products endpoint with different search parameters. For example, we can search for a specific SKU or for a product name containing a certain substring. Regardless of casing, products with names that include the specified substring will be returned.

Note: Keep in mind that the SKU search is case sensitive, so "AWMPS" and "awmps" will return different results.

And with that, we've successfully implemented searching functionality in our ASP.NET Core API! As with filtering and pagination, this functionality will significantly improve the usability of our API and allow clients to find exactly what they're looking for.

## Adding Advanced Searching <a name="adding-advanced-searching"></a>

To make our API more flexible and user-friendly, we can introduce an advanced search feature. This feature will allow users to perform a search that spans multiple fields.

### Updating the ProductQueryParameters Model for Advanced Searching <a name="updating-the-productqueryparameters-model-for-advanced-searching"></a>

First, we need to extend our `ProductQueryParameters` class to include a new `SearchTerm` property.

```csharp
public class ProductQueryParameters : QueryParameters
{
    public string Name { get; set; } = string.Empty;
    public string Sku { get; set; } = string.Empty;
    public string SearchTerm { get; set; } = string.Empty;
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
}
```
If `SearchTerm` is set, it will be used to search both the `Name` and `Sku` fields of the product.

### Updating the GetAllProducts Method for Advanced Searching <a name="updating-the-getallproducts-method-for-advanced-searching"></a>

Next, we need to modify the `GetAllProducts` method to utilize the `SearchTerm` parameter.

```csharp
[HttpGet]
public async Task<IActionResult> GetAllProducts([FromQuery] ProductQueryParameters queryParameters)
{
    IQueryable<Product> products = _context.Products;

    if (!string.IsNullOrEmpty(queryParameters.SearchTerm))
    {
        products = products.Where(
            p => p.Sku.ToLower().Contains(queryParameters.SearchTerm.ToLower()) 
            || p.Name.ToLower().Contains(queryParameters.SearchTerm.ToLower())
        );
    }

    if (!string.IsNullOrEmpty(queryParameters.Sku))
    {
        products = products.Where(p => p.Sku == queryParameters.Sku);
    }

    if (!string.IsNullOrEmpty(queryParameters.Name))
    {
        products = products.Where(p => p.Name.ToLower().Contains(queryParameters.Name.ToLower()));
    }

    if (queryParameters.MinPrice != null)
    {
        products = products.Where(p => p.Price >= queryParameters.MinPrice.Value);
    }

    if (queryParameters.MaxPrice != null)
    {
        products = products.Where(p => p.Price <= queryParameters.MaxPrice.Value);
    }

    products = products.Skip(queryParameters.Size * (queryParameters.Page - 1))
                       .Take(queryParameters.Size);

    return Ok(await products.ToListAsync());
}
```
Here, we first check whether `SearchTerm` is set. If it is, we perform a case-insensitive search on both `Sku` and `Name` fields. If specific Sku and/or Name are set, we then filter the result based on those fields. 

### Testing the Advanced Searching <a name="testing-the-advanced-searching"></a>

With these modifications in place, we can now test the advanced search functionality. To do this, we can use an API testing tool like Postman. 

In Postman, we can test this new feature by performing a GET request to the `products` endpoint and passing the `SearchTerm` parameter. We can see that our API successfully returns products that have the search term in either their `Name` or `Sku` fields. This demonstrates that our advanced search feature is working as expected.

## Implementing Sorting <a name="implementing-sorting"></a>

To enhance the usability of our API, we can introduce a sorting feature. This feature allows users to sort the result set by any property of the product.

### Updating the QueryParameters Model for Sorting <a name="updating-the-queryparameters-model-for-sorting"></a>

First, we need to add a couple of properties to our `QueryParameters` class to support sorting. One for the property by which the data should be sorted and the other for the order of sorting.

```csharp
public class QueryParameters
{
    private const int _maxSize = 100;
    private int _size = 50;

    public int Page { get; set; }

    public int Size
    {
        get
        {
            return _size;
        }
        set
        {
            _size = Math.Min(_maxSize, value);
        }
    }

    public string SortBy { get; set; } = "Id";
    private string _sortOrder = "ascending";

    public string SortOrder
    {
        get
        {
            return _sortOrder;
        }
        set
        {
            if (value == "ascending" || value == "descending")
            {
                _sortOrder = value;
            }
        }
    }
}
```
Here we added `SortBy` property, with a default value of "Id", and `SortOrder` property, with a default value of "ascending". 

### Adding IQueryableExtensions Class for Sorting <a name="adding-iqueryableextensions-class-for-sorting"></a>

To help us sort the data, we will create an extension method for `IQueryable`. This extension method, `OrderByCustom`, will take in the property and the sort order and generate a lambda expression to sort the data.

```csharp
public static class

 IQueryableExtensions
{
    public static IQueryable<T> OrderByCustom<T>(this IQueryable<T> source, string propertyName, string sortOrder)
    {
        // Implementation here...
    }
}
```

The method `OrderByCustom` uses reflection to dynamically generate a lambda expression based on the property name provided.

### Updating the GetAllProducts Method for Sorting <a name="updating-the-getallproducts-method-for-sorting"></a>

With the `OrderByCustom` method in place, we can now update our `GetAllProducts` method to support sorting.

```csharp
public async Task<IActionResult> GetAllProducts([FromQuery] ProductQueryParameters queryParameters)
{
    IQueryable<Product> products = _context.Products;

    if (queryParameters.MinPrice != null &&
        queryParameters.MaxPrice != null)
    {
        products = products.Where(
            p => p.Price >= queryParameters.MinPrice.Value &&
                 p.Price <= queryParameters.MaxPrice.Value);
    }

    if (!string.IsNullOrEmpty(queryParameters.SearchTerm))
    {
        if (queryParameters.SearchTerm.ToLower() == "true" ||
            queryParameters.SearchTerm.ToLower() == "false")
        {
            var isCheap = queryParameters.SearchTerm.ToLower() == "true";
            products = products.Where(p => p.IsCheap == isCheap);
        }
        else
        {
            products = products.Where(
                p => p.Name.ToLower().Contains(queryParameters.SearchTerm.ToLower()));
        }
    }

    if (!string.IsNullOrEmpty(queryParameters.SortBy))
    {
        if (typeof(Product).GetProperty(queryParameters.SortBy) != null)
        {
            products = products.OrderByCustom(queryParameters.SortBy, queryParameters.SortOrder);
        }
    }

    products = products
        .Skip(queryParameters.Size * (queryParameters.Page - 1))
        .Take(queryParameters.Size);

    return Ok(await products.ToListAsync());
}
```
In this updated method, we first check whether the `SortBy` property is set. If it is, we check whether the `Product` class has a property by that name. If it does, we use our custom extension method to sort the products based on the `SortBy` and `SortOrder` properties.

### Testing the Sorting <a name="testing-the-sorting"></a>

Now we can test our sorting functionality using a tool like Postman. By making a GET request to our `products` endpoint and passing `SortBy` and `SortOrder` as query parameters, we can confirm that our API correctly sorts the result set based on the specified properties. For instance, using `SortBy=Price&SortOrder=descending` should return the products ordered by price in descending order.
