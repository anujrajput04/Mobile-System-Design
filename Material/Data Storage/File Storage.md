## File Storage
File Storage refers to **storing raw files** (documents, images, videos, PDFs, logs, JSON exports, etc.) directly on the device’s **file system**, rather than in a database or key-value store.  
It’s ideal for **medium to large binary data** or **structured files** that don’t fit neatly into tables or key-value pairs.

### File Storage on iOS
#### 1. App Sandbox
Every app on iOS runs in its own **sandbox**, a private directory isolated from other apps.  
An app’s filesystem has a defined structure:
```
/AppName/
 ├── Documents/
 ├── Library/
 │    ├── Preferences/
 │    └── Caches/
 ├── tmp/
```

Each serves a specific purpose:

| Directory           | Purpose                                      | Backed Up? | Synced to iCloud? |
| ------------------- | -------------------------------------------- | ---------- | ----------------- |
| **Documents/**      | User data (files created by the user or app) | ✅ Yes      | ✅ Yes (optional)  |
| **Library/**        | App data not directly visible to user        | ✅ Yes      | ❌ No              |
| **Library/Caches/** | Temporary cache (can be purged by system)    | ❌ No       | ❌ No              |
| **tmp/**            | Temporary files, auto-deleted after reboot   | ❌ No       | ❌ No              |

#### 2. FileManager API

All file operations are done via `FileManager` in Swift.

Example:
```swift
let fileManager = FileManager.default
let docsURL = fileManager.urls(for: .documentDirectory, in: .userDomainMask).first!

let fileURL = docsURL.appendingPathComponent("data.json")
let content = "{ \"name\": \"Anuj\" }".data(using: .utf8)!

// Write
try? content.write(to: fileURL, options: .atomic)

// Read
if let data = try? Data(contentsOf: fileURL) {
    print(String(data: data, encoding: .utf8)!)
}
```

#### 3. Data Types Commonly Stored
- Media files (images, videos, audio recordings).
- PDFs, documents.
- Cached API responses (for offline mode).
- Logs, analytics batches.
- JSON exports or imports.

### Security and Privacy
- Files inside the sandbox are isolated — no other app can access them.
- For sensitive files, apply **Data Protection** attributes like:
    - `.complete`, `.completeUnlessOpen`, `.completeUntilFirstUserAuthentication`
- These integrate with the device’s passcode and Secure Enclave.

```swift
try data.write(to: fileURL, options: .completeFileProtection)
```

### Comparison with other Storage types
| Storage Type                   | Ideal For                           | Security    | Access Speed   | Size Handling |
| ------------------------------ | ----------------------------------- | ----------- | -------------- | ------------- |
| **UserDefaults**               | Simple prefs (booleans, strings)    | Low         | Fast           | Small         |
| **Keychain**                   | Credentials, tokens                 | Very High   | Medium         | Very small    |
| **SQLite / Core Data / Realm** | Structured data                     | Medium      | Fast           | Medium        |
| **File Storage**               | Binary data (images, docs, exports) | Medium High | Depends on I/O | Large         |

### iCloud & Shared Storage
- iOS allows file sync through **iCloud Drive** using the `NSUbiquitousContainer`.
- Use **FileProvider** framework if your app integrates with Files app.
- Large file sync can also be custom-implemented using backend APIs (S3, Firebase Storage, GCP).

### Android and Backend Equivalents
- **Android**:
    - Internal Storage (`getFilesDir()`, `getCacheDir()`) — private like iOS sandbox.
    - External Storage (`getExternalFilesDir()`) — user-visible, less secure.
- **Backend**:
    - File/Object Storage → AWS S3, Google Cloud Storage, Azure Blob.
    - Same design philosophy — store blobs, reference them via URLs or IDs in DB.

### Design Patterns in Mobile System Design
Typical file-handling layer in an app:

```
Network Service
   ↓
Cache Layer (File-based, for offline)
   ↓
Database Layer (metadata)
   ↓
UI Layer
```

Example:
- Downloaded media stored in `/Caches/Media/`
- Metadata stored in SQLite/Realm (name, size, timestamp).
- Cache cleanup managed by file size or time-based policy.


### Interview Insights
If asked in a system design interview:
- Highlight **sandbox model**, **directories**, and **lifecycle rules**.
- Mention **security layers** (file protection, sandboxing).
- Discuss **trade-offs vs databases**.
- Demonstrate awareness of **cross-platform and backend alignment** ie. file storage on mobile aligns conceptually with **object storage on backend**.