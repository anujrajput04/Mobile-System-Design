## GraphQL - Query Language for APIs
GraphQL is a **query language and runtime** for APIs developed by Facebook. Unlike REST (which exposes fixed endpoints), GraphQL allows **client to define the exact shape of the response** it needs.

**Key Features:**
- Single endpoint (`POST /graphql`)
- Strongly typed schema
- Client-driven queries
- Nested and related resources in one call
- Real-time support via subscriptions

**Example:**
```graphql
query {
	user(id: 1) {
		name
		posts {
			title
		}
	}
}
```

**Mobile Specific Considerations:**

| Concern         | GraphQL Advantage                                    | Relevance                                |
| --------------- | ---------------------------------------------------- | ---------------------------------------- |
| Bandwidth       | Asks only for needed fields (eg. skip large objects) | Ideal for slow mobile networks           |
| Latency         | Fetch related data in one request                    | Reduces roundtrips vs REST               |
| Versioning      | Avoids strict versioning with field-level evolution  | Good for long-lived mobile clients       |
| Over-fetching   | Avoided completely                                   | Saves battery and CPU                    |
| Offline caching | Requires custom strategies or 3rd party libraries    | Apollo iOS has built-in normalized cache |
| Error handling  | Partial errors can occur inside successful response  | Must be parsed differently than REST     |

**Example - Swift/iOS:**

Schema Code GEneration (via `.graphql` files and codegen):
```graphql
query GetUser {
	user(id: 1) {
		id
		name
		email
		posts {
			title
		}
	}
}
```

Calling the API:
```swift
ApolloClient.shared.fetch(query: GetUserQuery()) { result in
	switch result {
		case .success(let graphQLResult):
			if let user = graphQLResult.data?.user {
				print(user.name)
			}
		case .failure(let error):
			print("Error: \(error)")
	}
}
```

- Type-safe, auto-generated models
- Request batching, caching, subscriptions out of the box

**Cross-Platform & Backend Expectations:**
As an Architect:
- Define a **GraphQL schema** that documents all possible queries/mutations.
- Enforce **auth at resolver level** - some fields may be hidden unless authorized
- Handle **N+1 issues** on server using data loaders or joins
- Avoid overly complex queries from mobile by **query whitelisting or persisted queries**
- Use Apollo Federation or Hasura/PostGraphile if auto-generating from DB

Backend often includes:
- **Resolvers** mapped to DB calls/microservices
- **Batching and caching layers**
- **Rate limiting and query complexity analysis**

**Trade offs:**

| Advantage                              | Disadvantage                                     |
| -------------------------------------- | ------------------------------------------------ |
| Precise control over response shape    | More complex backend (resolvers, batching)       |
| Avoids over-fetching/under-fetching    | Difficult to cache at HTTP level (no URLs)       |
| Single endpoint simplifies API gateway | Apollo iOS adds build complexity and boilerplate |
| Useful for deep nested models          | Error model is less intuitive than REST          |

### Interview Talking Points
- Discuss how you implemented Apollo client caching (normalized or file-based)
- How you handled schema changes: did you do introspection or use persisted queries?
- Contrast REST's `/users/1/posts` with a GraphQL single query returning both.
- Share performance trade-offs and how you limited deeply nested queries on mobile
- Mention if you worked with query batching, persisted queries, or subscriptions

**Mobile Friendly GraphQL Enhancements:**
- **Persisted Queries**: Predefined server-side queries to save bandwidth
- **@defer directive**: Send critical fields first, heavy ones later.
- **Fragments**: Modular query parts for reuse across components
- **Directives** (`@include`, `@skip`): Control query fields conditionally

**When to prefer GraphQL over REST (Mobile Context):**
- Fetching nested data in one shot
- Customizing payloads per screen
- Supporting multiple clients with differing needs
- Avoiding hardcoded versioning
- Bandwidth-sensitive apps