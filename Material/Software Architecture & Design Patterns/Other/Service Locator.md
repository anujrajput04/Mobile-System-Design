## Service Locator
**Service Locator** is a design pattern that provides a **global registry** where dependencies (services) are registered and looked up dynamically at runtime. It acts as a **central container** that hides object creation logic, but unlike dependency injection, **objects pull** wheat they need rather than having it passed in.
>Instead of injecting dependencies, you as a registry to give them to you

![[Service Locator.excalidraw]]

**How it Works:**
```
Class → asks ServiceLocator → returns Dependency Instance
```

| Component              | Description                                  |
| ---------------------- | -------------------------------------------- |
| Service Protocol       | Interface of the dependency                  |
| Service implementation | Concrete class that performs the task        |
| Service Locator        | Central registry mapping types to instances  |
| Client                 | Code that asks for services from the locator |

**Example - Logger Service:**
```swift
/// Define Protocol
protocol Logger {
	func log(_ message: String)
}

/// Concrete Implementation
class ConsoleLogger: Logger {
	func log(_ message: String) {
		print("LOG:", message)
	}
}

/// Service Locator
final class ServiceLocator {
	static let shared = ServiceLocator()

	private var services: [String: Any] = [:]

	func register<T>(_ service: T) {
		let key = String(describing: T.self)
		services[key] = service
	}

	func resolve<T>() -> T {
		let key = String(describing: T.self)
		guard let service = services[key] as? T else {
			fatalError("No service registered for type \(T.self)")
		}
		return service
	}
}

/// Usage
class ReportManager {
	private let logger: Logger

	init() {
		self.logger = ServiceLocator.shared.resolve()
	}

	func generateReport() {
		logger.log("Report generation started.")
	}
}

/// Registereing Services
ServiceLocator.shared.register(Logger.self, service: ConsoleLogger())
// or
ServiceLocator.shared.register(ConsoleLogger as Logger)
```

**Pros:**
- Centralized control - You don't need to pass dependencies everywhere
- Runtime flexibility - Swap implementations easily (eg. mock vs prod)
- Low friction in small apps - Avoids constructor bloat
- Dynamic resolution - Load services based on conditions (e. A/B flag)

**Cons:**
- Hidden dependencies - Hard to trace what a class needs just by looking at it
- Global state - Makes unit testing harder and encourages tight coupling
- Runtime failure - In services aren't registered correctly, it crashes
- Harder to reason about - Especially in large teams and modular apps

**Best Practices:**
- Use **protocols** for registration keys, not class types directly
- Register all services **at app launch** (AppDelegate or Bootstrapper)
- Don't overuse: prefer **Dependency Injection** for business/core logic
- Use **generics** to keep type-safe resolution
- For testability, allow **reset or override** during tests

**Service Locator vs Dependency Injection:**

| Feature                    | Service Locator            | Dependency Injection          |
| -------------------------- | -------------------------- | ----------------------------- |
| Who creates dependencies   | Registry (`resolve`)       | External injector (container) |
| How dependencies arrive    | Pulled by class            | Passed to class constructor   |
| Visibility of dependencies | ❌ HIdden                   | ✅ Explicit                    |
| Ideal for                  | Frameworks, small tools    | Business logic, testable code |
| Testing impact             | Medium (requires override) | ✅ High (easy to mock)         |

**Use Cases:**
- Utility services (logger, analytics, crash reporting) - Low level services needed across the app
- When using legacy or UIKit-heavy codebases - Avoids refactoring for DI
- Building frameworks or SDKs - Exposes clean APIs without pushing complexity to consumer
- Runtime plugin systems - Dynamically resolve feature modules at runtime