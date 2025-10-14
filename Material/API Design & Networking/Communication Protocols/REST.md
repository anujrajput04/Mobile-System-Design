## Representational State Transfer
REST is an architectural style for designing networked applications. It uses standard HTTP methods (GET, POST, PUT, DELETE, etc) to perform CRUD operations on resources, which are identified by URLs.

**Key Principles:**
- **Stateless**: Each request contains all information to process it
- **Resource based**: Everything is a resource (eg. `/users`, `/orders/123`)
- **Uniform Interface**: Standard methods and status codes
- **Representation**: Resources can be represented in JSON, XML, etc
- **Client-Server Separation**: Clear boundary between frontend and backend

**Mobile Specific Considerations**:

| Concern        | Impact on Mobile                                   |
| -------------- | -------------------------------------------------- |
| Bandwidth      | Optimize payloads - use only required fields       |
| Latency        | Batch requests or reduce roundtrips                |
| Caching        | HTTP caching headers must be respected             |
| Error Handling | Graceful degradation required in poor connectivity |
| Offline Mode   | Needs local persistence + sync mechanisms          |
| Versioning     | Critical to avoid breaking old app versions        |

**Example - Swift iOS**:
```swift
/// GET Request using `URLSession` and Swift Concurrency
struct User: Decodable {
	let id: Int
	let name: String
}

func fetchUser() async throws -> User {
	let url = URL(string: "https://api.example.com/users/1")!
	let (data, _) = try await URLSession.shared.data(from: url)
	return try JSONDecoder().decode(User.self, from: data)
}

/// Using Combine
URLSession.shared.dataTaskPublisher(for: url)
	.map(\.data)
	.decode(type: User.self, decoder: JSONDecoder())
	.sink(receiveCompletion: { ... }, receiveValue: { user in ... })
```

**Cross-Platform & Backend Expectations:**
- Define **resource hierarchy** and **endpoint structure** shared across iOS/Android
- Create **single OpenAPI sepc** (Swagger) to auto-generate client code.
- Make backend **idempotent** and **stateless** to support mobile retries.
- Think of **pagination**, **sorting**, **filtering** as consistent, predictable patterns.
- Decide on **authentication** (OAuth2, token-based, API keys)
- Discuss **versioning** strategies: URI-based (`/v1/`), header-based, or media-type.

**Trade-offs:**

| Advantage                           | Disadvantage                                 |
| ----------------------------------- | -------------------------------------------- |
| Simplicity and widespread adoption  | Can lead to chatty APIs (multiple requests)  |
| Caching is easy with HTTP           | No standard for relationships (N+1 problems) |
| Easy for clients to debug and test  | Poor for complex queries or aggregation      |
| Tools: Postman, Charles Proxy, cURL | URL becomes verbose and unstructured         |

### Interview Talking Points
- Explain how you modeled API requests in your iOS app (eg. `UserService` layer)
- How you handled error codes (eg. 401 vs 500 vs 429) with retry/backoff
- How REST endpoints influenced your mobile architecture (eg. MVC/MVVM + Repository)
- Talk about negotiating API contracts with backend (eg. reducing payload, adding filters)

Quick REST Endpoint Design Example
```http
GET /users/1
```
```http
POST /orders
```
```http
PUT /orders/123
```
```http
DELETE /orders/123
```
```http
GET /products?category=shoes&page=2
```
Make sure to cover pagination (`?limit=20&offset=40`), filtering, sorting, and field selection.