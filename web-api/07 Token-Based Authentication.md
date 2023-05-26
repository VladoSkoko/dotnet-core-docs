# Implementing Token-Based Authentication in .NET Core

In this guide, we are going to cover the implementation of token-based authentication. Using .NET Core, we will build an API, which uses tokens to authenticate the users.

## Table of Contents
- [Adding Support for Controllers](#Adding-Support-for-Controllers)
- [Creating the Token Controller](#Creating-the-Token-Controller)
- [Implementing the Login Endpoint](#Implementing-the-Login-Endpoint)
- [Creating and Validating the Token](#Creating-and-Validating-the-Token)

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

Next up, we'll create a protected endpoint that expects a valid token from the client for authentication and authorization.
