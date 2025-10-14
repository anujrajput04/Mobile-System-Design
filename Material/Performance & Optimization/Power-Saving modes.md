## Power-Saving modes
iOS aggressively manages power via system enforced power saving policies that are out of app control.

### Low Power Mode
**Trigger**: Manually enabled by user or system (battery <= 20%)

**Detectable via:**
```swift
ProcessInfo.processInfo.isLowPowerModeEnabled
NotificationCenter.default.addObserver(forName: .NSProcessInfoPowerStateDidChange, ...)
```

**Effect:**

| Resource         | Low Power Mode Behaviour   |
| ---------------- | -------------------------- |
| CPU              | Throttled                  |
| Networking       | Background fetch disabled  |
| Background tasks | Suspended or reduced       |
| Animations       | Frame rate reduced         |
| Push fetch       | Suspended                  |
| Timers           | Less frequent              |
| Location         | Reduced accuracy/frequency |

**Best Practice:** Respect `isLowPowerModeEnabled`. Defer syncs, reduce animation frame rates, pause non-essential updates

### App Throttling
- iOS may automatically reduce performance for background or long running foreground apps
- Avoid doing work outside allowed system intervals, especially for non-user visible updates 

### APIs affected

| Subsystem       | Behavior                                    |
| --------------- | ------------------------------------------- |
| NSURLSession    | Background uploads/downloads may pause      |
| CoreLocation    | Reduced accuracy / no updates in background |
| AVPlayer        | May be paused/stopped                       |
| CADisplayLink   | Reduced frame rate                          |
| Timer / RunLoop | Lower frequency callbacks                   |

### Instruments Tool
Use **Energy Log** in Instruments to see:
- Wake ups
- Timer firing
- Networking
- CPU spikes
- Background task usage

---
## Android Comparison
- Android has multiple tiers:
    - Doze mode
    - App Standby
    - Battery Saver
    - Background restrictions (starting Android 8+)
- Apps may need to use `JobScheduler` or `WorkManager` to comply with background restrictions.

iOS: you _can't_ opt out of LPM
Android: can request exemptions but discouraged

---
## Architectural View
- Design power aware features (eg. defer analytics syncs, batch writes)
- Know how OS level energy models work. You don't fight them, you design with them
- Educate team to avoid polling, use push triggers or system callbacks
- Prioritize user control: allow disabling auto sync or animations
- Optimize for Battery per Feature: eg. background GPS may not be worth 5% battery/hour

### Interview Talk
>"iOS Low Power Mode is a hard constraint. We detect it using `ProcessInfo`, reduce refresh frequency, pause animations, and disable heavy networking. I have used Instruments' Energy Log to catch unexpected wake-ups and optimized a screen refresh logic that was causing battery drain in the background. While Android offers more granular modes, our job on iOS is to respect system signals and build adaptive, polite clients that scale battery-first"