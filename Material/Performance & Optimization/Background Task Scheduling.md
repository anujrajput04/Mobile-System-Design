## Background Task Scheduling
The purpose of Background Task Scheduling is to defer non-urgent work (eg. syncing data, refreshing feeds, uploading logs) to system-managed, battery-efficient time windows without affecting the UX.
Poor background task design = massive battery drain, OS throttling, task starvation, or user churn

### iOS Scheduling APIs
| API/Framework                            | Purpose                                  | Notes                                         |
| ---------------------------------------- | ---------------------------------------- | --------------------------------------------- |
| `BackgroundTasks` (BGTaskScheduler)      | Schedule deferrable tasks post-iOS 13    | System-coordinated, power-aware               |
| `URLSession (background)`                | File/network transfer in background      | OS handles retries, even if app is terminated |
| `UIApplication.beginBackgroundTask`      | Extend app time (~30s) during suspension | Limited; for short critical operations        |
| `Push Notification w/ content-available` | Silent push → app wakes to sync          | Needs user opt-in; throttled if abused        |
| `CoreMotion`, `HealthKit`, `Location`    | Background sensors                       | Allowed under special entitlements only       |
### BGTaskScheduler
- `BGAppRefreshTask` - light background refresh
- `BGProcessingTask` - heavier, long-running work (eg. uploads)

**Registration:**
```swift
BGTaskScheduler.shared.register(forTaskWithIdentifier: "com.company.app.refresh", using: nil) { task in 
	self.handleRefresh(task: task as! BGAppRefreshTask)
}
```

**Scheduling:**
```swift
let request = BGAppRefreshTaskRequest(identifier: "com.company.app.refresh")
request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60) // 15 min
try? BGTaskScheduler.shared.submit(request)
```

**Handling:**
```swift
func handleRequest(task: BGAppRefreshTask) {
	scheduleNextRefresh() // Always reschedule
	task.expirationHandler = { cancelAllWork() }
	performBackgroundWork {
		task.setTaskCompleted(success: true)
	}
}
```

#### Rules
- Always reschedule after execution
- Finish within ~30s or OS penalizes frequency
- Must declare task identifiers in Info.plist
- Requires background mode entitlement (fetch, processing, etc)

---
### When to use what

| Use Case                        | Best API                                      |
| ------------------------------- | --------------------------------------------- |
| Background sync every few hours | `BGAppRefreshTask`                            |
| Uploading large files           | `BGProcessingTask` or background `URLSession` |
| Data fetch on push              | Silent Push + `content-available: 1`          |
| Short task during suspend       | `beginBackgroundTask(expirationHandler:)`     |
| Critical logs on crash          | Combine `beginBackgroundTask` + disk flush    |
### Scheduling Strategy Design
- Break work into **idempotent chunks** → no issues if interrupted
- Use caching + queuing for retryable background operations
- Limit resource usage: Avoid CPU/GPU during background unless essential
- Think **graceful degradation** if task fails, never block the foreground logic

### Android Comparison

| Task                  | Android Equivalent                  | Differences                               |
| --------------------- | ----------------------------------- | ----------------------------------------- |
| BGAppRefreshTask      | WorkManager (Periodic Work)         | Android gives more control + retry logic  |
| BGProcessingTask      | WorkManager (Long running work)     | Same purpose, Android is more flexible    |
| Silent Push           | Firebase Cloud Messaging            | Android supports exact time scheduling    |
| background URLSession | JobScheduler / WorkManager + OkHttp | iOS more hands-off, Android dev owns flow |

---
### Backend Considerations
- Support incremental sync endpoints
- Respond fast to sync calls within time limits
- Enable stateless retries
- Allow partial data pull (pagination, delta tokens)
Poor backend design = sync tasks fails silently or get throttled by iOS

### Metrics & Observability

| Metric                 | Ideal                     |
| ---------------------- | ------------------------- |
| Task success rate      | >95%                      |
| Background retry count | <3 average                |
| Energy cost per task   | Low/Moderate (Energy Log) |
| Average task duration  | <30s                      |
| Task frequency per day | 2-4                       |

### Interview Points
>"We schedule our background work using BGTaskScheduler, breaking it into idempotent, short lived chunks that complete within system defined windows. All data syncs are backend paginated and retry safe. We wrap background uploads with URLSession background configuration for persistence across reboots. We monitor task success and energy usage via Metric Kit, and aggressively reschedule tasks only when prior ones succeed, avoiding spam that leads to OS throttling"

### Gotchas
- Task not running: Check Info.plist, entitlements, deadlines
- Silent push not triggering: APNs config, check `content-available`
- Task terminated early: Set `expirationHandler`, monitor duration
- No retry on crash: Persist retryable work via local DB