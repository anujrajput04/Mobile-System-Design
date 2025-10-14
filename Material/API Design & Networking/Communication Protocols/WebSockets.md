WebSockets provide a **persistent**, **full-duplex connection** between client and server over a single TCP connection. Unlike REST or GraphQL (which are request-response), WebSockets allow **continuous data exchange** either side can send messages anytime

**Protocol:** `ws://` or `wss://` (secure)
**Lifecycle**:
- Handshake (via HTTP Upgrade)
- Persistent connection stays open
- Messages sent as frames (text/binary)

**Mobile Specific Considerations**:
- Battery & CPU - Persistent connection drains more than REST/GraphQL
- Network Fluctuations - Requires reconnection logic
- Background Limitations - iOS suspends sockets when app goes inactive
- Real-time Requirements - Excellent for live features
- Throttling / Rate Limits - Can overwhelm server without careful handling
- Multiplexing - One socket → many message types (chat, presence, etc)

**Example:**
iOS doesn't offer first-party WebSocket APIs until iOS 13+ via `URLSessionWebSocketTask`
```swift
/// Using `URLSessionWebSocketTask`
let url = URL(string: "wss://example.com/socket")
let task = URLSession.shared.webSocketTask(with: url)
task.resume()

// Receiving
func receiveMessage() {
	task.receive { result in
		switch result {
			case .success(let message):
				switch message {
					case .string(let text):
						print("Received: \(text)")
					case .data(let data):
						print("Received binary: \(data)")
				}
				receiveMessage() // Recurse to continue listening
			case .failure(let error):
				print("WebSocket error: \(error)")
		}
	}
}

// Sending
func sendMessage(_ text: String) {
	task.send(.string(text)) { error in
		if let error {
			print("Send error: \(error)")
		}
	}
}
```

- Works with Combine, async/await
- Handles pings, closes, and heartbeats

**Cross-Platform & Backend Expectations:**
Architecture:
- Define **message structure**: JSON-based envelope with `type`, `payload`, `timestamp`, etc
```json
{
	"type": "chat_message",
	"payload": {
		"sender": "Anuj",
		"text": "Hi"
	},
	"timestamp": "2025-07-14T12:04:19+00:00"
}
```

Backend must support:
- Connection management (scale to many devices)
- Authentication (eg. via JWT during handshake)
- Heartbeats & timeouts
- Message routing & deduplication
- Fallback strategy (eg. polling)

For mobile reliability, use libraries with:
- Auto-reconnect with exponential backoff
- Message buffering during disconnects
- Queuing and replays

**Trade-offs**:

| Advantage                                | Disadvantage                                |
| ---------------------------------------- | ------------------------------------------- |
| Real-time updates (chat, sports, prices) | Harder to debug than REST                   |
| Low latency, no polling overhead         | iOS limits background execution for sockets |
| Full duplex, server push capable         | Needs custom protocol, no caching           |
| Efficient for frequent data updates      | No built-in delivery guarantees (use ACKs)  |

### Interview Talking Points
- Mention real-time use cases you worked on (chat, live status, collaborative UI)
- Discuss how you managed reconnection and message loss on mobile networks
- Describe how you avoided battery drain (eg. suspend socket in background)
- Talk about how you isolated message types on the socket channel
- Show how you validated the server using SSL pinning or token headers

**Use Cases in Mobile**:
- Chat / messaging
- Live collaboration (notes)
- Real-time game updates
- ~~Notifications~~ - Use Push Notifications instead
- ~~One-way data updates~~ - Consider Server sent events or polling

**Mobile Focused Enhancements**:
- **Throttling**: Rate-limit UI updates (eg. use `Combine.throttle` or `debounce`)
- **Deduplication**: Precent repeated rendering on same data
- **Message persistence**: Use local DB (Realm / CoreData) to buffer state
- **Connection-aware UI**: Show disconnected state, retry button, offline badges