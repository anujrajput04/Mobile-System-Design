## Key-Value Storage
Key-Value Storage is a lightweight data persistence method where data is stored as a dictionary: `key → value` 
This is typically used for small pieces of information, such as:
- User preferences (dark mode, locale)
- App settings (feature toggles)
- Flags (first launch, walkthrough completed)
- Session tokens
It's not meant for structured or relational data

### iOS: UserDefaults
UserDefaults, also NSUserDefaults
- Persistence: Data is persisted across app launches
- Threading: Not thread safe by default, synchronize if needed
- Data types: Supports `Int`, `Double`, `Bool`, `String`, `Array`, `Dictionary`, `Data`, and `URL`
- Backed by: A `.plist` file under the hood
- Use Cases: Theme preference, user login status, cached simple settings
```swift
// Saving
UserDefaults.standard.set(true, forKey: "hasSeenOnboarding")

// Retrieving
let seen = UserDefaults.standard.bool(forKey: "hasSeenOnboarding")
```

Not for:
- Sensitive data (like tokens, passwords)
- Large datasets
- Structured/relational storage

### Android: SharedPreferences
| Purpose            | iOS                | Android                                  |
| ------------------ | ------------------ | ---------------------------------------- |
| Non secure storage | `UserDefaults`     | `SharedPreferences`                      |
| Secure storage     | `Keychain`         | `EncryptedSharedPreferences` or Keystore |
| Complex/structured | Core Data / SQLite | Room / SQLite                            |

### System Design Relevance
Interview Usage Scenarios:
- Storing feature flags fetched from backend (eg. A/B tests)
- Persisting auth/session state (`UserDefaults` + `Keychain`)
- Bootstrapping app from cached data
- Offline first toggle cache
- Syncing lightweight settings across devices (via iCloud / Firebase Remote Config)
Considerations:
- **Size limits:** Don’t abuse UserDefaults with large blobs or images
- **Security:** Anything auth related should go to Keychain
- **Consistency:** Data is not transactional, don’t rely on atomicity across multiple keys
- **Access Patterns:** Reads are cheap, but avoid writing repeatedly in tight loops

### Best Practices
- Group keys logically using enums or constants
- Clear out keys on logout or reset
- Use app versioning to invalidate old settings (eg. migration strategies)
- Don't store derived state (only source of truth)
### Common Pitfalls
- Overusing for storage of dynamic data (eg. cache)
- Using it for sensitive data
- Storing custom types via `NSCoding` without considering future compatibility
- Not synchronizing changes when accessing from multiple threads

### Comparison
| Feature               | UserDefaults (iOS)   | SharedPreferences (Android) | Keychain (iOS) | EncryptedSharedPreferences (Android) |
| --------------------- | -------------------- | --------------------------- | -------------- | ------------------------------------ |
| Suitable for settings | ✅                    | ✅                           | ❌              | ❌                                    |
| Suitable for secrets  | ❌                    | ❌                           | ✅              | ✅                                    |
| Auto-synced to cloud  | ☁️ iCloud (optional) | ❌                           | ❌              | ❌                                    |
| Thread safety         | ⚠️ Not guaranteed    | ⚠️ Not guaranteed           | ✅              | ✅                                    |
| Backup & restore      | ✅                    | ✅                           | ✅              | ✅                                    |

### Interview Considerations
> "For small app settings and feature toggles, we use Key-Value storage. On iOS, that’s UserDefaults for non sensitive values, and Keychain for secure values like tokens. We make sure to invalidate and rehydrate on version upgrade and logout. To support sync across devices, we use iCloud Key-Value Store for some preferences. For Android, we mirror this via SharedPreferences and EncryptedSharedPreferences."