The most common sources of memory leaks in iOS apps. Each of these has been observed in real-world apps at scale

#### Retain Cycles in Closures
Cause: Capturing `self` strongly inside closures, especially when the closure is retained by the same object
Where it happens:
- Networking calls
- Animations
- Completion handlers
- Timers
- Notification observers

Example:
```swift
class ProfileVC: UIViewController {
	func fetchData() {
		apiClient.getData { result in
			self.updateUI() // Leak: self is retained strongly
		}
	}
}
```

Fix:
```swift
apiClient.getData{ [weak self] result in 
	self?.updateUI()
}
```

Always audit closures inside long lived objects eg. ViewControllers, Managers

---
#### Strong Delegates
Cause: Holding strong references to a delegate, creating a reference cycle

Where it happens:
- Custom views and cells
- Scroll view delegates
- Table/Data source patterns

Wrong:
```swift
class MyView {
	var delegate: MyViewDelegate? // strong by default
}
```

Correct:
```swift
weak var delegate: MyViewDelegate?
```

---
#### Timers & DisplayLinks
Cause: `Timer`, `CADisplayLink`, and similar APIs retain their target

Example:
```swift
class CountdownController {
	var timer: Timer?

	func start() {
		timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) {_ in 
			self.tick() // self retained strongly
		}
	}
}
```

Fix:
```swift
timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in 
	self?.tick()
}
```

or use `Combine` or `GCD` to schedule loosely coupled timers

---
#### NotificationCenter Observers
Cause: You must manually remove observers in pre-iOS 9 or when using `addObserver(forName:object:queue:using:)`

Fix: Use the returned `NSObjectProtocol` and remove observer in `deinit`
Better: Use`NSNotificationCenter.default.addObserver(self, selector:...)`, this auto cleans on `deinit`

---
#### Combine Subscriptions
Cause: Not cancelling Combine subscriptions, leading to retained objects
Where it happens: ViewModels, Network layers

Fix:
- Always cancel Combine subscriptions in `deinit`
- Use `AnyCancellable` stored in a `Set<AnyCancellable>`

```swift
class MyViewModel {
	var cancellables = Set<AnyCancellable>()

	func bind() {
		api.publisher
			.sink { _ in }
			.store(in: &cancellables)
	}
}
```

---
#### SwiftUI State Objects or ObservedObjects Leaking
Cause: View holds onto a reference too long or doesn't break dependency cycles
Tips:
- Be careful with custom Combine publishers in side `ObservableObject`
- Don't create strong references between view → view model → view
- Avoid `@ObservedObject` if lifecycle is owned elsewhere, use `@EnvionrmentObject`

---
#### Data Caching and Image Retention
Cause: Not invalidating in-memory caches ([[In-Memory Cache#iOS: `NSCache<KeyType, ObjectType>`|NSCache]], arrays, dictionaries)

Example:
- Holding large `UIImage`s or `Data` in memory unnecessarily
- Using unbounded arrays/lists to store fetched data or logs

Fix:
- Use [[In-Memory Cache#iOS: `NSCache<KeyType, ObjectType>`|NSCache]], it auto purges under memory pressure
- Invalidate caches in `didReceiveMemoryWarning()` or `sceneDidEnterBackground()`

---
#### Third party Libraries
Cause: Misused or outdated SDKs (analytics, ads, A/B Testing) that retain VC references
Tip:
- Use memory graph debugger after integrating third party libraries
- Avoid [[Creational#Singleton|Singletons]] storing VC references
- Isolate integrations behind protocol interfaces

---
#### DispatchQueue.async / OperationQueue
Cause: Retain cycles inside async blocks dispatched onto queues

Fix:
```swift
DispatchQueue.global().async { [weak self] in
	self?.doHeavyStuff()
}
```

---
#### How to Detect
- Instruments > Leaks
- Instruments > Allocations > Persistent objects
- Xcode Memory Graph: Look for strong references in object chains that should be weak or nil
- Unit tests: `XCTAssertNil(weakRef)` after object deallocation in test

---
#### Interview Angle
>"We proactively prevent leaks through code reviews, automated leak tests, and memory graph analysis in CI. Our closure patterns enforce `[weak self]` in view related closures, we use Combine responsibly with clear cancellation points, and cache large data using [[In-Memory Cache#iOS: `NSCache<KeyType, ObjectType>`|NSCache]] to avoid ownership retention. We have also wrapped third party SDKs to isolate memory leaks"