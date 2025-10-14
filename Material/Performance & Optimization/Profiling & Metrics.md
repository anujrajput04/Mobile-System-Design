## Profiling
Profiling is **measuring the performance** of the app or system in a structured way. It tells you where the app spends time, allocates memory, blocks threads, or draws too many frames.

#### Goals
- Find bottlenecks (CPU, memory, I/O, GPU, startup)
- Detect inefficiencies (battery drain, long frame times, unresponsiveness)
- Quantify performance regressions during CI/CD

#### Tools

| Tool                              | What it Profiles                   | When to use                         |
| --------------------------------- | ---------------------------------- | ----------------------------------- |
| Xcode Instruments                 | CPU, Memory, Time, Energy, Threads | Deep dives during dev/test          |
| Time Profiler                     | Function-level CPU cost            | Identify slow methods               |
| Allocations                       | Object creation, heap usage        | Memory bloat, leaks                 |
| Leaks                             | Retain cycles, unreleased memory   | Leak detection                      |
| Activity Monitor                  | CPU, Memory, Network, Energy live  | Run-time visibility                 |
| os_signpost / Logger              | Custom trace events, durations     | App-specific metric instrumentation |
| Instruments' FPS / Core Animation | Frame rendering time, jank         | Scrolling or animation performance  |

#### Metrics You Should Track
##### Core App Metrics

| Category    | Metric                                            | Thresholds (Typical)           |
| ----------- | ------------------------------------------------- | ------------------------------ |
| Launch Time | Cold, Warm, Resume                                | Cold < 2s, Warm < 1s           |
| Memory      | Peak RAM, Retained objects                        | < 400MB ideally                |
| CPU Usage   | % of main thread usage                            | < 50% sustained on main thread |
| GPU/Frames  | FPS, dropped/janky frames                         | 60 FPS; drop < 3% per session  |
| Battery     | mW consumption, background CPU                    | < 5% drain per hour            |
| Network     | Payload side, number of calls, time to first byte | Use HTTP/2 or batching         |
| Storage     | On-disk cache size, logs, temp files              | Clean up > 200MB cache         |

##### Custom Business Metrics
Can be tracked via os_signpost, Logger, Firebase, etc
- Time to first content (TTFC)
- Screen load time
- Success rate of screen render
- Scroll smoothness (FPS)
- Crash-free sessions
- Failure rate of key APIs
- Retention linked to performance (eg. slower app = less usage)

#### Logging & Metrics Reporting Frameworks

| Need                    | Tool                                   | Notes                               |
| ----------------------- | -------------------------------------- | ----------------------------------- |
| Crash + session reports | Firebase Crashlytics, Sentry, Instabug | Mandatory for release builds        |
| Performance tracing     | Firebase Performance, Sentry           | End-to-end trace timings            |
| Custom metrics          | `os_signpost`, Logger, OpenTelemetry   | Apple's preferred low level logging |
| User experience metrics | Mixpanel, Amplitude, Segment           | Link user actions with performance  |
| Real time diagnostics   | MetricKit                              | Power, crash, hangs, etc            |
| Network inspection      | Proxyman, Charles Proxy, Flex          | Inspect and replay network calls    |
### Architectural Perspective
Systemic Performance Discipline should be like to not just "use tools", design systems that support observability

##### Include in CI/CD
- Measure launch time and regressions per commit
- Memory usage trends on real devices
- Custom logs for key screens & APIs (signpost traces)
- Export Instruments logs and analyze trends weekly
##### Suggested Monitoring Architecture
- Release builds report anonymized performance logs to backend
- Dashboard showing:
	- Cold start trends
	- Frame drop heatmaps
	- Device specific performance
	- Regions with degraded experience
	- API response latency and failure %

#### Interview Angle
"We integrate Instruments during development and CI to proactively monitor performance. We use `os_signpost` to time key flows like login, product detail rendering, and checkout. These are sent to a backend via custom telemetry SDK. On the frontend, we optimize cold starts by lazy loading modules, profile startup on low-end phones, and cap on-disk cache to reduce I/O. Every release goes through a performance gate on memory, CPU, and frame jank"

#### What you should Profile
| Layer        | Metric                          |
| ------------ | ------------------------------- |
| App Launch   | Cold, warm, resume time         |
| View Loading | Time to first content           |
| API Calls    | Latency, retries, failures      |
| Scrolls      | FPS, dropped frames             |
| Navigation   | Transition perf                 |
| Background   | Memory leaks, battery impact    |
| Device Type  | Low-end vs High-end differences |

| Metric/Tool | iOS (Swift)                           | Android (Kotlin)                | Backend                   |
| ----------- | ------------------------------------- | ------------------------------- | ------------------------- |
| Cold Start  | LaunchTime via Instruments            | Systrace, Android Profiler      | N/A                       |
| API Latency | `URLSession` timing metrics, Firebase | OkHttp Interceptor logs         | Prometheus, OpenTelemetry |
| Memory      | Instruments: Alloc, Leaks             | Android Studio: Memory Profiler | Heap dumps, /proc/meminfo |
| Battery     | Energy template in Instruments        | Battery Historian               | N/A                       |
| Logging     | `os_log`, `Logger`, Crashlytics       | Logcat, Timber, Crashlytics     | Elastic, Fluentd, etc.    |
