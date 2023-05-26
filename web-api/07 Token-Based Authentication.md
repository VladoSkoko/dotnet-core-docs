# Implementing Token-Based Authentication in .NET Core

In this guide, we are going to cover the implementation of token-based authentication. Using .NET Core, we will build an API, which uses tokens to authenticate the users.

## Table of Contents
- [Adding Support for Controllers](#Adding-Support-for-Controllers)
- [Creating the Token Controller](#Creating-the-Token-Controller)
- [Implementing the Login Endpoint](#Implementing-the-Login-Endpoint)
- [Creating and Validating the Token](#Creating-and-Validating-the-Token)
  - [Using Token Authentication in a Protected Endpoint](#using-token-authentication)
  - [Configuring JWT Authentication](#configuring-jwt-authentication)
  - [The Authorize Attribute](#the-authorize-attribute)

<a name="Adding-Support-for-Controllers"></a>
## Adding Support for Controllers
The initial setup includes adding support for controllers in the application. This step is carried out in the `Program.cs` file of our pages-based web application. Here is how you can do it:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

In the `Startup.cs` file, we use `services.AddControllers()` to add support for controllers.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
}
```

We then map these controllers using `app.MapControllers()` in the `Configure` method:

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ...

    app.MapControllers();

    // ...
}
```

<a name="Creating-the-Token-Controller"></a>
## Creating the Token Controller
In the controllers folder, we create an API controller named `TokenController`. This controller will act as an endpoint to generate a token and also serve as a protected API endpoint.

```csharp
public class TokenController : ControllerBase
{
    private readonly UserManager<IdentityUser> _userManager;

    public TokenController(UserManager<IdentityUser> userManager)
    {
        _userManager = userManager;
    }
}
```

<a name="Implementing-the-Login-Endpoint"></a>
## Implementing the Login Endpoint
We will implement an HTTP POST endpoint called `Login`. This endpoint expects an `InputModel`, which holds a username or email address, a password, and a few other elements that aren't used in our implementation.

```csharp
[HttpPost("login")]
public async Task<IActionResult> Login([FromBody] InputModel model)
{
    var user = await _userManager.FindByEmailAsync(model.Email);
    if (user == null)
    {
        return Unauthorized();
    }

    var validPassword = await _userManager.CheckPasswordAsync(user, model.Password);
    if (!validPassword)
    {
        return Unauthorized();
    }

    // create token here

    return Ok();
}
```

<a name="Creating-and-Validating-the-Token"></a>
## Creating and Validating the Token
In the `Login` method, after validating the user, we generate a token containing a list of claims, such as the user's name, email address, and a unique identifier (GUID).

We use a symmetric key, known to both the code that generates and validates the token, to sign the JWT. In this case, we'll use a hardcoded key for simplicity, but in a real-world application, this should be stored securely and not be hardcoded.

The token will be valid for 20 minutes. This duration is arbitrary and can be modified based on the needs of your application.

```csharp
[HttpPost("login")]
public async Task<IActionResult> Login([FromBody] InputModel model)
{
    // validation steps...

    var claims = new List<Claim>
    {
        new Claim(ClaimTypes.Name, user.UserName),
        new Claim(ClaimTypes.Email, user.Email),
        new Claim(ClaimTypes.NameIdentifier, user.Id),
    };

    var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("Your_Secret_Key"));
    var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
    var expiry = DateTime.Now.AddMinutes(20);

    var tokenDescriptor = new SecurityTokenDescriptor
    {
        Subject = new ClaimsIdentity(claims),
        Expires = expiry,
        SigningCredentials = creds,
        Issuer = "Your_Issuer",
        Audience = "Your_Audience"
    };

    var tokenHandler = new JwtSecurityTokenHandler();
    var securityToken = tokenHandler.CreateToken(tokenDescriptor);

    return Ok(new { token = tokenHandler.WriteToken(securityToken) });
}
```
We use `SecurityTokenDescriptor` to define the properties of the token. We then pass this descriptor to the `CreateToken` method of `JwtSecurityTokenHandler` to generate the JWT. The `WriteToken` method is then used to serialize the token into a string that can be passed back in the response. This token can then be used to access other endpoints in the application that require authentication.

<a name="using-token-authentication"></a>
### Using Token Authentication in a Protected Endpoint

The next phase is utilizing the JWT to access a protected endpoint in our API. To make this possible, we'll implement a new endpoint that checks the provided JWT before it returns the requested data.

In the `TokenController`, we can add a new endpoint, `TestToken`. The token, which was previously obtained from the `Login` endpoint, will be required to access this endpoint.

```csharp
[HttpGet("test")]
[Authorize]
public async Task<IActionResult> TestToken()
{
    return Ok();
}
```
We mark this endpoint with the `Authorize` attribute, which requires a valid JWT to be provided in the `Authorization` header of the request. This JWT is validated using the secret key that was used to create it. If the token is valid, the request is allowed to proceed. If the token is not valid, a 401 Unauthorized response is returned.

<a name="configuring-jwt-authentication"></a>
### Configuring JWT Authentication

The final step is to configure the application to use JWT for authentication. This configuration occurs in the `Program.cs` file. You need to add the `AddAuthentication` and `AddJwtBearer` calls in the `ConfigureServices` method:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddJwtBearer(options =>
            {
                options.TokenValidationParameters = new TokenValidationParameters
                {
                    ValidateIssuer = true,
                    ValidateAudience = true,
                    ValidateLifetime = true,
                    ValidateIssuerSigningKey = true,
                    ValidIssuer = "Your_Issuer",
                    ValidAudience = "Your_Audience",
                    IssuerSigningKey = new SymmetricSecurityKey(
                        Encoding.UTF8.GetBytes("Your_Secret_Key"))
                };
            });
}
```

In the `Configure` method, ensure that `UseAuthentication` and `UseAuthorization` are called:

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // existing configuration...

    app.UseAuthentication();
    app.UseAuthorization();

    // existing configuration...
}
```

In the `ConfigureServices` method, the `AddAuthentication` call registers the authentication services in the application, while `AddJwtBearer` configures the JWT bearer authentication scheme. The `TokenValidationParameters` object is used to specify how the incoming JWT should be validated.

The `UseAuthentication` and `UseAuthorization` calls in the `Configure` method enable the authentication and authorization middleware in the application's request pipeline.

With this configuration in place, any request to the `/api/token/test` endpoint that doesn't provide a valid JWT will receive a 401 Unauthorized response. The client can obtain a valid JWT by making a POST request to the `/api/token/login` endpoint with valid user credentials.

These changes allow us to add token-based authentication to our API. This ensures that only authorized users, those with a valid token, can access protected resources.

<a name="the-authorize-attribute"></a>
### The Authorize Attribute

The `[Authorize]` attribute can specify an `AuthenticationSchemes` property to instruct the authentication middleware which authentication scheme(s) to use. The property accepts a comma-separated list of authentication schemes.

When no scheme is specified in the `[Authorize]` attribute, the default authentication scheme set up in the `Startup.cs` or `Program.cs` file will be used. In our previous example, we used the `AddAuthentication` method to set `JwtBearerDefaults.AuthenticationScheme` as the default authentication scheme. Hence, the `[Authorize]` attribute without a scheme would use the JWT Bearer authentication scheme by default.

If we want to specify the JWT Bearer scheme explicitly in the `[Authorize]` attribute, we could write it like this:

```csharp
[HttpGet("test")]
[Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
public async Task<IActionResult> TestToken()
{
    return Ok();
}
```

Note that specifying an authentication scheme in the `[Authorize]` attribute would override the default authentication scheme for that specific endpoint. This can be useful when dealing with multiple authentication schemes in the same application.
