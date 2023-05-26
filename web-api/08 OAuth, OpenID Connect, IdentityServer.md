# Understanding and Implementing OAuth, OpenID Connect, and IdentityServer in .NET Core

This guide provides an overview of securing an API using OAuth and OpenID Connect, and dives deeper into the implementation using Duende IdentityServer in a .NET Core application.

---
**Table of Contents**
- [Understanding OAuth and OpenID Connect](#Understanding-OAuth-and-OpenID-Connect)
- [Introduction to IdentityServer](#Introduction-to-IdentityServer)
- [Adding IdentityServer to the .NET Project](#Adding-IdentityServer-to-the-.NET-Project)
- [Protecting the API using IdentityServer](#Protecting-the-API-using-IdentityServer)
- [Implementing a Client Application](#Implementing-a-Client-Application)

---

<a name="Understanding-OAuth-and-OpenID-Connect"></a>
## Understanding OAuth and OpenID Connect

There are situations where APIs don't have credentials such as username and password combinations because authentication is outsourced to a third party like Google or Microsoft. OAuth and OpenID Connect are commonly used technologies to handle these scenarios.

OAuth 2.0 is the industry standard protocol for authorization, and it provides a wealth of information and specifications as part of the [IETF OAuth working group](https://www.oauth.net/).

On top of OAuth 2.0, we have OpenID Connect, an identity layer that simplifies the process of verifying an identity. OpenID Connect is particularly API-friendly, making it well suited to our needs. 

<a name="Introduction-to-IdentityServer"></a>
## Introduction to IdentityServer

When using .NET Core, we can use a library called IdentityServer to handle OAuth and OpenID Connect. IdentityServer3 and 4 were popular open-source libraries for ASP.NET and ASP.NET Core respectively. However, for more modern versions of ASP.NET Core, we need to use Duende IdentityServer.

Duende Software, the company behind IdentityServer, offers a commercial product with different licenses, including a community edition which is free for non-profit organizations or companies with annual gross revenue of less than 1 million US dollars.

<a name="Adding-IdentityServer-to-the-.NET-Project"></a>
## Adding IdentityServer to the .NET Project

IdentityServer comes with templates that can be used with the .NET CLI or within Visual Studio. The easiest way to install them is using the .NET new command as shown:

```bash
dotnet new install Duende.IdentityServer.Templates
```

After installation, we can add IdentityServer to our solution:

```bash
dotnet new is4inmem -n MyProject.IdentityServer
```

This command creates an IdentityServer project with in-memory stores and test users. Two important files are generated: `Startup.cs` and `Config.cs`. The `Startup.cs` file sets up IdentityServer and adds the test users, while the `Config.cs` file configures the IdentityServer and defines the clients that can talk to it.

<a name="Protecting-the-API-using-IdentityServer"></a>
## Protecting the API using IdentityServer

To secure an API, we add an `Authorize` attribute to our controller:

```csharp
[Authorize]
public class ProductsController : ControllerBase
{
    // ...
}
```

We also need to configure the authentication:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    services.AddAuthentication("Bearer")
        .AddJwtBearer("Bearer", options =>
        {
            options.Authority = "https://localhost:5001";
            options.TokenValidationParameters = new TokenValidationParameters
            {
                ValidateAudience = false
            };
        });
}

public void Configure(IApplicationBuilder app)
{
    app.UseAuthentication();
    app.UseAuthorization();
    // ...
}
```

This configuration specifies that we're using JWT Bearer Authentication. The `options.Authority` should point to the URL where the IdentityServer is running. 

The `ValidateAudience` property is set to false to simplify things for this example, but in a production application, audience validation should be implemented.

## Implementing a Client Application

This section will guide you through the process of creating a client application that communicates with an API protected by Duende IdentityServer.

1. **Adding a new Client application to the solution**

You can use either a Web application or Console Application. In this case, we are going to use a Console Application. Let's name it HPlusSport.ConsoleClient. Make sure to select .NET 6.0 for the version.

2. **Setting up Startup Projects**

In the solution properties, set the startup projects to be the API, IdentityServer, and the Console application. These applications need to start concurrently because the Console application will communicate with IdentityServer and the API.

3. **Setting up HttpClient and Discovery Document**

To start with, create an instance of HttpClient, which will be used to communicate with the server. You will also need to access a "discovery document", which is a document that contains necessary details about IdentityServer and the token endpoint.

```csharp
var client = new HttpClient();
var discoveryDocument = await client.GetDiscoveryDocumentAsync("https://localhost:5001");
```

This example uses the IdentityModel package, a helper project developed by the creators of IdentityServer, to access the discovery document. The URL parameter for GetDiscoveryDocumentAsync is the URL where your IdentityServer instance is running.

4. **Requesting an Access Token**

Next, you will request an access token from IdentityServer:

```csharp
var tokenResponse = await client.RequestClientCredentialsTokenAsync(new ClientCredentialsTokenRequest
{
    Address = discoveryDocument.TokenEndpoint,
    ClientId = "client",
    ClientSecret = "secret",
    Scope = "scope1"
});
```

The ClientCredentialsTokenRequest object specifies the endpoint where the token request will be sent (`discoveryDocument.TokenEndpoint`), as well as the client ID, client secret, and the scope of access. Replace "client", "secret", and "scope1" with the actual values you have configured in your IdentityServer setup.

5. **Displaying the Access Token**

You can now display the access token in your console application to verify that it was retrieved successfully:

```csharp
Console.WriteLine(tokenResponse.AccessToken);
```

6. **Making Requests with the Access Token**

Now that you have an access token, you can use it to make authorized requests to your protected API. Just add the token to the Authorization header of your requests, preceded by the word "Bearer".

This is how you set up a console client application that can authenticate with IdentityServer and make authorized requests to an API. Remember that there are multiple ways to secure APIs with IdentityServer, and this is just one of them. This example demonstrates machine-to-machine communication where the client and server share a known secret.

It's worth noting that for single-page applications or Blazor applications, a common pattern for securing APIs is the Backend-for-Frontend (BFF) pattern. IdentityServer supports this pattern and can help you secure such applications with relatively little additional effort. More information on this can be found on the Duende Software homepage.
