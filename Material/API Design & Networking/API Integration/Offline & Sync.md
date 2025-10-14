## Offline & Sync
In real-world mobile environments, users expect apps to work even with intermittent or no connectivity. Offline capability + intelligent sync ensures:
- Data persistence when offline
- Automatic reconciliation when connectivity is restored
- Seamless user experience with minimum latency

---
### Conflict Resolution
Conflicts occurs when local data is modified while the server version has also changed.

**Types of Conflicts**:
- **Client vs Server**: Local update conflicts with remote update
- **Client vs Client**: Two devices modify the same object offline

**Conflict Resolution Strategies**:

| Strategy                             | Description                               | Pros                    | Cons                                             |
| ------------------------------------ | ----------------------------------------- | ----------------------- | ------------------------------------------------ |
| **Last Write Wins (LWW)**            | Use latest timestamp to decide            | Simple                  | Risk of overwriting meaningful changes           |
| **Sever Wins / Client Wins**         | Force preference to one side              | Deterministic           | May discard valid user data                      |
| **Manual Merge / User Intervention** | Prompt user to choose version             | Accurate                | UX friction, rarely used                         |
| **Field-level Merging**              | Merge non-conflicting fields individually | Balanced                | Complex to implement, especially for nested data |
| **Operational Transform / CRDT**     | Mathematical merging model                | Conflict-free by design | Heavy, often used in collaborative editing       |

**Implementation Tip**: Save both client and server versions, present a diff view (like Notes app merge UI)

---
### Queueing
Queueing is implemented to ensure reliable write operations when offline

**Concept**:
- Enqueue failed or deferred write operations (POST, PUT, DELETE) locally
- Persist them on disk (eg. Core Data, SQLite, Realm)
- Replay them in order when back online

**Typical Queue Metadata**:
- Operation type (POST/PUT/DELETE)
- Resource ID or path
- Payload (snapshot of change)
- Retry count / exponential backoff strategy
- Last attempted timestamp

**iOS Techniques**:
- Use CoreData or file-based queueing system
- URLSession background tasks for uploads
- Combine with `AppDelegate.applicationDidBecomeActive` or NWPathMonitor to trigger retry

---
### Background Retry
Automatically retry queued operations once conditions improve

**When to Retry**:
- Network is available
- App is in foreground or background task allowed
- Power state is sufficient (avoid retrying on low battery)

**Techniques**:

| Platform | Tools                                                                                            |
| -------- | ------------------------------------------------------------------------------------------------ |
| iOS      | `URLSession` with background config, background fetch, Combine, or timers in app lifecycle hooks |
| Android  | WorkManager, JobScheduler, AlarmManager                                                          |

---
### Data Integrity & Sync Strategy

| Strategy                 | Use Case                              |
| ------------------------ | ------------------------------------- |
| Pull on App Launch       | Light apps with small data sets       |
| Periodic Background Sync | Critical apps like notes, tasks       |
| Push + Delta Pull        | Real-time apps with selective syncing |
| Full Resync on Conflict  | Last resort fallback                  |

---
### Common Architectures

- **Single Source of Truth**: Local DB (Realm, CoreData) is always the source. Remote acts as sync provider
- **Optimistic UI Updates**: Assume operation will succeed, revert on failure
- **State Machine for Sync Statuses**: `pending → syncing → synced`, with failure and retry branches