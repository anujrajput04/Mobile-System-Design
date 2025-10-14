Dependency Injection is a design pattern that deals with how objects receive their dependencies instead of creating them internally.
> "Don't ask for your dependencies - have them given to you"

## **Core Concept**
**Instead of this:**
```swift
class AnalyticsManager {
	private let logger = ConsoleLogger()
}
```

**Do this:**
```swift
class AnalyticsManager {
	private let logger: LoggerProtocol

	init(logger: LoggerProtocol) {
		self.logger = logger
	}
}
```

Now the `AnalyticsManager` does not depend on the concrete implementation. It just depends on the abstraction, which makes it more flexible, testable, and modular.

## Types of Dependency Injection

| Type                  | Mechanism                             | Example                              |
| --------------------- | ------------------------------------- | ------------------------------------ |
| Constructor Injection | Inject via `init()`                   | `init(apiClient: APIClientProtocol)` |
| Property Injection    | Inject via public/settable properties | `viewModel.logger = logger`          |
| Method Injection      | Inject via method parameters          | `configure(with logger: Logger)`     |
Constructor Injection is the cleanest and safest for most use cases.

## Example - Constructor Injection

```swift
protocol Logger {
	func log(_ message: String)
}

class ConsoleLogger: Logger {
	func log(_ message: String) {
		print("LOG: \(message)")
	}
}

class AnalyticsManager {
	private let logger: Logger

	init(logger: Logger) {
		self.logger = logger
	}

	func track(_ event: String) {
		logger.log("Tracking event: \(event)")
	}
}
```

## In Tests

```swift
class MockLogger: Logger {
	var loggedMessages: [String] = []
	func log(_ message: String) {
		loggedMessages.append(message)
	}
}

let mockLogger = MockLogger()
let analytics = AnalyticsManager(logger: mockLogger)
analytics.track("signup_success")

// Now you can asset on `mockLogger.loggedMessages`
```

## Use Cases

| Use Case      | DI Benefit                               |
| ------------- | ---------------------------------------- |
| ViewModels    | Inject services, data sources            |
| Coordinators  | Inject factories and flows               |
| Network Layer | Mockable `APIClient`                     |
| Feature Flags | Dynamic toggles without hardcoding logic |
| Unit/UI Tests | Inject mocks/fakes/stubs easily          |

## Dependency Injection vs Singleton

| Aspect        | Dependency Injection | Singleton             |
| ------------- | -------------------- | --------------------- |
| Scope Control | Caller decides       | Global scope          |
| Testability   | High                 | Low                   |
| Coupling      | Loose                | Tight                 |
| Flexibility   | High                 | Low (single instance) |
> Use DI when flexibility and testability are priorities. Use Singleton when global shared state is unavoidable. eg `UserDefaults`

## DI Frameworks
- Swinject - full featured DI container
- Needle (by Uber) - Compile time safe DI
- Resolver - Swift friendly DI container

You don't need a DI framework to do DI. In small to medium iOS projects, manual DI is sufficient and preferred.

## Interview Questions
- How do you inject dependencies in your iOS app?
- Why would you choose DI over Singleton?
- How does DI help with testability?
- How do you avoid tight coupling in your view controllers?