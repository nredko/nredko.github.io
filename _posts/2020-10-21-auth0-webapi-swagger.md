---
title: Swagger-UI with authentication via Auth0
tags: [aspnetcore-3.1, webapi, auth0, swagger]
style: fill
color: success
description: How to implement aspnetcore-3.1 webapi application with embedded swagger-ui using auth0.com authentication
---

## Auth0

[Auth0.com](https://auth0.com) is a great service which allows you to not care of authentication, implementing social logins, one-time passwords, SMS verifications etc.

They have great [tutorials](https://auth0.com/docs) how to use their service in almost all possible situations using a lot of imlementation environments, programming languages and frameworks.

This article concerns quite specific case when you have implemented .net core 3.1 webapi application (to use it by separate front-end or native apps) and would like to provide Swagger UI for your REST API.

## Swagger

If you've found this page then most possible you know what is [Swagger](http://swagger.io), how to use [Swashbuckle Swagger](https://docs.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle?view=aspnetcore-3.1) in .Net Core applications and how useful it is for testing and documenting your API.

The only problem is that configuration of authentication in swagger-ui is not documented for Auth0. There is a lot of [topics](https://community.auth0.com/search?q=swagger) by auth0 community but there is no good example.

## Solution

If you already are using webapi with some frontend - you already have configured auth0 [application](https://auth0.com/docs/quickstart/spa/vanillajs#configure-auth0) and [api](https://auth0.com/docs/quickstart/backend/aspnet-core-webapi#configure-auth0-apis) according their docs.

To use swagger-ui you'll need to have settings from both:

Application:
* Domain
* Client ID
* Client Secret

API: 
  * Audience

Put them to your appsettings.json

```json
  "Auth0": {
    "Domain": "YOUR_AUTH0_DOMAIN",
    "ClientId": "YOUR_CLIENT_ID",
    "ClientSecret": "YOUR_CLIENT_SECRET",
    "Audience": "YOUR_UNIQUE_IDENTIFIER"
  }
```

### aspnetcore project configuration

Auth0 will not work with `http://` scheme so configure your `applicationUrl` as `https://`. 

```json
{
  "$schema": "http://json.schemastore.org/launchsettings.json",
  "profiles": {
    "WebAPIApplication": {
      "commandName": "Project",
      "launchBrowser": true,
      "launchUrl": "swagger",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "https://localhost:5000"
    }
  }
}
```

### Significant auth0 configuration options


All URL on Application settings page should point to your deployment URL, `https://localhost:5000` in my sample.


Also, you should set signature algorithm to RS256 (by default auth0 set HS256, which will not work):

Application -> Advanced Settings -> OAuth -> JsonWebToken Signature Algorithm -> RS256


### Code


Configuration of API auth in .net application is exactly as in [this sample](https://auth0.com/docs/quickstart/backend/aspnet-core-webapi). Below are required additions in `Startup.cs` for Swagger UI:

**Configure()**
```c# 
  app.UseSwagger();
  app.UseSwaggerUI(settings =>
  {
      settings.SwaggerEndpoint("/swagger/v1/swagger.json", "API v1.0");
      settings.OAuthClientId(Configuration["Auth0:ClientId"]);
      settings.OAuthClientSecret(Configuration["Auth0:ClientSecret"]);
      settings.OAuthUsePkce();
  });
```

**ConfigureServices()**
```c# 
  services.AddSwaggerGen(options =>
  {
      options.SwaggerDoc("v1", new Microsoft.OpenApi.Models.OpenApiInfo
      {
          Title = "API Documentation",
          Version = "v1.0",
          Description = ""
      });
      options.ResolveConflictingActions(x => x.First());
      options.AddSecurityDefinition("oauth2", new OpenApiSecurityScheme
      {
          Type = SecuritySchemeType.OAuth2,
          BearerFormat = "JWT",
          Flows = new OpenApiOAuthFlows
          {
              Implicit  = new OpenApiOAuthFlow
              {
                  TokenUrl = new Uri($"https://{Configuration["Auth0:Domain"]}/oauth/token"),
                  AuthorizationUrl = new Uri($"https://{Configuration["Auth0:Domain"]}/authorize?audience={Configuration["Auth0:Audience"]}"),
                  Scopes = new Dictionary<string, string>
                  {
                      { "openid", "OpenId" },
                  
                  }
              }
          }
      });
      options.AddSecurityRequirement(new OpenApiSecurityRequirement
      {
          {
              new OpenApiSecurityScheme
              {
                  Reference = new OpenApiReference { Type = ReferenceType.SecurityScheme, Id = "oauth2" }
              },
              new[] { "openid" }
          }
      });

      var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
      var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
      options.IncludeXmlComments(xmlPath);

  });
```

![](/assets/img/auth0-swagger.png)

## Github

Here is the [sample project](https://github.com/nredko/auth0-aspnetcore-webapi-samples/tree/master/Samples/user-info-swagger)
