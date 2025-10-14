## gRPC - High Performance RPC Framework by Google
gRPC (Google Remote Procedure Call) is a **contract-first**, **binary protocol** built on HTTP/2. Unlike REST (resource-based) or GraphQL (query-based), gRPC is **function-call based** - you invoke methods on a remote server as if it were local.
It uses:
- **Protocol Buffers (Protobuf)** for message definition and serialization
- **HTTP/2** for multiplexed, low-latency communication
- **Stub code generation** for clients/servers in many languages

**Example `.proto` file**:
```proto
service UserService {
	rpc GetUser (UserRequest) return (UserResponse);
}

message UserRequest {
	int32 id = 1;
}

message UserResponse {
	int32 id = 1;
	string name = 2;
}
```

**Mobile-Specific Considerations**:

| Concern              | gRPC in Mobile Context                               |
| -------------------- | ---------------------------------------------------- |
| Payload Size         | Protobuf is much smaller than JSON                   |
| Battery Efficiency   | Efficient due to binary format + multiplexed HTTP/2  |
| Reconnection         | Needs custom logic on unstable mobile networks       |
| Background Execution | gRPC channels can close silently on suspend (iOS)    |
| Tooling              | Less native Swift/iOS support than REST/GraphQL      |
| Debuggability        | Harder to inspect payloads without decoding Protobuf |

Apple didn't provide native support for gRPC for a long time, so many developers use:
- [gRPC Swift](https://github.com/grpc/grpc-swift): Community driven, now official
- **Bridges** to REST (backend exposes REST facade over gRPC)
- **Kotlin Multiplatform Shared gRPC clients** (in cross-platform teams)

**Example - Swift/iOS**:
`GetUser.proto`
```proto
syntax = "proto3";

message GetUserRequest {
	int32 user_id = 1
}

message User {
	int32 id = 1;
	string = name = 2;
}

service UserService {
	rpc GetUser (GetUserRequest) returns (User);
}
```

Generate Swift client using `protoc`:
```bash
protoc user.proto \
	--swift_out=. \
	--grpc-swift_out=.
```

Swift Call:
```swift
let client = UserServiceClient(channel: ClientConnection.insecure(group: group)
								.connect(host: "localhost", port: 8080))

let request = GetUserRequest.with { $0.userID = 1 }

let call = client.getUser(request)

call.response.whenSuccess { user in
	print("Received user: \(user.name)" )
}
```

**Cross-Platform & Backend Expectations**:
From Architect view:
- Define **all RPCs in `.proto` files**, single source of truth for backend + all clients
- Ensure **backwards compatibility** with field numbering in Protobuf
- Backend infra needs:
	- **Load balancing**
	- **Service discovery**
	- **Observability (gRPC logs, metric, tracing)**
- Decide on **transport strategy**:
	- Native gRPC on Android
	- gRPC-Web for browsers
	- REST-proxy for iOS if gRPC is too heavy
- **Security**: Use `wss` or mTLS for encrypted channels

**Trade-offs**:

| Advantage                                    | Disadvantage                          |
| -------------------------------------------- | ------------------------------------- |
| Extremely efficient (binary, HTTP/2)         | Not human-readable - harder to debug  |
| Strong typing and schema enforcement         | iOS tooling still less mature         |
| Code generation reduces boilerplate          | Bigger binary size due to codegen     |
| Built-in support for streaming and deadlines | Needs more infra investment than REST |

### Interview Talking Points
- If you have worked in microservices: explain how mobile clients called gRPC via REST proxy
- Discuss trade-offs between gRPC and REST in terms of app startup time and cold boot performance
- Highlight your role in designing `.proto` files and negotiating field evolution with backend
- Mention how you handled reconnection, timeout, and retries for unstable networks.

**gRPC Streaming in Mobile**
gRPC supports:
- **Unary RPC**: Single request, single response
- **Server Streaming**: Server continuously sends data
- **Client Streaming**: Client sends stream of data
- **Bidirectional Streaming**: Both stream data concurrently
Excellent for:
- Real-time collaboration
- File uploads/downloads
- Log/event streaming
on iOS:
- Socket timeouts, app suspension, and connection instability make bidirectional streaming tricky
- Requires careful management of lifecycle and memory

**Use Cases**:
- High-throughput API with tight latency budget
- Internal app with shared contracts across Android/iOS/backend
- ~~Interop with legacy systems or browsers~~: needs REST proxy or gRPC-Web
- ~~Highly dynamic client payloads (eg. UI varies)~~: GraphQL is better