## App Startup Time
App startup time is a critical performance metric, it's the user's first impression and directly tied to retention.

### Types of App Launch
- Cold Start: App is not in memory. Process, system, and runtime all need to be initialized
- Warm Start: App is in memory but not running in foreground. Needs UI reload
- Hot Start: App is suspended and resumes from memory. Fastest

For Performance tracking, cold start is the most important

### iOS Launch Timeline

| Phase                             | What happens                                | Optimize for                               |
| --------------------------------- | ------------------------------------------- | ------------------------------------------ |
| Process Launch                    | OS loads binary, dyld loads dependencies    | Binary size, dependency count              |
| `main()` to `UIApplicationMain`   | Static initializers run, environment setups | Avoid heavy global initializations         |
| App Delegate `didFinishLaunching` | App sets up state, services, view hierarchy | Lazy loading, async setup, parallelization |
| First Frame Render                | UIKit renders first frame on screen         | Avoid blocking main thread, sync IO        |

Apple defines startup tim as:
time from user tap → first frame rendered

### Measuring Startup Time
Tools:
- Instruments > Time Profiler
- Instruments > Logging - Signpost API
- `os_signpost` to mark custom startup phases
- Xcode Organizer (Launch Time histogram per build)
- Console Logs, see `App Launch` markers

Third-party:
- Firebase Performance Monitoring
- Datadog, Instana, etc. with manual instrumentation

### Startup Optimization Strategies
**Reduce Binary and Asset Size**
- App thinning (bitcode, on-demand resources)
- Compress or defer loading heavy assets
- Remove unused dependencies

**Lazy Initialization**
- Postpone setting up SDKs, caches, or data until needed
- Avoid object graph creation during `didFinishLaunching`

**Defer Network Calls**
- Avoid making API calls before first screen
- Prefetch in background where possible

**Move Work Off Main Thread**
- Use async/await, DispatchQueue.global(), or background queues to offload disk or compute operations

**Reduce View Hierarchy Cost**
- Avoid complex layouts or auto-layout chains at launch
- Use LaunchScreen.storyboard smartly, it's just a static image

### Best Practices

| Practice                                | Why it matters                                                 |
| --------------------------------------- | -------------------------------------------------------------- |
| Use lightweight LaunchScreen.storyboard | Keeps the system UI illusion smooth and fast                   |
| Avoid DB migrations on launch           | Move to background queue or do during idle time                |
| Defer non-critical SDK setups           | Especially crash logging, analytics, etc.                      |
| Measure all changes                     | Performance regressions are often subtle and gradual           |
| Use MetricsKit                          | Provides `applicationLaunchTime`, `applicationResumeTime`, etc |

### Cross-Platform Considerations

| Platform | Strategy                                         |
| -------- | ------------------------------------------------ |
| Android  | Leverage SplashScreens API, App Startup Profiler |
| Backend  | Reduce API response size and time                |
| Infra    | Use CDNs for critical assets                     |

### Interview Framing
- Speak in phases: binary load → runtime init → app delegate → UI render
- Talk about measuring before optimizing
- Show tradeoff thinking (eg. cold start vs warm resume)
- Mention bootstrapping essential features first, deferring the rest
- Bonus: Architecting apps with modular lazy loaded features