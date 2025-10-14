## Push Notifications
**Push Notifications** allow the server to send messages to a mobile device **via the OS-level push service** (APNs for iOS, FCM for Android), even when the app is **not running** or is **in the background**
**For iOS**:
- **APNs (Apple Push Notification service)** is the only channel to reach the device outside the app lifecycle.
- The app receives a **device token**, which is sent to the app's backend
- The backend uses this token to **send messages via APNs**

![[Push Notifications.excalidraw]]

**Types**:
- **Silent Push**: No alert shown, used to refresh data in background
- **Visible Push**: Includes alerts, sound, badge, etc
- **VoIP Push**: (deprecated) Used to wake app for call scenarios, now replace by `PushKit`  with restrictions

**Mobile Specific Considerations**:

| Concern          | Impact                                                |
| ---------------- | ----------------------------------------------------- |
| App Lifecycle    | Push works even when app is terminated                |
| Delivery Control | Sent to APNs → Delivery not guaranteed                |
| Latency          | Usually fast (~0.5-2s), not real time like WebSockets |
| Rate Limiting    | Apple enforces silent push delivery quotas            |
| User Permissions | Users must allow notifications                        |
| Privacy & Abuse  | Apple can throttle/spam-filter                        |

**App capabilities needed**:
- Enable **Push Notifications** in App Capabilities
- Register `UNUserNotificationCenter` for auth
- Use `UNNotificationServiceExtension` to modify notification payloads (eg. images, rich content)

**Example**:
```swift
/// Requesting Permission
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) { granted, error in
	if granted {
		DispatchQueue.main.async {
			UIApplication.shared.registerForRemoteNotifications()
		}
	}
}

/// Handling Token
func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
	let tokenString = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
	// Send tokenString to app backend server
}

/// Handling Incoming Notification
func userNotificationCenter(_ center: UNUserNotificationCenter,
							didReceive response: UNNotificationResponse,
							withCompletionHandler completionHandler: @escaping () -> Void) {
	let userInfo = response.notification.request.content.userInfo
	// Handle the push
	completionHandler()
}
```

**Cross-Platform & Backend Expectations**:
For Architects:
- Backend stores and manages **device tokens**, supports token expiry and refresh
- Sends pushes using **APNs HTTP/2 API**, which includes:
	- JWT Auth via Apple Key
	- JSON payload
	- Device token as identifier
- Push flow:
	1. Client registers & gets token
	2. Client sends token to backend
	3. Backend sends push via APNs with relevant payload
**Important Backend Constraints**:
- APNs enforces 4KB payload limit
- Silent pushes can be dropped by OS if app is force quit or uses too many background updates
- Delivery is not guaranteed

**Trade-offs**:

| Advantages                                        | Disadvantages                                  |
| ------------------------------------------------- | ---------------------------------------------- |
| Works outside app lifecycle, even when terminated | Requires OS level setup and Apple config       |
| Low battery cost, no open socket                  | Delivery is not guaranteed                     |
| Immediate user attention                          | User opt-in required                           |
| Great for transactional or time sensitive UX      | Not suitable for high frequency real-time data |

### Interview Talking Points
- Explain how you managed device token lifecycle - register, refresh, invalidation
- Talk about silent push limitation and fallback strategies eg. user initiated refresh
- Describe use cases you implemented: reminders, promotions, chat alerts, badge updates
- Discuss observability: APNs response codes, retries, delivery receipts (via FCM + APNs)

**Push Notification Payload Example**:
```json
{
	"aps": {
		"alert": {
			"title": "Order Update",
			"body": "Your pizza is out for delivery"
		},
		"badge": 1,
		"sound": "default",
		"content-available": 1
	},
	"customKey": "customValue"
}
```

**When to use Push Notifications**:
- Message alert when app is closed
- Silent background data sync
- Critical system alerts
- ~~High frequency stock price updates~~
- Polling replacement: only for low frequency events