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