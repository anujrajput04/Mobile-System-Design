## API Evolution
Maintaining API stability over time is critical, especially when multiple client versions (iOS, Android, Web) interact with the same backend. Any change in the API must not break existing clients

#### Why API Evolution is needed
- Requirements change: new features, deprecations
- Clients are not all updated at once
- Public APIs must not break for external consumers

### Versioning Strategies
#### URI Versioning
```http
GET /v1/users/123
GET /v2/users/123
```

**Pros**: Easy to cache, clear for clients
**Cons**: Might break HATEOAS or hypermedia driven clients

#### Header Versioning
```http
GET /users/123
Headers:
	Accept: application/vnd.company.app-v2+json
```

**Pros**: Cleaner URLs, more RESTful
**Cons**: Not visible in browser/devtools directly

#### Query Param Versioning
```http
GET /users/123?version=2
```

**Pros**: Quick to implement
**Cons**: Often abused, poor caching

### Backward Compatibility
**Do**:
- Add new fields to a response
- Add optional parameters to request payloads
- Change internal logic without affecting interface
**Don't**:
- Remove fields used by clients
- Change the meaning/type of existing fields
- Add required parameters unless defaulted

---
### Safe Evolution Techniques
#### 1. Use Nullable Fields for Additions
```json
{
	"name": "Anuj",
	"nickname": null
}
```
#### 2. Deprecation Strategy
- Document fields or endpoints as deprecated
- Support them for a fixed time before removal
- Use monitoring to detect usage before killing them
#### 3. Graceful Fallbacks in Clients
- In iOS/Android, use decoding with default values:
```swift
struct User: Codable {
	let name: String
	let nickname: String? // optional for backward compatibility
}
```

---
### Tooling Support
- iOS Codable is tolerant to missing/new fields by default
- Protobuf/gRPC are strongly versioned and backward compatible when used correctly
- GraphQL offers field-level versioning, clients can stop querying deprecated fields

#### Considerations in Secure APIs
- Tokens, OAuth scopes, and permissions must also evolve safely
- If new scopes are added, they shouldn't break existing tokens

---
### Example
Imagine you're evolving this:

**v1 Response**:
```json
{
	"user_id": 123,
	"name": "Anuj"
}
```

**v2 Response**:
```json
{
	"id": 123,
	"full_name": "Anuj R",
	"nickname": "Anu"
}
```

**Safe evolution**:
- Keep `user_id` for backward compatibility
- Add new fields, mark old ones deprecated

| Change                 | Backward Compatible? |
| ---------------------- | -------------------- |
| Add new optional field | ✅ Yes                |
| Remove existing field  | ❌ No                 |
| Change field name      | ❌ No                 |
| Add enum value         | ✅ Yes (if handled)   |
| Change enum meaning    | ❌ No                 |
