## Serialization & Parsing
**Serialization** = Convert in-memory data to transport format (eg. JSON, XML)
**Deserialization** (Parsing) = Convert received JSON/XML into usable in-memory models

In API clients, this often means mapping JSON → Model → App logic

---
### iOS
#### Codable:
`Codable` is a protocol that combines `Encodable` + `Decodeable`. Adopt his to serialize or deserialize your models.

```swift
struct User: Codable {
	let id: Int
	let name: String
}
```

#### Decoding: JSON → Model
```swift
let data = ... // from URLSession
let user = try JSONDecoder().decode(User.self, from: data)
```

#### Encoding: Model → JSON
```swift
let jsonData = try JSONEncoder().encode(user)
```

#### JSONDecoder / JSONEncoder Customization

| Use case                | API                                                   |
| ----------------------- | ----------------------------------------------------- |
| Change date format      | `decoder.dateDecodingStrategy`                        |
| Convert snake_case keys | `decoder.keyDecodingStrategy = .convertFromSnakeCase` |
| ISO8601 date parsing    | `.dateDecodingStrategy = .iso8601`                    |
| Custom decoding         | Conform to `init(from decoder: Decoder)` manually     |

```swift
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .formatted(DateFormatter())
decoder.keyDecodingStrategy = .convertFromSnakeCase
```

#### Nested JSON / Optional Fields Handling
- For deeply nested fields, use `CodingKeys` and `nestedContainer`.
- Use optional properties with defaults for resilience.
- Graceful fallback on parsing failure using `try?`, custom wrappers.

#### DTO (Data Transfer Object) ↔ Domain Model Mappers
Avoid leaking API structure into your core logic.
```swift
struct UserDTO: Codable {
    let id: Int
    let name: String
}

struct User {
    let id: Int
    let displayName: String
}

extension User {
    init(dto: UserDTO) {
        self.id = dto.id
        self.displayName = dto.name
    }
}
```

---
### Android
#### Gson
```kotlin
val gson = Gson() val user = gson.fromJson(jsonString, User::class.java)
```
- Flexible, but less strict. Fields not found in JSON → default to `null`.

#### Moshi (Preferred)
```kotlin
val moshi = Moshi.Builder().build() val adapter = moshi.adapter(User::class.java) val user = adapter.fromJson(json)
```
- Stricter, better for Kotlin.
- Supports Kotlin’s `nullability`, sealed classes, and custom adapters.

#### DTO ↔ Domain Mapping (Kotlin)
```kotlin
data class UserDTO(val id: Int, val name: String) data class User(val id: Int, val displayName: String)  fun UserDTO.toDomain() = User(id, name)
```
- Use extension functions or Mapper interfaces.
- Prefer sealed hierarchies and data classes in domain layer for type safety.

---

### Key Considerations across Platforms
|Concern|Handling|
|---|---|
|Field name mismatches|Custom keys / converters|
|Optional/missing fields|Use default values / nullable types|
|Enum safety|Add fallback `.unknown` case|
|Versioned APIs|Create new DTO structs, don’t reuse