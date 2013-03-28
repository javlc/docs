# Using Auth0 with ASP.NET

This tutorial explains how to integrate Auth0 with an ASP.NET application (any kind: WebForms, MVC 1, 2, 3 or 4 and even Web API).

## Before you start

You will need Visual Studio 2010 or 2012.

## Integrating Auth0 with ASP.NET

#### 1. Install Auth0-MVC NuGet package

Use the NuGet Package Manager (Tools -> Library Package Manager -> Package Manager Console) to install the **Auth0-MVC** package, running the command:

```
Install-Package Auth0-ASPNET
```

> This package creates an ASP.NET Http Handler (`LoginCallback.ashx`) that will be responsible for the token exchange (based on OpenID Connect / OAuth) and setting a cookie that will be used for subsequent requests. It will also register a module based on the WIF `SessionAuthenticationModule` that will be responsible for serializing/deserializing the session cookie.

#### 2. Setting up the callback URL in Auth0

Run your web application and go to [Auth0 Settings](https://app.auth0.com/#/settings) and make sure to set the callback URL to your application URL:

```
http://localhost:port/LoginCallback.ashx
```

![](img/settings-callback-aspnet.png)

#### 3. Filling Web.Config with your Auth0 settings

The NuGet package also created four settings on `<appSettings>`. Replace those with the following settings:

```
<add key="auth0:ClientId" value="@@account.clientId@@" />
<add key="auth0:ClientSecret" value="@@account.clientSecret@@" />
<add key="auth0:Domain" value="@@account.namespace@@" />
<add key="auth0:CallbackUrl" value="@@account.callback@@" />
```

#### 4. Triggering login manually or integrating the Auth0 widget

Open your master page (or wherever you have the Log On link) and use the following to open the login widget.

```
<script src="https://sdk.auth0.com/auth0.js#client=@@account.clientId@@&scope=openid"></script>
<a href="javascript: window.Auth0.signIn({onestep: true})">Log On</a>
```

> Notice we are injecting a JavaScript that will create a global variable `window.Auth0`. This is used to invoke the widget programatically. Alternatively, you could have used a regular link and redirect the users straight to the desired connection. For example, this link `https://@@account.namespace@@/authorize?response_type=code&scope=openid`
`&client_id=@@account.clientId@@`
`&redirect_uri=@@account.callback@@`
`&connection=google-oauth2` would redirect the user straight to the Google login page. Using this mechanism, you have full control of the user experience.

#### 3. Accessing user information

Once the user succesfuly authenticated to the application, a `ClaimsPrincipal` will be generated which can be accessed through the `User` property or `Thread.CurrentPrincipal`

    public ActionResult Index() 
    {
        var claims = (User.Identity as IClaimsIdentity).Claims
        string email = claims.SingleOrDefault(c => c.ClaimType == "email");
    }
    
**Congratulations!**

----

### More information...

#### Authorization

You can use the usual authorization techniques since the `LoginCallback.ashx` handler and the Http Module will generate an `IPrincipal` on each request. This means you can use the declarative `[Authorize]` or `<location path='..'>` protection or code-based checks like `User.Identity.IsAuthenticated`

#### Log out

To clear the cookie generated on login, use the `ClaimsCookie.ClaimsCookieModule.Instance.SignOut()` method.

#### Flow the identity to a WCF service

If you want to flow the identity of the user logged in to a web site, to a WCF service or an API, you have to use the `scope=openid` parameter on the login (as shown in the example above). When sending that paramter, Auth0 will generate an `id_token` which is a [JsonWebToken](http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-06) that can be either send straight to your service or it can be exchanged to generate an `ActAs` token. . Notice that by default it will only include the user id as part of the claims. If you want to get the full claim set for the user, use `scope=openid%20profile`. [Read more about this](/wcf-tutorial).

#### Manage environments: Dev, Test, Production

We recommend creating one application per environment in Auth0. You can create new applications under the same domain from [Auth0 Settings](https://app.auth0.com/#/settings).

![](img/environments.png)

Each application has a different `Client Id` and `Client Secret` and can be configured with a different callback URL. You can use the [Web.config transformations](http://msdn.microsoft.com/en-us/library/dd465326.aspx) to apply a transformation depening on the Build Configuration you use. For instance

`Web.config`
```
<add key="auth0:ClientId" value="YOUR_DEV_CLIENT_ID" />
<add key="auth0:ClientSecret" value="YOUR_DEV_CLIENT_SECRET" />
<add key="auth0:CallbackUrl" value="http://localhost:port/LoginCallback.ashx" />
```

`Web.Release.config`
```
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <appSettings>
    <add key="auth0:ClientId" value="YOUR_PROD_CLIENT_ID" xdt:Transform="Replace" />
    <add key="auth0:ClientSecret" value="YOUR_PROD_CLIENT_SECRET" xdt:Transform="Replace" />
    <add key="auth0:CallbackUrl" value="http://mysite.com/LoginCallback.ashx" xdt:Transform="Replace" />
  </appSettings>
</configuration>
```

Then, whenever you have to reference the ClientID, Secret or callback, you use this syntax:

```
<script src="https://sdk.auth0.com/auth0.js#client=@System.Configuration.ConfigurationManager.AppSettings["auth0:ClientId"]&scope=openid"></script>
```