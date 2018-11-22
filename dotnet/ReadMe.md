
Install via NuGet (https://www.nuget.org/packages/Xrm.Tools.CRMWebAPI/)

Support for full .NET, .NET Core and .NET Standard projects

Install-Package Xrm.Tools.CRMWebAPI 
Here is how to get an instance of CRMWebAPI passing an AccessToken
````
        public static CRMWebAPI GetAPI()
        {

            CRMWebAPI api = new CRMWebAPI("https://orgname.api.crm.dynamics.com/api/data/v8.0/",
                    "<token>");
            return api;

        }
````
Here is how to get an instance of CRMWebAPI passing an ADAL with a user and password - to understand how to get a client ID visit the walk through here https://msdn.microsoft.com/en-us/library/mt622431.aspx This requires a native application and is not supported by .NET Core

````
  public static CRMWebAPI GetAPI()
  {
      string authority = "https://login.microsoftonline.com/common";
      string clientId = "<clientid>";
      string crmBaseUrl = "https://xx.crm.dynamics.com";

      var authContext = new AuthenticationContext(authority);
      UserCredential userCreds = new UserPasswordCredential("<email>", "<password>");
      var result = authContext.AcquireTokenAsync(crmBaseUrl, clientId, userCreds).Result;
      CRMWebAPI api = new CRMWebAPI(crmBaseUrl + "/api/data/v9.0/", result.AccessToken);
      
      return api;
  }
````

Here is how to get an instance of CRMWebAPI passing an ADAL with a Server-to-server authentication - to understand Server-to-server authentication visit
https://msdn.microsoft.com/en-us/library/mt790168.aspx
````
  public async static Task<CRMWebAPI> GetAPI()
  {
      string authority = "https://login.microsoftonline.com/";
      string clientId = "<clientid>";
      string crmBaseUrl = "https://xx.crm.dynamics.com";
      string clientSecret = "<clientSecret>";
      string tenantID = "<tenantId>"; 

      var clientcred = new ClientCredential(clientId, clientSecret);
      var authContext = new AuthenticationContext(authority + tenantID);
      var authenticationResult = await authContext.AcquireTokenAsync(crmBaseUrl, clientcred);
  
      return new CRMWebAPI(crmBaseUrl + "/api/data/v8.0/", authenticationResult.AccessToken);
  }
````

Here are a few simple examples 
````
        Task.Run(async () =>
            {
                var api = GetAPI();

                dynamic data = new ExpandoObject();
                data.name = "test " + DateTime.Now.ToString();

                Guid createdID = await api.Create("accounts", data);                

                var retrievedObject = await api.Get("accounts", createdID, new CRMGetListOptions() { FormattedValues = true });

                var retrievedObjectEx = await api.Get<ExpandoObject>("accounts", createdID);

                dynamic updateObject = new ExpandoObject();
                updateObject.name = "updated name " + DateTime.Now.ToString();

                var updateResult = await api.Update("accounts", createdID, updateObject);
                //update with an upsert
                var upsertResult = await api.Update("accounts", Guid.NewGuid(), updateObject, Upsert: true);

                await api.Delete("accounts", upsertResult.EntityID);

                var results = await api.GetList("accounts", new CRMGetListOptions() { Top = 5, FormattedValues=true });

                string fetchXml = "<fetch mapping='logical'><entity name='account'><attribute name='accountid'/><attribute name='name'/></entity></fetch>";

                var fetchResults = await api.GetList("accounts", QueryOptions: new CRMGetListOptions() { FetchXml = fetchXml });

                var count = await api.GetCount("accounts");
        }).Wait();
