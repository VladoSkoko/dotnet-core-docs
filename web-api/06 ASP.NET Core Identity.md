# Implementing ASP.NET Core Identity in your Application

ASP.NET Core Identity is an API that enables user interface (UI) login functionality. It also provides user registration, two-factor authentication, account confirmation, account lockouts, and password reset. This lesson will guide you on how to add ASP.NET Core Identity to your existing application using the Razor Pages web project.

## Table of Contents

- [Adding ASP.NET Core Identity](#adding-aspnet-core-identity)
- [Scaffolding Identity Files](#scaffolding-identity-files)
- [Adjustments After Scaffolding](#adjustments-after-scaffolding)

## Adding ASP.NET Core Identity

To add ASP.NET Core Identity to your project, first, you will need to install a new NuGet package `Microsoft.VisualStudio.Web.CodeGeneration.Design`.

The installation can be done via the .NET CLI by running the following command in your terminal:

```bash
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
```

This package has a few dependencies, so installation might take a while.

## Scaffolding Identity Files

For better customization and control, it is advisable to scaffold the individual files into your project. This step will generate all the necessary Identity-related Razor pages.

Here's the process to scaffold the Identity files:

1. Add a new scaffolded item to your project.
2. Go to the Identity section.
3. Choose to add all Identity files.

You will need to specify a layout page, and the scaffold will respect this. In case you already have a layout page in your project (let's say `Layout.cshtml`), you should use that:

```csharp
// The layout page that should be used
var layout = "Layout.cshtml";
```

Two additional classes need to be specified during scaffolding - `Data Context` and `User` classes. If they don't exist, the scaffolding process will generate them:

```csharp
// If Data Context and User classes don't exist, the scaffolding process creates them
public class ApplicationDbContext : IdentityDbContext
{
    // ...
}

public class ApplicationUser : IdentityUser
{
    // ...
}
```

This scaffolding operation may take a while and can result in an error message on some versions of Visual Studio. If this happens, simply retry the process.

## Adjustments After Scaffolding

Once the scaffolding is complete, you will notice a new `Areas` folder in your project. This folder contains an `Identity` directory and several Razor pages, such as `ConfirmEmail`, `ForgotPasswords`, etc.

There are a few adjustments you might want to make. For example, in the `_LoginPartial.cshtml` file, you can see the login and logout links. You can put these links in your main layout file as follows:

```html
@await Html.PartialAsync("_LoginPartial")
```

Lastly, a common error may occur where the User class cannot be found. This can be resolved by adding the missing `using` directive to your file:

```csharp
using Microsoft.AspNetCore.Identity;
```

By following these steps, you've successfully added ASP.NET Core Identity to your existing application. The Identity system is now ready to manage your users' authentication and authorization processes. You are free to modify the generated files and tailor them to suit your needs.
