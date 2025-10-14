## HTTP Caching

### Cache-Control Header
Controls how responses are cached and re-used

**Example**:
```http
Cache-Control: public, max-age=3600
```

**Directives**:
- `no-cache`: always revalidate with server
- `no-store`: don't cache
- `public`: cacheable by any cache
- `private`: cacheable only by client
- `max-age=N`: cache lifespan in seconds

### ETag / Last-Modified
Used for **conditional requrests**
- `ETag`: hash of resource (strong validator)
- `Last-Modified`: timestamp (weak validator)

**Workflow**:
- Client caches response with ETag `xyz123`
- On next request: `If-None-Match: xyz123`
- Server responds:
	- `304 Not Modified` if unchanged
	- `200 OK` with new content if changed