
## Azure Side Configuration for Azure AD

### Steps 
1. Login https://portal.azure.com/
2. Search - Azure Active Directory in search tab and open it
3. Select App Registration -> New Registration
4. Write Name of App , Select Platform as Web , add redirect url https://localhost:12345 and click on Register button
5. Now overview page will open of created App registration
6. Go in certificate and Secrets add new client secret and copy the value of secrets 

With above steps you will get the client ID,Tenant ID and Client Sectret.


### If you want only selected user or group to login in your site then follow below steps

1. Open your registred app then go in Token Configuration
2. Add Group Claims and select "Group Assigned to the application" and add it.
3. Go to overview and open url for "Managed application in local directory"
4. Click "Assign Users and Groups" Url
5. Then click on Add User/group and add user or group
6. After that in left panel of "Manage application interface" you will see "properties" open
7. Open it and mark "Assignment Required" to Yes.

-------------------------------------------------------------------------------

## Code Related Things

### Step 1
* SSL to true (press F4 in main project)

### Step 2
* Install below package
```C#
Install-Package Microsoft.Owin
Install-Package Microsoft.Owin.Security.OpenIdConnect
Install-Package Microsoft.Owin.Security.Cookies
Install-Package Microsoft.Owin.Host.SystemWeb
Install-Package Microsoft.IdentityModel.Protocol.Extensions
Install-Package System.IdentityModel.Tokens.Jwt
```


### Step 3
* Add in Web.config Azure related details

```C#
<add key="ida:ClientId" value="{Your client ID}" />
<add key="TenantId" value="{Your Tenant ID}" />
<add key="ida:Tenant" value="common" />
<add key="ida:AADInstance" value="https://login.microsoftonline.com/{0}" />
<add key="ida:PostLogoutRedirectUri" value="https://localhost:12345/" />
```

### Step 4
* Create partial Startup Class 

```c#
public partial class Startup 
    { 
        public void Configuration(IAppBuilder app)
        {
            ConfigureAuth(app);
        }
    }
    ```

### Step 5
* Create class Satrtup.Auth.cs

 ``` c#
public partial class Startup
    {        
        private static string clientId = ConfigurationManager.AppSettings["ida:ClientId"];
        private static string tenant = ConfigurationManager.AppSettings["ida:Tenant"];
        private static string aadInstance = ConfigurationManager.AppSettings["ida:AADInstance"];
        private static string postLogoutRedirectUri = ConfigurationManager.AppSettings["ida:PostLogoutRedirectUri"];

        // Concatenate aadInstance, tenant to form authority value       
        private string authority = string.Format(CultureInfo.InvariantCulture, aadInstance, tenant);


        // ConfigureAuth method  
        public void ConfigureAuth(IAppBuilder app)
        {
            app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

            app.UseCookieAuthentication(new CookieAuthenticationOptions
            {
                CookieManager = new SystemWebCookieManager()
            });

            app.UseOpenIdConnectAuthentication(

                            new OpenIdConnectAuthenticationOptions
                            {
                                ClientId = clientId,
                                Authority = authority,
                                PostLogoutRedirectUri = postLogoutRedirectUri,
                                TokenValidationParameters = new TokenValidationParameters
                                {                                    
                                    ValidateIssuer = false
                                },
                                Notifications = new OpenIdConnectAuthenticationNotifications
                                {
                                    AuthenticationFailed = (context) =>
                                    {
                                        context.HandleResponse();                                        context.OwinContext.Response.Redirect("/Home/Index");
                                        return Task.FromResult(0);
                                    }
                                }
                            });
        }  
    }

```


### Step 6 
* Create Controller for adding singin and signout method

``` c#
public void AzureSignIn()
        {
            if (!Request.IsAuthenticated)
            {

                HttpContext.GetOwinContext()
                    .Authentication.Challenge(new AuthenticationProperties { RedirectUri = "/" },
                        OpenIdConnectAuthenticationDefaults.AuthenticationType);
            }
        }

        public void AzureSignOut()
        {

            HttpContext.GetOwinContext().Authentication.SignOut(
                OpenIdConnectAuthenticationDefaults.AuthenticationType, CookieAuthenticationDefaults.AuthenticationType);

        }
```

### Step 7 
* Create login.cshtml file for login button

```C#
 @if (!Request.IsAuthenticated)
    {
      <style>
                .sso-login {
                    text-decoration: none;
                    display: block;
                    text-align: center;
                    border: 1px solid #ccc;
                    padding: 15px;
                    margin: 25px;
                    font-size: 16px;
                }

                    .sso-login:hover {
                        text-decoration: none;
                    }
            </style>
            <a href="/Account/AzureSignIn" class="sso-login">
                <span><i class="fas fa-sign-in-alt"></i>  Login with Azure</span>
            </a>
    }
    else
    {
        <span><br />Hello @System.Security.Claims.ClaimsPrincipal.Current.FindFirst("name").Value;</span>
        <br />
        @Html.ActionLink("Sign out", "AzureSignOut", "Account")
    }
    @if (!string.IsNullOrWhiteSpace(Request.QueryString["errormessage"]))
    {
        <div style="background-color:red;color:white;font-weight: bold;">Error: @Request.QueryString["errormessage"]</div>
    }
```

### Step 8
* Add authorize attribute over controller for which you need login.

