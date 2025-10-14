## Realm
Realm is a high performance, cross platform, object oriented database built specifically for mobile. Unlike SQLite, which is relational, Realm uses it's own storage engine and enables live objects, reactive queries, and thread local access
- Embedded database with object first modeling
- Live updates: Realm objects auto update across threads
- Built in sync: Realm Sync / Atlas Device Sync via MongoDB
- No need for object serialization/deserialization like SQLite
- Supports background thread writes/reads without blocking UI

### iOS implementation
- Schema is inferred from property types
- Thread confined objects
```swift
class Dog: Object {
	@Persisted var name: String
	@Persisted var age: Int
}

let realm = try! Realm()
try! realm.write {
	realm.add(Dog(name: "Jerry", age: 8))
}
```

### Android Equivalent
Realm is also available for Android with nearly identical API and behavior

### System Design Interview Relevance
Use Realm in scenarios like:
- Offline first apps with frequent read/write access
- Messaging/chat apps with live updates, local writes
- Data heavy apps with reactive UI eg. finance, health
- Apps needing easy to setup data sync across users/devices
- Apps needing cross platform consistency

### Realm Sync & Backend Coordination
With Realm Sync:
- Real time syncing with backend
- Conflict resolution strategies
- User authentication
- Fine grained permissions
- Offline write then sync pattern

If you're designing a **multi-device, multi-user** app (e.g., collaborative note-taking or health tracking), Realm Sync offloads a lot of the backend complexity

But be aware of vendor lock-in if you rely on Realm Sync + MongoDB cloud

### Schema Design and Migrations
- Define schema via models (classes)
- Add properties = non breaking change
- Remove/change type = requires migration
```swift
let config = Realm.Configuration(
	schemaVersion: 2,
	migrationBlock: { migration, oldSchemaVersion in
		// custom migration logic
	}
)
Realm.Configuration.defaultConfiguration = config
```

### Pitfalls & Gotchas
| Pitfall            | Explanation                                              |
| ------------------ | -------------------------------------------------------- |
| Thread confinement | Realm objects are not cross-thread safe                  |
| Lazy loading       | Accessing fields triggers I/O, watch UI impact           |
| Strong references  | Realm objects are auto updating, can cause retain cycles |
| Sync latency       | Realm Sync isn’t instant in flaky networks               |
| Vendor lock in     | Realm Sync ties you to MongoDB                           |
| Migrations         | Schema changes require careful migration handling        |
|                    |                                                          |

### When Not to Use Realm
- If you need SQL querying or ad hoc joins
- If app logic heavily depends on cross thread shared state
- If avoiding third party vendor lock in is critical
- If your backend doesn’t support Realm Sync and you need full control
### System Design Interview
>“For our chat app, we use Realm for storing messages and contacts locally. It provides a reactive and performant layer that updates the UI in real time. We use Realm Sync for backend coordination, so messages written locally are queued and synced automatically. The schema is versioned, and we handle migrations through the configuration block. Our background sync service handles attachments and uses the file system separately from Realm.”

### Interview Discussion
>**Interviewer:** "Your app needs to sync health tracking data across devices and show real time updates. How would you architect local storage?"

>**You:** "I'd use Realm to persist local data due to its reactive API, performance, and offline-first support. Realm Sync would handle two-way data syncing with conflict resolution. To prevent UI blocking, writes would happen in background queues. For sensitive data, we’d store tokens in Keychain, not in Realm."