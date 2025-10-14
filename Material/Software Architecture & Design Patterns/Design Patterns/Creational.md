Creational Design Pattern deal with object creation mechanisms. Instead of instantiating objects via `init()` or `new`, these patterns abstract and control the instantiation process the make the system:
- Control when, how, and what kind of objects are created
- Loosely coupled, avoids tight coupling
- Easier to test and scale, promotes code reuse, flexibility, and maintanability

>It's about how you construct objects without hardcoding their instantiation logic.

---
## Singleton
The Singleton Pattern ensures that the class has only one instance in the memory and provides a global point of access to it. It's commonly used when exactly one object is needed to coordinate actions across the system. eg. shared service, config, cache
**Examples:**
- `UserDefaults.standard`
- `UIApplication.shared`
- `URLSession.shared`
- `NotificationCenter.default`

**Implementation:**
```swift
final class AnalyticsManager {
	static let shared = AnalyticsManager()

	private init() { }

	func track(event: String) {
		print("Tracking Event: \(event)")
	}
}
```

**Usage:**
```swift
AnalyticsManager.shared.track(event: "screen_view")
```

**Pros:**
- Controlled access to shared resources.
- Lazy Initialization in Swift (via `static let`)
- Prevents race conditions in global states
**Cons:**
- Tight coupling - difficult to mock/test
- Global state - hidden dependencies
- Hard to subclass or reuse in flexible architectures
- Potential for memory leaks if misused (with self retaining closures or observers)

**Alternatives:**

