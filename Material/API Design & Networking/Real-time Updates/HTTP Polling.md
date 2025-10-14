## HTTP Polling
**HTTP Polling** is a technique where clients periodically sends HTTP requests (usually `GET`) to the server to check for updates. It's the simplest and most widely compatible way to **simulate real-time communication**. It is **stateless**, easy to implement, and works over traditional REST infrastructure.

![[HTTP Polling.excalidraw]]

**Typical flow**:
- Every X seconds, the mobile app sends a request like: `GET /notifications?since=last_timestamp`
- Server returns new data or empty response
- Client waits, then polls again

**Mobile-Specific Considerations**:

| Concern                | Impact on Mobile                                              |
| ---------------------- | ------------------------------------------------------------- |
| Battery consumption    | Frequent wakeups drain battery                                |
| Network usage          | High if polling interval is short and responses are empty     |
| Background limitations | iOS suspends tasks, polling stops unless configured           |
| Responsiveness         | Delay = polling interval, latency not instant                 |
| Push alternative       | Not a replacement for critical events like messages or alarms |
You control:
- **Interval** (eg. evet 15s, 30s, 60s)
- **Conditionals** (eg. only poll when screen is active)
- **Differential fetching** (using timestamps or [[Delta Updates#ETag]]s to avoid fetching same data)

**Example - Swift**:
```swift
var timer: Timer?

func startPolling() {
	timer = Timer.scheduledTimer(withTimeInterval: 30, repeats: true) { _ in
		fetchUpdates()
	}
}

func fetchUpdates() {
	let url = URL(string: "https://api.example.com/updates?since=\(lastFetchedTime)")!
	URLSession.shared.dataTask(with: url) { data, response, error in
		// parse and update UI
	}.resume()
}
```

**Better**: Use `DispatchSourceTimer` with control over thread and backpressure
**Best**: Use Combine or async/await with lifecycle-aware throttling

**Cross-Platform & Backend Expectations**:
From an Architect's view:
- **Backend must support delta fetches** eg. `?since=timestamp` or `ETag`-based
- **Rate limiting** to protect backend from floods (especially during app launches)
- Consider **backend caching** so repeated identical polls are inexpensive
Polling can be:
- **Client-initiated** - most common
- **Smart polling** - controlled by [[Feature Flags & Remote Config]], A/B experiments, or server-configured intervals.
Design Decisions:
- Should clients back off if no updates are received?
- How to detect network loss and stop polling?
- What's the strategy when app resumes from background?

**Trade-offs**:

| Advantage                                 | Disadvantage                                                         |
| ----------------------------------------- | -------------------------------------------------------------------- |
| Easy to implement                         | Wasteful if there's no new data                                      |
| Works with existing REST APIs             | Adds network load and latency                                        |
| Compatible with all firewalls and proxies | Delayed updates compared to [[WebSockets]] or [[Push Notifications]] |
| Fine-grained control on polling strategy  | iOS background limitations can break continuity                      |

### Interview Talking Points
- Mention if you used polling as a **fallback to WebSockets**
- Describe how you adjusted polling intervals dynamically based on user activity
- Explain how you handled **de-duplicating** and **delta responses**
- Talk about power-aware decisions eg. avoid polling on low battery or in background
- Highlight how polling works predictably across poor networks, unlike persistent sockets

**Polling Interval Strategy**:

| App State           | Recommended Interval                  |
| ------------------- | ------------------------------------- |
| Foreground + active | 15-30 seconds                         |
| Background          | 5-15 minutes (via background fetch)   |
| Poor connectivity   | Exponential backoff (1s → 2s → 4s...) |
| Screen not in use   | Pause or poll rarely                  |

**Alternatives & Complements**:
- [[WebSockets]]: Real push, lower latency, higher complexity
- [[Server-Sent Events (SSE)]]: One-way push, not natively supported on iOS
- [[Push Notifications]]: Best for rare or important updates
- [[GraphQL]] Subscriptions: Similar to WebSockets but structured in schema
- [[HTTP Long-Polling]]: Polling variant where client holds the request until data is available