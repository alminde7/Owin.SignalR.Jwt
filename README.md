# JwtSignalRAuthentication Middleware

This is an extension that enables SignalR to make use of Json Web Tokens as authentication.

## How to configure the thing?

In order to enable the extension add the following to your AppStart Owin pipeline:

```csharp
app.UseJwtSignalRAuthentication(authQueryKey: "someQueryKey");

app.UseJwtBearerAuthentication(
        new JwtBearerAuthenticationOptions
        {
            AuthenticationMode = AuthenticationMode.Active,
            AllowedAudiences = new[] {AppConfig.ClientId},
            IssuerSecurityTokenProviders = new IIssuerSecurityTokenProvider[]
            {
                new SymmetricKeyIssuerSecurityTokenProvider(AppConfig.AuthenticationIssuer,
                    TextEncodings.Base64Url.Decode(AppConfig.ClientSecret))
            }
        });
```

First add `app.UseJwtSignalRAuthentication()`. It takes one parameter which is the name of the query key.
By default the key is **authtoken** but can be changed by providing another value as a parameter.

Secondly add `app.UseJwtBearerAuthentication()` and provide whatever value match your case. 
`app.UseJwtBearerAuthentication()` is part of the nuget package `Microsoft.Owin.Security.Jwt`, 
it can be downloaded from Nuget. 

## How to use the thing?

First obtain an token from your token provider. When connection to the Jwt-Enabled-SignalR instance,
then provide the token and the aforeconfigured query key. 

```csharp
var token = "SomeJwtToken";
var connection = new HubConnection(url, "authtoken="+ token);
```

Obs. Remember to use the `[Authorize]` attribute on your SignalR methods in order for this to 
have any effect.

## How does it actually work?

The Jwt authentication middleware `app.UseJwtBearerAuthentication()` works by reading the 
`Authorization` header from the request. When connecting to an SignalR instance it is not possible to
add an Authorization header, and that is why we have to provide is as a query parameter. 

`.UseJwtSignalRAuthentication()` takes the token from the query parameter and intercept it into 
an `Authorization` header on the original request. 

By doing that it is possible to make use of Jwt Authentication. 
