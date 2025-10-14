## Background API Calls
These are network requests that can continue running even if the app is suspended, terminated, or moved to the background. They are critical for handling large uploads/downloads or periodic syncs while respecting system power and memory constraints.

---
### iOS: `URLSession` - Background Configuration
Apple provides a special background configuration of `URLSession`

```swift
let config = URLSessionConfiguration.background(withIdentifier: "com.youapp.background")
let session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
```

**Use Cases**:
- Uploading photos/videos
- Downloading large files
- Syncing data when the app is in the background or terminated

**Behaviour**:
- The system takes control of the request after it's handed off
- Even if your app is killed, iOS will complete the request
- You get a callback in the app delegate method

```swift
func application(_ application: UIApplication, handleEventsForBackgroundURLSession identifier: String, completionHandler: @escaping () -> Void)
```
>iOS will relaunch the app silently and call this method

**Gotchas**:
- Only works with HTTP/HTTPS
- No streaming or WebSocket support
- No support for regular requests (like JSON APIs), best suited for file uploads/downloads
- Needs proper configuration of `Info.plist` background modes

___
### Android: `WorkManager` & `JobScheduler`
**WorkManager** (Recommended):
WorkManager is the modern Android library to schedule deferrable background tasks.
```kotlin
val workRequest = OneTimeWorkRequestBuilder<SyncWorker>().build()
WorkManager.getInstance(context).enqueue(workRequest)
```

- Works across API levels
- Handles retries, constraints (network, charging, idle), and persistence
- Ideal for background syncing, API fetches

**JobScheduler** (Legacy):
- Introduced in API 21+
- Schedules background jobs with OS constraints
- Does not survive app uninstall or clear data
- Mostly superseded by WorkManager