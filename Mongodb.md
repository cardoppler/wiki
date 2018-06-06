- https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/
- https://www.nuget.org/packages/MongoDB.Driver.Core/

```
"C:\Program Files\MongoDB\Server\3.4\bin\mongo.exe"

> use ProxyDB
> db.createCollection('Proxies')
> db.Proxies.insert({'ProxyID':1,'ProxyURL':'http://some_proxy_url:8080'})
> db.Proxies.find({})
```