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
