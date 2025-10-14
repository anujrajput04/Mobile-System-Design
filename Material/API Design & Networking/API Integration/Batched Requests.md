Batched requests involve combining multiple API calls into single HTTP request to reduce network round-trips, improve performance, and save bandwidth, particularly useful in mobile environments where latency and power consumption matter.

**Why use Batched Requests in Mobile?**
- **Fewer round trips**: Lower latency and better performance
- **Reduced overhead**: One HTTP request header and RCP handshake instead of many
- **Atomicity**: Either all requests succeed or none, if the server supports it
- **Reduced battery usage**: Fewer wakeups and radio usage

**Implementation Patterns**:
Client Side:
- Prepare multiple operations to be executed
- Wrap them in a common structure (JSON array or custom envelope)
- Send one HTTP request
- Unpack the responses

Server Side:
- Parse each sub-request
- Execute them serially or concurrently
- Package the responses into single HTTP response

**Example**:
Request:
```json
[
	{
		"method": "GET",
		"path": "/users/123"
	},
	{
		"method": "POST",
		"path": "/events",
		"body": {
			"type": "click",
			"timestamp": "2025-07-15T10:00:00Z"
		}
	}
]
```

Response:
```json
[
	{
		"status": 200,
		"body": {
			"id": 123,
			"name": "Anuj"
		}
	},
	{
		"status": 201,
		"body": {
			"event_id": "xyz"
		}
	}
]
```

**iOS Implementation**:
**Manual batching** using:
- `Codable` structs
- `URLSession` with custom request construction
- Combine with multiple network calls into one payload

With **GraphQL**, batching is supported natively, Apollo Client supports this

**Android Implementation**:
With Retrofit:
- Native batching isn't directly supported
- Manual envelope creation and single endpoint can mimic batching

With GraphQL + Apollo or Relay, batching is possible

**Trade-offs**:

| Trade-Off               | Explanation                                                 |
| ----------------------- | ----------------------------------------------------------- |
| Error Handling          | Needs per-sub-response error structure                      |
| Debugging Complexity    | Harder to trace individual requests                         |
| Caching                 | Cache behavior is non-standard and may require custom logic |
| Latency vs Payload Size | Large batches = slower single response                      |
| Atomicity               | May or may not be supported depending on server logic       |

**When not to use**:
- If requests are independent and small
- When individual failures must be handled immediately
- For real-time or critical user interactions