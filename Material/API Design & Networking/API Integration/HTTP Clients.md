## HTTP Clients
HTTP Clients are libraries used to:
- Initiate network requests (GET, POST, etc)
- Handle request configuration (headers, timeouts, authentication)
- Parse responses
- Manage retries, logging, and errors

As an Architect, reason about:
- Custom interceptors (logging, auth token refresh)
- Caching strategies
- Timeout/resiliency control
- Decoupling of networking from business logic

---
### iOS Clients
**URLSession** (Native, Flexible):
- **Foundation API** for network tasks
- Supports **delegate patterns**, **completion handlers** and **`async/await`**
- Well suited for low-level control over headers, background downloads, multipart uploads

**Example**:
```swift
let url = URL(string: "https://api.example.com/data")!
let (data, _) = try await URLSession.shared.data(from: url)
let result = try JSONDecoder().decode(MyModel.self, from: data)
```

**Alamofire** (High level abstraction):
- Built on `URLSession`
- Handles request building, JSON parsing, retries, chainable calls
- Great for structured requests and complex headers like OAuth

**Advanced Use**:
- **Request interceptors** for token refresh
- **Network Reachability** to delay/retry offline requests

**Example**:
```swift
AF.request("https://api.example.com/data")
  .validate()
  .responseDecodable(of: MyModel.self) { response in
	  switch response.result {
	  case .success(let data): print(data)
	  case .failure(let error): print(error)
	  }
  }
```

---
### Android Clients
**OkHttp** (Low Level but powerful):
- Handles HTTP request lifecycle
- Supports caching, connection pooling, interceptors, retries

**Use when**:
- You need fine control (eg. manual retries, logging, custom SSL pins)

**Example**:
```kotlin
var client = OkHttpClient()
var request = Request.Builder().url("https://api.example.com/data").build()
var response = client.newCall(request).execute()
```

**Retrofit** (High level abstraction):
- Built on OkHttp
- Converts HTTP APIs to Kotlin interfaces
- Auto-decodes JSON to model objects using Moshi or Gson
- Supports RxJava, Kotlin Coroutines, and LiveData

**Example**:
```kotlin
interface ApiService {
	@GET("data")
	suspend fun getData(): MyModel
}
```

---

### Architectural Integration
As Architect you should think about:
- **Clean API Layer** - Networking logic stays outside view/controllers
- **Separation of Concerns** - Decode, validate, and transform in services/repositories
- **Retry + Backoff** - Unified logic in interceptors or URLProtocol stubs
- **Logging** - Hooked at HTTP client level
- **Pluggable mock client** - for testing

### Interview Talking Points
- "We used `URLSession` with `async/await` for lean, modern concurrency. We wrapped it in a `NetworkService` protocol to abstract calls across the app and enable mocking"
- "Alamofire helped us plug in retry + exponential backoff logic. We built a request interceptor for token refresh that retries failed requests"
- "For Android parity, I reviewed their Retrofit interfaces to ensure query param conventions, error shapes, and status codes matched across platforms"
- "I defined a standard `ApiError` enum shared across modules with translations from HTTP codes, JSON parse failures, and timeouts"

### API Integration Layer Maturity
As an Architect:
- Push for a **shared OpenAPI schema** across platforms
- Align **error handling and decoding** logic across iOS and Android
- Introduce **dependency injection** in HTTP clients for testability
- Advocate retry/caching behavior to be **deterministic and observable**
- Prefer **URLRequest building DSLs or factories** to minimize repeated code