---
title: Configuring authentication for minimal APIs
author: safia
description: Learn how to configure authentication and authorization in minimal API apps
ms.author: safia
monikerRange: '>= aspnetcore-7.0'
ms.date: 10/17/2022
uid: fundamentals/minimal-apis/security
---

Minimal APIs support the full spectrum of authentication and authorization options availalbe in ASP.NET and provide some additional functionality to improve the experience for working with authentication.

## Enabling authentication in minimal applications

To enable authentication in an application, invoke the `AddAuthentication` method to register the required authentication services on the application's service provider.

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication();

var app = builder.Build();

app.Run();
```

Typically, a specifically authentication strategy will be used. In the code sample below, the application is configured with support for cookie-based authentication.

```csharp
using Microsoft.AspNetCore.Authentication.Cookie;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication().AddCookie();

var app = builder.Build();

app.Run();
```

In the code sample below, the application is configured with support for JWT-based authentication.

```csharp
using Microsoft.AspNetCore.Authentication.Cookie;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication().AddJwtBearer();

var app = builder.Build();

app.Run();
```

By default, the WebApplication will automatically register the authentication and authorization middlewares if certain authentication and authorization services are enabled. In the code sample below, it is not necessary to invoke `UseAuthentication` or `UseAuthorization` to register the middlewares since `WebApplication` does this automatically after services are added.

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication().AddJwtBearer();
builder.Services.AddAuthorization();

var app = builder.Build();

app.Run();
```

However, if it is necessary to manually register authentication and authorization, as in the case of controlling middleware order, then it is possible to do so. In the code sample below, the authentication middleware will run _after_ the CORS middleware has run.

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication().AddJwtBearer();
builder.Services.AddAuthorization();

var app = builder.Build();

app.Run();
```

### Configuring authentication strategy

Authentication strategies typically support a variety of configurations that are loaded in via options. Minimal application support loading options from configuration for the following authentication strategies:

- JWT bearer-based authentication strategies
- OpenID Connection-based authentication strategies

The ASP.NET framework expects to find these options under the `Authentication:Schemes:{SchemeName}` section in configuration. The `appsettings.json` definition below defines two different schemes, `Bearer` and `LocalAuthIssuer`, with their respective options. The `Authentication:DefaultScheme` option can be used to configure the default authentication strategy that will be used.

```json
{
  "Authentication": {
    "DefaultScheme":  "LocalAuthIssuer",
    "Schemes": {
      "Bearer": {
        "ValidAudiences": [
          "https://localhost:7259",
          "http://localhost:5259"
        ],
        "ValidIssuer": "dotnet-user-jwts"
      },
      "LocalAuthIssuer": {
        "ValidAudiences": [
          "https://localhost:7259",
          "http://localhost:5259"
        ],
        "ValidIssuer": "local-auth"
      }
    }
  }
}
```

In `Program.cs`, we register two JWT bearer-based authentication strategies: one with the default scheme name ("Bearer") and one with the "LocalAuthIssuer" scheme name. The scheme name is used to uniquely identify an authentication strategy and is used as the lookup key when resolving authentication options from config.

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication()
  .AddJwtBearer()
  .AddJwtBearer("LocalAuthIssuer");
  
var app = builder.Build();

app.Run();
```



## Configuring authorization policies in minimal applications

While authentication is used to identify and validate the identity of users against an API, authorization is used to validate and verify access to resources in an API. 

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthorization();

var app = builder.Build();

app.Run();
```

## Using `dotnet user-jwts` to improve development time testing