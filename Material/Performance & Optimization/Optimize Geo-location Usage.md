### Why Geolocation is Expensive?
| Resource | Cost                                                                        |
| -------- | --------------------------------------------------------------------------- |
| Battery  | GPS chip is power hungry, constant usage drains battery                     |
| CPU      | Frequent polling, background updates, location filtering                    |
| Network  | If location is sent to backend frequently (eg. for tracking and geofencing) |
| Privacy  | Location data is sensitive and tightly controlled by OS                     |

### Choosing the right Location Accuracy
In iOS, `CLLocationManager` lets you define:
- `.bestForNavigation` (most precise, most power hungry)
- `.best`
- `.nearestTenMeters`
- `.hundredMeters`
- `.kilometer`
- `.threeKilometers` (least power hungry)

Strategy:
Start coarse → refine only when needed (eg. user opens Map screen)
If the app doesn't visibly depend on location, use `.significatnLocationChanges`

### Use the Right APIs
| Purpose            | API                                           | Energy Efficiency        |
| ------------------ | --------------------------------------------- | ------------------------ |
| Continuous updates | `startUpdatingLocation()`                     | Very High                |
| Big movement only  | `startMonitoringSignificatnLocationChanges()` | Very Low                 |
| Visit detection    | `startMonitoringVisits()`                     | Very Low                 |
| One time fetch     | `requestLocation()`                           | Medium                   |
| Region entry/exit  | `startMonitoring(for:)`                       | Medium (uses geofencing) |

### Background Execution Considerations
- Use Background Location Mode sparingly (`UIBackgroundModes` in `Info.plist`)
- Combine with `CLVisit` or `significant location changes` to reduce polling
- Avoid constant updates unless user expects live tracking (eg. ride sharing)
Abuse leads to App Store rejections and user backlash

### Optimization Techniques
- Debouncing: Don't update location to backend every second, batch and throttle (eg. every 500m or 2mins)
- Distance Filter: Set `locationManager.distanceFilter = 100` (only send updates after 100m change)
- Combine with Motion: Use `CLMotionActivityManager` to detect if user is stationary, then pause location updates
- Low Power Mode Check: Reduce frequency if `ProcessInfo.processInfo.isLowPowerModeEnabled == true`
- Time of Day Awareness: Don't track when location is irrelevant (eg. night time, if app allows)

### Backend Collaboration
- Use Location to Snap-to-Road APIs like Google Roads or Mapbox to reduce device side computation
- Process location change deltas server side
- Detect stale clients and optimize backend retries if location updates are missing

### Testing Tools
- Xcode > Debug > Simulate Location > custom GPX files
- Instruments > Energy Log + Location Usage
- Console + Logging to check rate of updates and battery drain
- Use TestFlight + iOS Console app onMac for real device testing logs

### Trade-offs
| Optimization                          | Trade-off                                                   |
| ------------------------------------- | ----------------------------------------------------------- |
| Lower accuracy                        | Less precise UX (eg. navigation or map centering feels off) |
| Batch updates                         | Less real-time responsiveness                               |
| Passive tracking (visits/significant) | Might miss short trips                                      |
Defend choices based on use case:
"For fitness tracking, we use high accuracy + real time. For delivery status checks, we fetch coarse location every 5-10 minutes. For location tagging, we snapshot once and refine later if needed."

### iOS vs Android
| Concern        | iOS API                             | Android Equivalent                                |
| -------------- | ----------------------------------- | ------------------------------------------------- |
| One time fetch | `requestLocation()`                 | `FusedLocationProvider.getCurrentLocation()`      |
| Background use | `UIBackgroundModes`                 | Foreground service + `ACCESS_BACKGROUND_LOCATION` |
| Optimization   | `distanceFilter`, `desiredAccuracy` | `LocationRequest.setInterval()`, `setPriority()`  |