## HTTP Long-Polling
**HTTP Long Polling** is a technique where the client sends a request and the server **delays its response** until new data is available or a timeout occurs. As soon as the response is received, the client immediately **re-issues the request**
It improves over regular polling by:
- Reducing unnecessary empty responses
- Reacting more quickly to new events

![[HTTP Long-Polling.excalidraw]]

**Flow**:
1. Client sends a request: `GET /updates?since=timestamp`
2. Server **holds** the request open (eg. up to 30s)
3. If new data is available, it responds immediately, else time out
4. Client immediately sends another request (loop continues)

**Mobile Specific Considerations**:

| Concern               | Long Polling Impact on Mobile                     |
| --------------------- | ------------------------------------------------- |
| Responsiveness        | Near instant delivery of updates                  |
| Battery drain         | Better than short interval polling                |
| HTTP connection reuse | Requires persistent keep-alive HTTP/1.1 preferred |
| Background execution  | iOS may suspend the app so long polls get killed  |
| Network drop          | Must retry failed/aborted requests                |

**Mobile friendly tips**:
- Use `URLSession` with timeout configured appropriately
- Cancel previous request if app goes to background
- Add jitter/randomness to retry intervals to avoid thundering herd problem

**Example**:
```swift
func longPoll() {
	var request = URLRequest(url: URL(string: "https://api.example.com/updates")!)
	request.timeoutInterval = 30 // Server timeout for long poll
	URLSession.shared.dataTask(with: request) { data, response, error in
		if let data {
			// Process update
			print("Received update: \(data)")
		}
		// Regardless of success/failure, reinitiate poll
		longPoll()
	}.resume()
}
```
You should manage cancellation tokens and app lifecycle to avoid dangling calls

**Cross-Platform & Backend Expectations**:
As an Architect:
- Server must be able to **hold open HTTP requests**, ideally with async workers or event driven model (eg. Node.js, Go, or Python with async)
- Must support **timeouts** and **immediate flush** when data is available
- Backends often track client state with **session IDs** or **userIDs**
- Ensure request **deduplication** - clients may retry on timeout or error
- Backend should be able to scale under thousands of held connections, usually via reverse proxy like NGINX or using connection queueing

**Trade-offs**:

| Advantage                                 | Disadvantage                                 |
| ----------------------------------------- | -------------------------------------------- |
| Near instant updates, no constant polling | Still inefficient vs WebSockets or Push      |
| Works with firewalls/proxies              | Still high connection count on backend       |
| No special protocols required             | Server must be tuned for long-lived HTTP     |
| Easy to layer on top of REST API          | App suspension interrupts long poll silently |

### Interview Talking Points
- Discuss how you tuned polling interval and server timeout for balance
- Explain fallback strategy eg. regular polling if long polling fails
- Describe how app lifecycle and background suspension was handled
- Talk about how this was used in chat/message/read receipt/notification updates
- Mention when and why you chose long polling over WebSockets (eg. infra limitation or backend language constraints)

**Visual Sequence of Long Polling**:
```
Client → Server: GET /updates
		(Server holds request 25s.. no update)
Server → Client: 204 No Content (timeout)

Client → Server: GET /updates
		(Server holds request 8s.. new data)
Server → Client: 200 OK + data
Client → Server: GET /updates (re-initiated)
```

**When to use Long Polling in Mobile**:
- Near real-time chat/messages
- Live data in legacy stack
- ~~Network constrained devices~~: Use [[Push Notifications]] instead
- ~~Background only updates~~: Use [[Push Notifications]] instead