| Pattern                      | When to use                                              |
| ---------------------------- | -------------------------------------------------------- |
| [[#Dependency Injection]]    | When you want loose coupling and testability             |
| Service Locator              | When global access is needed without enforcing singleton |
| Environment Values (SwiftUI) | For injecting Singletons in more testable/reactive way   |

### Interview Questions
- “Is Singleton a good fit here?” → Justify use, don’t default to it
- “How would you test this class?” → Inject dependencies or add reset/override methods for tests
- “What are the limitations of a Singleton in app-wide coordination?” → Discuss modularity, maintainability

---
## Factory
The Factory Pattern provides an interface for creating objects, allowing subclasses or external factories to decide which class to instantiate, without exposing instantiation logic to the client.

**Types:**

| Type                  | Description                                                            |
| --------------------- | ---------------------------------------------------------------------- |
| Simple Factory        | Central place that returns concrete instances based on input           |
| Factory Method        | Subclasses override method to create specific objects                  |
| [[#Abstract Factory]] | Produces family of related objects without specifying concrete classes |

**Example:**
```swift
enum ScreenType {
	case home, settings, profile
}

protocol ViewControllerFactory {
	func makeViewController(for type: ScreenType) -> UIViewController
}

class DefaultViewControllerFactory: ViewControllerFactory {
	func makeViewController(for type: ScreenType) -> UIViewController {
		switch type {
			case .home: return HomeViewController()
			case .settings: return SettingsViewController()
			case .profile: return ProfileViewController()
		}
	}
}
```

**Usage:**
```swift
let factory = DefaultViewControllerFactory()
let vc = factory.makeViewController(for: .home)
```

**Real World Examples:**

| Use Case            | Example                                                       |
| ------------------- | ------------------------------------------------------------- |
| Coordinator pattern | Creating view controllers based on navigation flow            |
| UI Testing          | Injecting mock view controllers from factory                  |
| Feature flags       | Deciding between `NewCheckoutVC` or `OldCheckoutVC`           |
| Push Notifications  | Deciding which screen to open from payload                    |
| Reusable modules    | Creating platform specific components from abstract factories |
**Pros:**
- Separates creation from use
- Improves modularity
- Easier to extend and test

**Cons:**
- Introduces additional layers
- Overkills for simple object creation
- Factory logic can become brittle and bloated if poorly structured

### Interview Questions
- How would you structure screen creation in a large iOS app?
- What design pattern would you use to avoid `switch-case` hell in navigation?
- How can you support different view controller logic per feature flag or environment?

---
## Builder
The Builder Pattern allows you to construct complex objects step by step, separating the construction process from representation. It is ideal when:
- You need to assemble an object from multiple parts
- The construction logic is non-trivial
- You want to avoid long **init()** methods with many parameters

**When to use**
- Creating complex structs/classes like `URLRequest`, `NSAttributedString`, `UIAlertController`
- Building configurable UIs (eg. Forms, Onboarding screens)
- Generating models with optional fields or conditional defaults
- Constructing test fixtures with reusable configuration blocks

**Example:**
```swift
struct URLRequestBuilder {
	private var url: URL?
	private var method: String = "GET"
	private var headers: [String: String] = [:]
	private var body: Data?

	mutating func setURL(_ url: URL) -> Self {
		self.url = url
		return self
	}

	mutating func setMethod(_ method: String) -> Self {
		self.method = method
		return method
	}

	mutating func setHeader(key: String, value: String) -> Self {
		headers[key] = value
		return self
	}

	mutating func setBody(_ data: Data) -> Self {
		self.body = body
		return self
	}

	func build() -> URLRequest? {
		guard let url = url else { return nil }
		var request = URLRequest(url: url)
		request.httpMethod = method
		request.allHTTPHeaderFields = headers
		request.httpBody = body
		return request
	}
}
```

**Usage:**
```swift
var builder = URLRequestBuilder()
let request = builder
	.setURL(URL(string: "https://api.example.com"))
	.setMethod("POST")
	.setHeader(key: "Content-Type", value: "application/json")
	.setBody(Data())
	.build()
```

**Real Examples:**

| Use Case           | Builder-like Pattern                                     |
| ------------------ | -------------------------------------------------------- |
| NSAttributedString | Combine fonts, colors, spacing step-by-step              |
| UIView layout      | Using method chaining with Auto Layout DSLs like SnapKit |
| Form builders      | Input fields, validation, optional sections              |
| Test data builders | Creating mock models with default + override options     |

**Pros:**
- Cleaner than initializer with many parameters
- Improves readability and maintanability
- Allows fluent interfaces
- Good for creating immutable objects step-by-step

**Cons:**
- Nay require mutable intermediate state
- Can lead to code duplication if overused
- Not necessary for simple models or one liners

**Builder vs Factory**

| Aspect       | Builder                            | Factory                                 |
| ------------ | ---------------------------------- | --------------------------------------- |
| Purpose      | Step by step construction          | Object creation based on type/config    |
| Suitable for | Complex object config              | Object type selection                   |
| Example      | `NSAttributedString`, `URLRequest` | `ViewControllerFactory`, `ThemeFactory` |
### Interview Questions
- Asked for **complex object assembly**: e.g., `UserProfile` from many optional inputs
- Compare with **Factory**: Builder = _how_ to build; Factory = _what_ to build
- Often combined with **Fluent Interface** for API clients or config systems

---
## Abstract Factory
The Abstract Factory pattern is a Design Pattern that provides an interface for creating families of related or dependent objects without specifying concrete classes,
> Think of it as, "A factory of factories"

**Core problem it solves:**
When you need to create objects that must work together as a group, and you want to swap families of these objects without touching the client code.

**Use Cases:**

| Use Case                  | Example                                               |
| ------------------------- | ----------------------------------------------------- |
| Theming systems           | Light vs Dark vs High Contrast UI components          |
| Cross platform components | iOS specific vs macOS specific UI factories           |
| A/B feature experiments   | Different object families for different user groups   |
| Component Libraries       | Swappable UI factories for clients of a design system |
### **Examples:**
#### **UI Theme Factory**

**Step 1: Abstract Product Protocols**
```swift
protocol Button {
	func render() -> String
}

protocol Label {
	func render() -> String
}
```

**Step 2: Concrete Product Variants**
```swift
class LightButton: Button {
	func render() -> String { "Light Button" }
}

class DarkButton: Button {
	func render() -> String { "Dark Button" }
}

class LightLabel: Label {
	func render() -> String { "Light Label" }
}

class DarkLabel: Label {
	func render() -> String { "Dark Label" }
}
```

**Step 3: Abstract Factory**
```swift
protocol ThemeFactory {
	func makeButton() -> Button
	func makeLabel() -> Label
}
```

**Step 4: Concrete Factories**
```swift
class LightThemeFactory: ThemeFactory {
	func makeButton() -> Button { LightButton() }
	func makeLabel() -> Label { LightLabel() }
}

class DarkThemeFactory: ThemeFactory {
	func makeButton() -> Button { DarkButton() }
	func makeLabel() -> Label { DarkLabel() }
}
```

**Step 5: Usage**
```swift
let themeFactory: ThemeFactory = DarkThemeFactory()
let button = themeFactory.makeButton()
let label = themeFactory.makeLabel()

print(button.render()) // "Dark Button"
print(label.render()) // "Dark Label"
```

#### Component Library

**Step 1: Abstract Product Protocols**
```swift
protocol UIComponentButton {
	func render() -> UIButton
}

protocol UIComponentCard {
	func render() -> UIView
}
```

**Step 2: Concrete Product Implementation**
- Default Style
```swift
class DefaultButton: UIComponentButton {
	func render() -> UIButton {
		let button = UIButton(type: .system)
		button.setTitle("Default Button", for: .normal)
		button.backgroundColor = .systemBlue
		return button
	}
}

class DefaultCard: UIComponentCard {
	func render() -> UIView {
		let view = UIView()
		view.backgroundColor = .white
		view.layer.cornerRadius = 9
		return view
	}
}
```

- Branded Style
```swift
class BrandedButton: UIComponentButton {
	func render() -> UIButton {
		let button = UIButton(type: .system)
		button.setTitle("Branded", for: .normal)
		button.backgroundColor = UIColor(named: "BrandColor")
		return button
	}
}

class BrandedCard: UIComponentCard {
	func render() -> UIView {
		let view = UIView()
		view.backgroundColor = UIColor(named: "BrandBackground")
		view.layer.cornerRadius = 16
		view.layer.borderWidth = 2
		view.layer.borderColor = UIColor(named: "Accent")?.cgColor
		return view
	}
}
```

**Step 3: Abstract Factory Protocol**
```swift
protocol ComponentFactory {
	func makeButton() -> UIComponentButton
	func makeCard() -> UIComponentCard
}
```

**Step 4: Implement Concrete Factory**
```swift
class DefaultComponentFactory: ComponentFactory {
	func makeButton() -> UIComponentButton { DefaultButton() }
	func makeCard() -> UIComponentButton { DefaultCard() }
}

class BrandedComponentFactory: ComponentFactory {
	func makeButton() -> UIComponentButton { BrandedButton() }
	func makeCard() -> UIComponentButton { BrandedCard() }
}
```

**Step 5: Client Code**
```swift
class HomeViewController: UIViewController {
	private let factory: ComponentFactory

	init(factory: ComponentFactory) {
		self.factory = factory
		super.init(nibName: nil, bundle: nil)
	}

	required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

	override func viewDidLoad() {
		super.viewDidLoad()
		view.backgroundColor = .systemBackgroundColor

		let button = factory.makeButton().render()
		let card = factory.makeCard().render()

		view.addSubview(card)
		view.addSubview(button)

		card.frame = CGRect(x: 20, y: 100, width: 300, height: 150)
		button.frame = CGRect(x:20, y: 270, width: 200, height: 44)
	}
}
```

**Benefits:**
- Isolates families of related objects
- Makes theme switching, feature toggling, or platform abstraction easy
- Encourages interface driven design
- Clean separation of object creation from usage

**Pros:**
- Clear grouping of related objects
- Easy to extend new object families
- Plug and play architecture

**Cons:**
- Can become verbose with many interfaces
- Increases complexity for small apps
- Risk of over architecting early

### Interview Questions
- Design a **theme engine** for an app that supports 4 UI themes
- Abstract a **cross-platform SDK** that provides platform-specific views
- Enable **A/B testing** of feature UIs without hardcoding conditions
- Build **plugin architecture** for clients to inject their own views/services

---
## Prototype
The Prototype Pattern is a Design Pattern where you create new objects by cloning existing ones (ie. by copying a prototype) rather than instantiating from scratch.
> "Don't build from blueprint. Copy from an existing object"

This is useful when:
- Object creation is expensive or complex.
- You want to preserve internal configuration/state
- You need to create multiple similar objects quickly and dynamically.

**When to use:**

| Use Case                       | Why Prototype works                                                  |
| ------------------------------ | -------------------------------------------------------------------- |
| Repeating styled UIViews       | Clone once configured views instead of rebuilding                    |
| Animations with similar layers | Duplicate `CALayer` trees                                            |
| Prefilled forms or models      | Create a copy of a template model object                             |
| View or object pooling         | Reset and reuse cloned components instead of deallocating/allocating |
| Custom drag & drop previews    | Clone UI elements for animations or previews                         |
**Example - Cloning a UIView:**
**Define a Clonable Protocol**
``
```swift
protocol Clonable {
	func clone() -> Self
}
```

**Extend a Custom View:**
```swift
class StyledCardView: UIView, Clonable {
	var title: String: ""

	override init(frame: CGRect) {
		super.init(frame: frame)
		self.backgroundColor = .white
		self.layer.cornerRadius = 12
	}
	
	required init?(coder: NSCoder) { fatalError() }

	func clone() -> Self {
		let copy = Self.init(frame: self.frame)
		copy.title = self.title
		copy.backgroundColor = self.backgroundColor
		copy.layer.cornerRadius = self.layer.cornerRadius
		return copy
	}
}
```

**Usage:**
```swift
let card1 = StyledCardView(frame: CGRect(x: 0, y: 0, width: 300, height: 200))
card1.title = "Card A"

let card2 = card1.clone()
card2.title = "Card B"
```

**Prototype in Data Models:**
```swift
struct FormData: Codable, Equatable {
	var name: String
	var email: String

	func clone() -> Self {
		return self
	}
}
```

**For reference type data:**
```swift
class UserProfile: NSCopying {
	var name: String
	var interests: [String]

	init(name: String, interests: [String]) {
		self.name = name
		self.interests = interests
	}

	func copy(with zone: NSZone? = nil) -> Any {
		return UserProfile(name: self.name, interests: self.interests)
	}
}
```

**Pros**
- Avoids rebuilding complex objects
- Efficient in runtime duplication
- Useful for configurable templates
- Reduces need for large constructor parameters

**Cons**
- Shallow copies can lead to shared mutable state bugs
- Deep copying gets messy with complex object graphs
- Requires all class to conform to `clone` / `copy`

### Interview Questions
- "How would you implement object pooling?" → Use Prototype to clone base objects
- "How do you handle reusable UI components that retain internal state?" → Clone and reset
- "What if the creation cost is high?" → Use Prototype to optimize performance