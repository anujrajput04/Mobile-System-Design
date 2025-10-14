## Content Delivery Network
Content Delivery Network (CDN) is a geographically distributed network of proxy servers and data centers that delivery content, especially static assets, closer to the end users, improving latency, availability, and scalability

#### Why use a CDN
- **Lower latency**: Requests are served from edge servers near the client
- **Improved availability**: Redundancy across multiple nodes mitigates origin server failures
- **Bandwidth offloading**: Reduces load on the origin server
- **DDoS protection**: Many CDNs provide rate-limiting and attack mitigation features

#### How CDNs Work
1. User requests content (eg. https://cdn.example.com/image.jpg)
2. DNS resolution routes to the closest edge server
3. If the edge server has a cached version, it returns it
4. If not, it fetches from the origin, caches it, then serves the client
5. Subsequent users in the region are served from the cache

#### Use Cases
|Use Case|Content Type|CDN Benefit|
|---|---|---|
|Static Assets|JS, CSS, images|Quick load times|
|Video Streaming|HLS, MPEG-DASH|Progressive delivery|
|API Responses|JSON/XML|Low latency (when edge caching enabled)|
|App Updates|Binary files|Efficient distribution|
#### Configurable Behaviors
- **Time to Live (TTL)** headers for controlling cache duration
- **Cache Invalidation APIs** for purging stale data
- **Edge logic** for conditional routing or transformation
- **Custom headers** for auth or debugging

#### CDN for APIs
- Not suitable for highly dynamic, personalized responses unless combined with cache keys or token based edge logic.
- Works well for:
    - Static versioned APIs (`/v1/countries`)
    - Rate limiting and IP filtering at the edge
    - Fast failover from primary to fallback origin