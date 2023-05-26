# Securing .NET Core Applications with HTTPS 

In .NET Core, you have the option to enforce the usage of HTTPS, which significantly improves the security of your application. This guide will explain how HTTPS enforcement works in .NET Core and provide examples of how you can implement it in your own applications.

## Table of Contents
- [Understanding Configure for HTTPS](#understanding-configure-for-https)
- [Using HTTP Strict Transport Security (HSTS) Middleware](#using-http-strict-transport-security-hsts-middleware)

## Understanding Configure for HTTPS <a name="understanding-configure-for-https"></a>
When you create a new .NET Core API project, you will notice an option labeled "Configure for HTTPS". This option is active by default, and it's there to help secure your application. When you choose this option, two application URLs are automatically configured in the `properties/launchsettings.json` file: one for HTTP and one for HTTPS. The "Configure for HTTPS" option also modifies the behavior of your application so that it redirects all HTTP traffic to HTTPS. This is done by calling `app.UseHTTPSRedirection()` in the `program.cs` file, which adds HTTPS redirection middleware to your application's pipeline. Here's what that looks like:

```csharp
public class Startup
{
    // ...

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        // ...

        app.UseHttpsRedirection();

        // ...
    }
}
```

This middleware ensures that when the web server receives a request over HTTP, it redirects to the same URL using HTTPS instead. This behavior is observable by checking your browser's developer tools and watching for the HTTP 307 status code, which indicates a temporary redirect.

## Using HTTP Strict Transport Security (HSTS) Middleware <a name="using-http-strict-transport-security-hsts-middleware"></a>
In addition to the HTTPS redirection middleware, .NET Core provides another middleware for reinforcing HTTPS usage: `UseHsts()`. This middleware enables HTTP Strict Transport Security (HSTS), a security feature that instructs browsers to only communicate with the server over HTTPS. Here's how to implement it:

```csharp
public class Startup
{
    // ...

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (!env.IsDevelopment())
        {
            app.UseHsts();
        }

        // ...

        app.UseHttpsRedirection();

        // ...
    }
}
```

In this configuration, `UseHsts()` sends a `Strict-Transport-Security` header with each HTTP response, which tells the browser to only use HTTPS when communicating with the server. This setting is remembered by the browser for 30 days by default. 

However, because you might want to test certain behaviors using HTTP or avoid certificate issues during development, it's recommended to enable HSTS only in non-development environments. This is achieved by checking the environment before calling `UseHsts()` as demonstrated in the example above.

For more information on enforcing HTTPS in .NET Core applications, refer to the official [.NET Core documentation](https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl).
