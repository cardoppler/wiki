# C#

### C# in VSCode:
- Install VSC
- Install code 
- Open VSC
- Open terminal:
```
> dotnet new console -o NewProject
> cd New Project ;; dir
-a----  Thu, 13, 04, 17     11:59            183 Program.cs
-a----  Thu, 13, 04, 17     11:59            170 NewProject.csproj
> code .\Program.cs
> dotnet restore
> dotnet run
Hello World!
```

# Repository
Is a layer of abstraction between me and the real data source (the entity framework). Source can be a db, an xml file, a webservice, ... An API maybe?

# Generics
allow code reuse with type safety

# Delegates type
Variable that points to different methods.
It's  way to invoke methods through a layer of indirection

- **Action**. Returns void.
`Action<double> print = d => Console.WriteLine(d); # Anonymous action method`

- **Function**. Return a value

- **Predicate**. returns a boolean

## Generic Constraints
`... where TEntity : class`

## Events
`public event EventHandler<EventArgs> ItemDiscareded;`



# Watch again from C# Generics:
- Generic Methods and Delegates
`IEnumerable<int> AsEnumerableOf<TOutput>(); ``

# Watch after:
- LINQ
where, orderby select, joint, groupby
- Lambda expressions
- Design patterns (e.g. Repositories)
- Entity Framework

# To read about:
- yield
- anonymous method (use lambda expressions instead)

## Load json file at runtime:
```
        static void Main(string[] args)
        {
            var jsonFile = File.ReadAllText(@"\path\to\file.json");
            dynamic jsonObject = Newtonsoft.Json.JsonConvert.DeserializeObject<dynamic>(jsonFile);
            foreach ( var system_name in jsonObject.system_name)
                Console.WriteLine(system_name);
        }
```
json.file:
```
{
    "systems": [
        {
            "system_name": "some name here",
            "critical": 1,
            ...
        },
        { ... }
    ]
}    
```

# OctopusDeploy API
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using Newtonsoft.Json;
using RestSharp;
using RestSharp.Authenticators;

public class OctopusDeployConnector
{
    string server = "http://yourserver:port";
    string apiKey = "API-SECRET";
    string apiUser = "X-Octopus-ApiKey";

    RestClient client;

    public OctopusDeployConnector()
    {
        client = new RestClient(server);
        client.Authenticator = new HttpBasicAuthenticator(apiUser, apiKey);
    }

    public List<InterruptionItem> getDeploymentInfo()
    {
        var usersListrequest = new RestRequest("/api/users/");
        usersListrequest.AddHeader(apiUser, apiKey);
        var usersListJsonResponse = client.Execute(usersListrequest);
        var usersListResponse = JsonConvert.DeserializeObject<UsersResponse>(usersListJsonResponse.Content);

        // [https://github.com/OctopusDeploy/OctopusDeploy-Api/wiki/Interruptions] "The results will be sorted by date from most recently to least recently created. The response will be a collection of resources. This query uses pagination, so a maximum of 30 items will be returned per page of results."
        var nextPage = "/api/interruptions/";
        var interruptionsList = new List<InterruptionItem>();
        var currentOldest = new DateTimeOffset();
        var today = DateTimeOffset.Now;
        var oldestWanted = today.AddDays(-10); // We only want interruptions within the last 10 days
        do
        {
            var interruptionsListRequest = new RestRequest(nextPage);
            interruptionsListRequest.AddHeader(apiUser, apiKey);
            var interruptionsListJsonResponse = client.Execute(interruptionsListRequest);
            var interruptionsListResponse = JsonConvert.DeserializeObject<InterruptionsResponse>(interruptionsListJsonResponse.Content);
            nextPage = interruptionsListResponse.Links.Where(x => x.Key == "Page.Next").FirstOrDefault().Value;
            interruptionsList.AddRange(interruptionsListResponse.Items);
            currentOldest = interruptionsList.Min(x=> x.Created);
            interruptionsList.Where(x => x.Created == currentOldest);
        } while ( nextPage != null && DateTimeOffset.Compare(currentOldest, oldestWanted) >= 0 );

        interruptionsList.RemoveAll(x => x.Created < oldestWanted);

        foreach (var interruption in interruptionsList)
            interruption.ResponsibleUserEmail = usersListResponse.Items.Where(x => x.Id == interruption.ResponsibleUserId).Select(x => x.EmailAddress).FirstOrDefault();

        return interruptionsList;
    }
}
```
