## CPU Usage
### What consumes CPU?
- Heavy computations (JSON parsing, image decoding, layout, encryption)
- Main thread blocking operations
- Frequent timers, polling
- Misuse of Combine/async/closures
- Core Animation and layout reflows
- Poor caching → recomputation

### How to measure CPU
Xcode Instruments → Time Profiler
- Samples stack traces during execution
- Pinpoints time spent per function
- Detects hot paths and main thread blocking
Debug Navigator (live in Xcode)
- View live CPU % usage by threads and modules

Common CPU Optimization practices
- Offload work to background queues
- Avoid syncing too much UI work on main thread
- Use efficient parsers (eg. JSONDecoder over manual parsing)
- Precompute layout using async rendering (eg. TextKit, diffable data sources)
- Use value types (structs) and lazy properties for expensive objects

---

## Battery
### What drains Battery?
- CPU Overuse: Tight loops, unnecessary animations
- Network Overuse: Polling, large payloads, chat, image loading
- Location Services: High accuracy GPS, continuous location updates
- Background Execution: Long running background tasks
- Sensors & Peripherals: Camera, Bluetooth, Accelerometer
- UI Overdraw: Unoptimized rendering cauing GPU & CPU churn

#### iOS-Specific Battery Instruments
- Instruments → Energy Log
    - Captures CPU wakeups, GPU usage, thermal state
    - Detects excessive backgrounding, high energy operations
- MetricKit
    - Collects crash, hang, energy, and CPU diagnostics from real devices
    - Aggregated automatically, low overhead

---
### Real World Code Smells
- App frame drops, jitter: Heavy work on main thread
- Fast battery drain when idle: Background timer firing, location tracking, push polling
- Heat on device during video call: Re-encoding, inefficient codec use, high resolution streaming
- UI freezes on data load: Sync decoding/parsing, image resizing on main thread

### Best Practices: CPU & Battery Optimization
##### CPU
- Profile with **Time Profiler** and optimize hot paths
- Avoid dispatching large closures to main thread
- Prefer lightweight view trees and **cell prefetching** in table/collection views
##### Battery
- Coalesce timers and reduce their frequency
- Prefer push over polling for notifications
- Use `deferred` location accuracy unless high precision is essential
- Release camera/sensor resources immediately after use
- Avoid overuse of background fetch

### Architecture Design Level Strategies
**Example 1: Chat App**
Bad: Polling every 2s for new messages
Good: WebSockets, push notifications, and on-demand sync

**Example 2: Image heavy App**
Bad: Loading full resolution images from network into `UIImageView`
Good: Downscale server side → lazy load → cache with NSCache

**Example 3: Fitness Tracker**
Bad: Real time GPS logging every second → drains battery
Good: Log coarse updates unless moving fast → use `desiredAccuracy`, `distanceFilter`

## iOS vs Android vs Backend
| Category        | iOS                              | Android                                     | Backend                           |
| --------------- | -------------------------------- | ------------------------------------------- | --------------------------------- |
| CPU Control     | GCD, Instruments                 | HandlerThreads, Executors, WorkManager      | Threads, Task schedulers          |
| Energy Profiler | Energy Log, MetricKit            | Battery Historian, Profiler                 | N/A (runs on powered infra)       |
| Wake Control    | Background Tasks, location modes | AlarmManager, WorkManager, Doze Modes       | N/A                               |
| Optimization    | Value types, main thread limits  | Avoid GC churn, LruCache, lifecycle cleanup | Efficient serialization, batching |

### Interview Points
>"We routinely profile CPU usage using Time Profiler, identify hot paths like heavy JSON parsing or layout work on the main thread, and migrate those to background queues. We use MetricKit to track energy diagnostics on live devices. On the battery side, we optimize location updates, avoid polling via server driven sync, and cache images at multiple tiers to reduce recomputation and I/O. Our CI runs performance baselines and flags regressions in both frame jank and CPU peaks"