# Understanding CORS in .NET Core

Cross-Origin Resource Sharing, also known as CORS, is a crucial security measure for modern web applications. It is a mechanism that restricts requests for resources from a different origin than the one serving the resources. This article guides you through understanding and implementing CORS in .NET Core, with detailed descriptions and code examples. By the end of this guide, you will know how to set up CORS and apply CORS policies to a .NET Core application.

### Table of Contents
- [(Optional) Setting Up the Front-end Application](#setting-up-the-front-end-application)
- [Understanding CORS](#understanding-cors)
- [Implementing CORS in .NET Core](#implementing-cors-in-net-core)
- [Running the Application](#running-the-application)

<a name="setting-up-the-front-end-application"></a>
## (Optional) Setting Up the Front-end Application

<details>
<summary>We'll need a front-end application to consume our .NET Core API. If you don't have one, follow these steps to set up a basic ASP.NET Core web application with Razor Pages.</summary>

### Creating a New Web Application

First, let's add a new project to our solution. This will be an ASP.NET Core Web Application with Razor Pages. We'll name it `HPlusSport.Web`.

1. In Visual Studio, right-click on the solution and select `Add` > `New Project`.
2. Select `ASP.NET Core Web App`, then click `Next`.
3. Name the project `HPlusSport.Web`, then click `Next`.
4. In the `Additional Information` section, ensure that `ASP.NET Core 5.0` (or the version you're working with) is selected, and choose `Web Application` as the template. Do not select the "Enable Docker" option.
5. Click `Create`.

### Modifying the Index Page

Once the application is created, we're going to add some output to the `Index.cshtml` page. This output will be a list of products returned by the API. 

First, add an unordered list with an id to the page:

```html
<ul id="productsList"></ul>
```

This list will be populated with JavaScript later.

### Adding JavaScript to Call the API

Now, we'll add a JavaScript section to the `Index.cshtml` page to fetch the products from the API and display them in the list.

We can add a `<script>` tag in the `@section Scripts { }` block of the `Index.cshtml` file, which is intended for JavaScript and will be rendered towards the end of the page.

Here's how the script could look like:

```javascript
@section Scripts {
    <script>
        const apiUrl = "https://localhost:7233/products";
        const productsList = document.getElementById("productsList");

        fetch(apiUrl)
            .then(response => response.json())
            .then(data => showProducts(data))
            .catch(error => console.error('Error:', error));

        function showProducts(products) {
            products.forEach(product => {
                let li = document.createElement("li");
                let text = `${product.name} - $${product.price}`;
                li.appendChild(document.createTextNode(text));
                productsList.appendChild(li);
            });
        }
    </script>
}
```

In this script:

- We specify the URL of our API endpoint, which is running on `https://localhost:7233/products`.
- We then use the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) to make a GET request to this URL. 
- Once we get the response, we convert it into a JavaScript object with `response.json()`.
- If the request is successful, we pass the data to the `showProducts` function, which iterates over the products and creates a list item for each product. Each list item contains the product name and price, which are appended to the products list.
- If there's an error with the request, we log it in the console.

### Running the Application

Now that the front-end application is set up, you can set it as the startup project and run it. The application will make a request to the API and display the list of products on the page.

If you get an error, don't worry! We'll use CORS to allow cross-origin requests from our front-end application to our API.
</details>
  
<a name="understanding-cors"></a>
## Understanding CORS

When we talk about a "cross-origin" request in JavaScript, we're talking about requests between different origins. An origin is defined by three components: the protocol (http or https), the fully qualified domain name, and the port (80 for HTTP and 443 for HTTPS, unless a non-default port is used).

The CORS security feature in browsers blocks JavaScript HTTP requests to different origins, unless the server that hosts the resource agrees to accept the request. This is to prevent potential security risks associated with executing cross-origin scripts.

Here's a basic flow of how CORS works:

1. The client initiates a JavaScript HTTP request (like a `GET`, `POST` request) to a server on a different origin.
2. The browser sends an HTTP OPTIONS request (also known as preflight request) with an `Origin` header to the server. The `Origin` header contains the origin of the requesting site.
3. If the server agrees to the request, it responds with an `Access-Control-Allow-Origin` header, which contains the origin of the client site. If the browser receives this header and the origin matches, the actual request is sent.
4. For requests that do not change the state of the server (like `GET` or `HEAD`), the process is slightly different. The actual request is sent first, with the `Origin` header. If the server agrees, it responds with the `Access-Control-Allow-Origin` header. If this header is received and the origin matches, JavaScript gets access to the data from the server.

Therefore, for the client-side JavaScript to access data from a different origin, the server needs to include the appropriate `Access-Control-Allow-Origin` header in its response.

<a name="implementing-cors-in-net-core"></a>
## Implementing CORS in .NET Core

.NET Core provides robust support for CORS, simplifying the process of enabling and managing CORS in your applications.

We'll start by defining a CORS policy and then applying it to our application. This policy can be applied to specific controllers or actions, or as a default policy for all endpoints.

Here's how we can implement a default CORS policy in our `Program.cs`:

```csharp
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(
        policyBuilder =>
        {
            policyBuilder.WithOrigins("http://localhost:7257")
                         .WithHeaders("X-API-Version");
        });
});
```

In this code snippet:

- `AddCors` is used to add CORS services.
- `AddDefaultPolicy` is used to define a default policy.
- `WithOrigins` is used to specify the origins that are allowed. Replace `"http://localhost:7257"` with your own origin.
- `WithHeaders` is used to allow specific headers in a request. Replace `"X-API-Version"` with your custom headers, if any.

Note: The `AllowAnyOrigin` method, although convenient, should be used with caution. It allows requests from any origin, which could lead to security vulnerabilities.

After defining the policy, we need to add the CORS middleware to the application's pipeline using the `UseCors` method. This should be done before the `MapControllers` call.

```csharp
builder.UseCors();

// Add other middleware, for instance:
builder.MapControllers();
```

The `UseCors` method without any parameters applies the default CORS policy to all endpoints in the application.

<a name="running-the-application"></a>
## Running the Application

Before running the application, make sure that both the web application and the API are running. You can either run them individually or set them as startup projects in your development environment.

When the application is successfully running, the web application will be able to make cross-origin JavaScript HTTP requests to the API, as long as the requests comply with the CORS policy we set up.

To confirm that CORS is working as expected, you can inspect the network activity in your browser's developer tools. Look for the API call in the network tab, and you should be able to see the `Access-Control-Allow-Origin` header in the response headers.

Remember, implementing CORS is essential to ensure the security of your .NET Core applications. Be mindful of the origins and headers you allow in your CORS policy to keep your application secure. For more information on CORS in .NET Core, you can refer to the official [Microsoft documentation](https://docs.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-6.0).
