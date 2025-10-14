## Server-Sent Events (SSE)
**Server-Sent Events** is a **one-way streaming protocol** where the server pushes data to the client over a **single long-lived HTTP connection**. Unlike WebSockets, which are bidirectional, SSE is **strictly server-to-client**

It uses:
- `Content-Type: text/event-stream`
- A single `GET` request from client
- Auto-reconnection support (built-in on the client side)

Sample message from server:
```yaml
data: {"message": "New chat received"}
id: 12345
event: message
```

**Mobile-Specific Considerations**:

| Concern              | SSE on iOS                                       |
| -------------------- | ------------------------------------------------ |
| Directionality       | One-way (server → client only)                   |
| Protocol             | HTTP/1.1. not HTTP/2, no multiplexing            |
| Background execution | Connection dies when app goes inactive/suspended |
| Battery + network    | More efficient than polling but worse than push  |
| Native support       | No native SSE support in iOS SDK                 |
| Tooling              | Must use 3rd-party libs or manually parse stream |

You have to **manually implement** an `EventSource`-like mechanism in iOS using `URLSession` and parsing line-by-line streamed text.

**Example - Usage**:
```swift
class SSEClient {
    var task: URLSessionDataTask?

    func startListening() {
        let url = URL(string: "https://example.com/events")!
        var request = URLRequest(url: url)
        request.setValue("text/event-stream", forHTTPHeaderField: "Accept")

        task = URLSession.shared.dataTask(with: request) { data, _, _ in
            guard let data = data else { return }
            let text = String(data: data, encoding: .utf8) ?? ""
            print("Received SSE: \(text)")
            // Manual parsing of 'data:', 'event:', etc.
        }
        task?.resume()
    }

    func stopListening() {
        task?.cancel()
    }
}
```

**Cross-Platform & Backend Expectations**:
On the backend:
- SSE works well with **event-driven architectures** (Node.js, Go, etc).
- Must support:
    - **Proper `text/event-stream` formatting**
    - **HTTP keep-alive**
    - **Client reconnections** using `Last-Event-ID` header
    - **Timeouts and heartbeats**

On mobile clients:
- SSE is ideal for use cases like:
    - Real-time status updates (eg. order tracking)
    - Feed refreshes
    - Alerts/notifications (non-critical)

But **not** ideal when:
- You need bidirectional communication    
- App must operate in background
- There’s poor connectivity

**Trade-offs**:

| Advantage                           | Disadvantage                                 |
| ----------------------------------- | -------------------------------------------- |
| Simple server-push over HTTP        | iOS lacks native support                     |
| Auto-reconnect and message ordering | Unidirectional only (no client → server)     |
| Text-based, human-readable          | Higher bandwidth vs Protobuf/WebSocket       |
| Built-in in modern browsers         | iOS SDKs require 3rd-party or manual parsing |

### Interview Talking Points
- Explain how you implemented SSE client-side with URLSession and parsing logic
- Mention its use in situations where WebSockets were overkill
- Highlight where SSE fit into a microservices based backend: stream from Kafka → SSE
- Talk about fallback strategies to polling when SSE failed (eg. due to proxy blocking)

**SSE Message Format**:
```yaml
id: 42
event: update
data: {"status":"shipped","orderId":1234}
```

**When to Use SSE in Mobile**:
- One-way notifications or status 
- ~~Bidirectional chat~~
- Low-power real-time feed
- ~~App in background (iOS)~~
- Streaming logs or progress