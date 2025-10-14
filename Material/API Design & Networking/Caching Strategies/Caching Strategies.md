## Caching Strategies
Breakdown of Caching in context of API design and mobile clients

![[HTTP Caching]]

![[In-Memory Cache]]

![[Disk Cache]]

![[Custom Cache Invalidation Strategies]]

### When yo use what

| Use Case                      | Cache Type                            |
| ----------------------------- | ------------------------------------- |
| Fast, short-lived reuse       | In-Memory (NSCache / LRUCache)        |
| Persist across sessions       | Disk Cache                            |
| Controlled by server          | HTTP caching (ETag, Cache-Control)    |
| Customised for business logic | Manual + Version/Tag-based strategies |