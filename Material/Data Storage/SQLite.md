## SQLite
SQLite is a lightweight, embedded, relational database engine. It's serverless, runs in-process with the app, and stores data in a single `.sqlite` file on disk

Use Cases:
- Local caching of backend data
- Structured data with relationships (1:1, 1:N, etc)
- Offline-first apps (especially for write-heavy use cases)
- Local persistence of dynamic data (eg. messages, content feeds, logs)

### iOS Specific Usage
Direct SQLite, via C APIs or third party wrappers like FMDB or SQLite.swift
- Offers low level access and full SQL syntax
- Good for performance-critical or schema-controlled use cases
- Manual schema creation, migrations, queries, etc
```swift
import SQLite3

var db: OpaquePointer?
if sqlite3_open(path, &db) == SQLITE_OK {
	// ready to use `db`
}
```

Not Recommended: Writing raw SQL directly in production apps unless there's a compelling performance or portability need

### iOS Alternatives
| Purpose                 | Use              | Apple Tech             | Third-Party |
| ----------------------- | ---------------- | ---------------------- | ----------- |
| Object graph management | Domain models    | Core Data              | Realm       |
| Lightweight relational  | Structured cache | SQLite, SQLite.swift   | FMDB        |
| Secure, custom logic    | Custom solutions | Combine + SQLite, etc. | GRDB.swift  |

### Android Equivalent
| Feature    | Android              | iOS                 |
| ---------- | -------------------- | ------------------- |
| ORM layer  | Room                 | Core Data / GRDB    |
| Raw access | SQLiteOpenHelper     | SQLite.swift / FMDB |
| Migrations | Room Auto Migrations | Manual in iOS       |
Room is an abstraction over SQLite and includes:
- Compile time validation of SQL
- Auto migrations
- LiveData / Flow support
- Cleaner DAO pattern

### System Design Interview Relevance
Common Scenarios:
- Design a local cache system for a paginated news feed
- Offline support for messaging apps
- Storing local logs or analytics events before upload
- Local data joins and filtering

As an Architect, should be aware of:
- Schema evolution strategies (migrations)
- Read/write performance optimizations (indexing, batching)
- Sync strategy with backend
- Tradeoffs between local DB vs remote query

### Core Concepts
| Concept                   | Design Relevance                                                            |
| ------------------------- | --------------------------------------------------------------------------- |
| Schema Design             | Normalize data to reduce redundancy, but consider denormalization for reads |
| Indexing                  | Index frquently queried fields, but know the write costs                    |
| Transactions              | Use `BEGIN TRANSACTION` to batch writes and reduce I/O                      |
| Migrations                | Plan schema evolution via version numbers                                   |
| Conflict Resolution       | Required for syncing with server data                                       |
| Write-Ahead Logging (WAL) | Increase write performance                                                  |
| Size Limits               | Single DB can grow to GBs, but avoid storing blobs                          |
| Threading                 | DB access should off the main thread                                        |

### Real Life Use Case
>"In our app, we use SQLite via GRDB for storing offline product catalog and user bookmarks. We batch write server responses using transactions, and we maintain a `last_synced_at` per table. On app launch, if the schema version doesn't match, we trigger migrations. Data sync with backend is eventually consistent, and conflicts are resolved using `last_updated_at` timestamps"

### Pitfalls & Anti-Patterns
- Using SQLite for key-value storage is overkill
- Storing large blobs. Images/videos should be stored on file system and store path in DB
- Blocking main thread with DB operations
- Not handling migrations, the app crashes on schema change
- Ignoring schema versioning, especially dangerous in multi-device scenarios

### When NOT to use SQLite
|Alternative|When to Prefer It|
|---|---|
|Core Data|Complex object graph, iCloud sync|
|Realm|Reactivity, live updates, easy setup|
|FileManager|Storing unstructured data or blobs|
|Key-Value Store|Simple preferences/settings|
|Backend only|When you don’t need offline persistence|
### Sync strategy with Backend
- Sync via pull: server → SQLite
- Sync via push: local changes → server
- Sync conflict resolution:
	- Timestamp based (last write wins)
	- Versioned records (incremental)
	- Operational transforms (for collaborative apps)
- Upload changes:
	- Store "dirty" records with flags (`needsUpload = true`)
	- Batch upload when connected
	- Mark as synced when confirmed

### Example System Design Prompt
"Design an offline-first mobile app that allows users to view and annotate PDFs. Annotations must sync back to the server."

Expected SQLite answer elements:
- Table for PDFs, annotations, users
- Local save of PDFs in file system, DB stores metadata
- Annotation writes stored locally, marked dirty
- Periodic sync service to upload annotations
- Use transactions for batched insert/update
- Use schema versioning and migrate as needed