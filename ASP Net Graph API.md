### Nuget package required for Graph API
* Azure.Identity
* Azure.Identity


### Permission need to provide from Azure App Registration
* Open your registered app in Azure Active Directory
* Open API Permission section
* Add a permission
* Select Microsoft Graph
* Select Application Permission
* Add below listed permission if you want user or group details
![[Pasted image 20230202194723.png]]

## Fetch Azure User details By Azure Object ID

```C#

public async Task<string> getUserDisplayNameFromAzure(string objectId)
        {
            
            var clientSecret = ConfigurationManager.AppSettings["ClientSecret"];
            var clientId = ConfigurationManager.AppSettings["ida:ClientId"];
            var tenant = ConfigurationManager.AppSettings["TenantId"];
                       
            var scopes = new[] { "https://graph.microsoft.com/.default" };
            
                var options = new TokenCredentialOptions
                {
                    AuthorityHost = AzureAuthorityHosts.AzurePublicCloud
                };

                var clientSecretCredential = new ClientSecretCredential(
                    tenant, clientId, clientSecret, options);

                var graphClient = new GraphServiceClient(clientSecretCredential, scopes);

                var userDetails = await graphClient.Users[objectId]
            .Request()            .Select("displayName,givenName,postalCode,identities,OnPremisesSamAccountName,Mail")
            .GetAsync();

                if (userDetails != null)
                {
                    return userDetails.displayName;
                    
                }
                else
                {
                    return "";

                }            
        }
```


## Fetch User List By Security Group ID

```C#
        public async Task<List<userData>> getAllUserOfGroupByGroupId(string groupId)
        {
            var tenant = ConfigurationManager.AppSettings["TenantId"];
            var clientId = ConfigurationManager.AppSettings["ida:ClientId"];
            var clientSecret = ConfigurationManager.AppSettings["ClientSecret"];
            var scopes = new[] { "https://graph.microsoft.com/.default" };
            
            List<userData> groupUsers = new List<userData>();
            

            var options = new TokenCredentialOptions
            {
                AuthorityHost = AzureAuthorityHosts.AzurePublicCloud
            };

            var queryOptions = new List<QueryOption>()
{
    new QueryOption("$count", "true")
};

            var clientSecretCredential = new ClientSecretCredential(
                tenant, clientId, clientSecret, options);

            var graphClient = new GraphServiceClient(clientSecretCredential, scopes);

            List<Microsoft.Graph.User> graphUser = new List<Microsoft.Graph.User>();
            var groupMembers =  graphClient.Groups[groupId].Members.Request().GetAsync().GetAwaiter().GetResult(); //this will fetch few records and will pass next page id to fecth further group list

            graphUser.AddRange(groupMembers.CurrentPage.OfType<Microsoft.Graph.User>());
            // fetching next page
            while (groupMembers.NextPageRequest != null)
            {
                groupMembers =  groupMembers.NextPageRequest.GetAsync().GetAwaiter().GetResult();
                graphUser.AddRange(groupMembers.CurrentPage.OfType<Microsoft.Graph.User>());
            }
            List<string> userId = new List<string>();
            //graphUser contains all group user
            foreach (var userss in graphUser)
            {               
                var userDetails =  graphClient.Users[userss.Id].Request().Select("displayName,givenName,CompanyName,State,Department,OfficeLocation,JobTitle,MobilePhone,OfficeLocation,city,surname,postalCode,identities,OnPremisesSamAccountName,Mail").GetAsync().GetAwaiter().GetResult();
                if (userDetails != null)
                {             

                    groupUsers.Add(new userData
                    {
                        displayName = userDetails.DisplayName,
                        emailAddr = userDetails.Mail,                        
                        mobileNumber = userDetails.MobilePhone,                
                        postalCode = userDetails.PostalCode,
                        officeType = userDetails.OfficeLocation,
                        city = userDetails.City,
                        company = userDetails.CompanyName,
                        telephoneNumber = userDetails.MobilePhone,
                        title = userDetails.JobTitle,
                        st=userDetails.State
                    });
                }               
            }
            return groupUsers;
        }
```


