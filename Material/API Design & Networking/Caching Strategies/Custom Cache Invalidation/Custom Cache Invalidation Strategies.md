## Custom Cache Invalidation Strategies
#### Time-based
Invalidate cache after TTL (time to live)
- eg. Invalidate every 15 minutes regardless of headers

#### Tag-based / Key-based
Manual invalidation using custom keys

#### Version-based
Cache keys include API or app version to avoid stale data

#### Dependency-based
Evict cache if related entity changes eg. cache of user feed invalidated when user profile is updated