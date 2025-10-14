Structural Design Patterns focus on how classes and objects are composed to form larger structures.
These patterns:
- Organize code into flexible, maintainable modules
- Control relationships and dependencies between objects
- Build scalable, reusable object hierarchies or wrappers

>It's about how things are put together, not how they behave or are created.

---
## Facade
The Facade Pattern is a Design Pattern that provides a simplified interface to a complex subsystem. It hides the complexity of multiple interacting components and exposes a single entry point to the client.
> "Put a friendly front door on a messy house."

- The facade doesn't add new functionality
- It delegates tasks to other classes and components internally
- It's useful when subsystems are overwhelming or tightly coupled

**Use Cases**:

| Scenario                   | Role                                                       |
| -------------------------- | ---------------------------------------------------------- |
| Onboarding Flow            | Wrap navigation, analytics, UI prep into one manager       |
| Media Player               | Wrap `AVPlayer`, controls, notifications into a simple API |
| Networking                 | Wrap `URLSession`, headers, logging, and decoding          |
| Core Data Stack            | Wrap context, store coordination, migrations, etc          |
| Push Notification Handling | Wrap registration, parsing, analytics, and routing         |

**Example - Media Player:**
- Without Facade
```swift
let player = AVPlayer()
player.replaceCurrentItem(with: AVPlayerItem(url: someURL))
player.play()
NotificationCenter.default.addObserver(self, selector: #selector(handleEnd), name: .AVPlayerItemDidPlayToEndTime, object: nil)
```

- With Facade
```swift
class SimpleMediaPlayer {
	private let player = AVPlayer()

	func play(from url: URL) {
		let item = AVPlayerItem(url: url)
		player.replaceCurrentItem(with: item)
		player.play()
		NotificationCenter.default.addObserver(
			self,
			selector: #selector(handleEnd),
			name: .AVPlayerItemDidPlayToEndTime,
			object: item
		)
	}

	@objc private func handleEnd() {
		print("Playback finished.")
	}
}
```

Usage:
```swift
let mediaPlayer = SimpleMediaPlayer()
mediaPlayer.play(from: someURL)
```

Pros:
- Simplifies complex subsystems
- Reduces coupling between client and many subcomponents
- Improves maintanability and readability
- Great for cross functional teams: backend devs, product teams, or QA can work with a single API

Cons:
- Many become a god object if not kept minimal
- Overuse can hide important details needed for debugging or edge case control
- Can add extra abstraction layer that's not necessary

**Real World Examples:**
- `UIKit` - Hides low level drawing and event logic behind `UIView`
- `URLSession`- Hides sockets, streams, and caching
- Custom `APIClient` - Wraps session, encoding, decoding, headers
- `FirebaseManager` - Wraps Auth, Firestore, Analytics, Crashlytics

### Interview Questions
- You have a complex data pipeline. How will you abstract it for client modules?
- Design a notification manager that handles APNs, Firebase, Analytics, and Routing
- Create a user account manager that handles login, session, cache, and profile loading

---
## Proxy
Proxy Pattern is a Structural Design Pattern that provides a placeholder or surrogate for another object to control access, add functionality, or delay instantiation.
> "Same interface, added control"

A proxy acts on behalf of a real object and decides how or when to forward requests

**Types:**

| Type                     | Purpose                             | Use Case                                        |
| ------------------------ | ----------------------------------- | ----------------------------------------------- |
| Virtual Proxy            | Delay expensive object creation     | Lazy loading heavy views or assets              |
| Remote Proxy             | Represent an object over a network  | Web service stubs or simulated APIs             |
| Protection Proxy         | Control access based on permissions | Gate access to premium features                 |
| Logging/Monitoring Proxy | Add analytics, logging, throttling  | Wrap services to track usage or add retry logic |

**Example - Analytics Proxy:**
```swift
protocol Analytics {
	func track(event: String)
}

/// Real Implementation
class RealAnalytics: Analytics {
	func track(event: String) {
		print("Tracking: \(event)")
		// Send to real analytics backend
	}
}

/// Proxy Implementation
class AnalyticsProxy: Analytics {
	private let realAnalytics: Analytics
	private var isUserConsentGive: Bool = false

	init(realAnalytics: Analytics) {
		self.realAnalytics = realAnalytics
	}

	func track(event: String) {
		guard isUserConsentGiven else {
			print("Skipped tracking for: \(event)")
			return
		}
		realAnalytics.track(event: event)
	}

	func giveConsent() {
		isUserConsentGiven = true
	}
}

/// Usage
let analytics = AnalyticsProxy(realAnalytics: RealAnalytics())
analytics.track(event: "view_loaded") // Will not send without consent
analytics.giveConsent()
analytics.track(event: "button_clicked") // Now sends
```

Pros:
- Adds flexibility and control over how and when real object is used
- Enables lazy initialization, acces control, logging, or network retry
- Helps mock real services without changing client code

Cons:
- Adds indirection, can reduce readability if overused
- Easy to misuse for things better solved with decorators or middleware
- Tight coupling if proxies aren't interface driven

**Use Cases:**

| Scenario                    | Role                                                 |
| --------------------------- | ---------------------------------------------------- |
| Lazy loading images         | Use proxy to load only when in view                  |
| View lifecycle analytics    | Proxy between VC and analytics layer                 |
| Premium feature restriction | Gate access to logic via proxy                       |
| Mocking APIClient           | Swap real network layer with test proxy              |
| Retrying failed API calls   | Proxy that wraps network call and retries on failure |

### Interview Questions
- How would you delay loading an image heavy section until visible?
- How would you implement feature restrictions for a freemium app?
- Can you swap your network layer with a mock implementation without changing calling code?

---
## Adapter
The Adapter Pattern is a Structural Design Pattern that allows objets with incompatible interfaces to work together by wrapping one inside another
> "I don't change the plug, I use an adapter"

- You wrap an existing class to make it conform to a required interface
- Commonly used when integrating third party library, legacy code, or platform APIs that don't match app's abstractions

**Use Cases:**

| Scenario                       | Role                                                  |
| ------------------------------ | ----------------------------------------------------- |
| Integrating legacy API         | Make older models conform to modern protocols         |
| Using third party libraries    | Wrap SDK types into your domain models                |
| Bridging Swift and Objective-C | Create adapters that work across languages            |
| Protocol oriented design       | Wrap platform types to conform to app level protocols |
| Mocking in tests               | Adapt real objects to fit test doubles                |

**Example - Analytics Adapter:**
```swift
/// App Level Protocol
protocol Analytics {
	func track(event: String)
}

/// Third Party Class (Incompatible)
class FirebaseTracker {
	func logEvent(_ name: String) {
		print("Firebase logged: \(name)")
	}
}

/// Adapter
class FirebaseAnalyticsAdapter: Analytics {
	private let tracker = FirebaseTracker()

	func track(event: String) {
		tracker.logEvent(event)
	}
}

/// Usage
let analytics: Analytics = FirebaseAnalyticsAdapter()
analytics.track(event: "screen_loaded")
```

**Real World examples:**
- `UITableViewDataSource` with custom data → Adapter between model array and cell logic
- Legacy models in new architecture → Adapter to new `Codable` or SwiftUI models
- Combine to async/awat bridging → Adapter wrapping publishers in async streams
- Custom drawing libraries → Adapter from their `Canvas` to your `Drawable` protocol

Pros:
- Preserves interface compatibility without modifying original code
- Clean separation of your app's interface vs third party interface
- Helps with gradual migration of legacy code
- Makes testing easier through protocol conformance

Cons:
- Adds extra indirection
- Too many adapters = bloated architecture
- Can be misused to force fit bad abstraction boundaries

### Interview Questions
- How would you integrate Firebase Auth with your app's own `URLSession` model?
- You're using two analytics services. How would you abstract and unify them?
- Your legacy API returns Objective-C objects. How will you make them usable in Swift code?

---
## Decorator
The Decorator Pattern is a Structural Design Pattern that allows you to dynamically add behavior to an object without modifying its structure or subclassing.
> Wrap the object, extend its behavior, without changing the original

**When to use:**
- You need to enhance an object's behavior at runtime
- You want to compose features without creating massive subclass tree
- You want to add cross cutting concerns (logging, caching, analytics)

**Use Cases:**

| Scenario                 | Role                                               |
| ------------------------ | -------------------------------------------------- |
| Logging/caching wrappers | Wrap around `APIClient`, `FileManager`, etc        |
| UI theming wrappers      | Wrap views for styling, animations                 |
| Analytics tracking       | Add tracking around service methods                |
| Retry/backoff logic      | Decorate API calls without changing the base logic |
| SwiftUI view modifiers   | SwiftUI `.modifier()` is a form of decoration      |

**Example - Logging Decorator for APIClient:**
```swift
/// Common Protocol
protocl APIClient {
	func fetchData(endpoint: String) -> String
}

/// Real Implementation
class RealAPIClient: APIClient {
	func fetchData(endpoint: String) -> String {
		return "Data from \(endpoint)"
	}
}

/// Decorator Implementation
class LoggingAPIClient: APIClient {
	private let wrapped: APIClient

	init(wrapping: APIClient) {
		self.wrapped = wrapping
	}

	func fetchData(endpoint: String) -> String {
		print("Fetching from \(endpoint)")
		let result = wrapped.fetchData(endpoint: endpoint)
		print("Received: \(result)")
		return result
	}
}

/// Usage
let client = LoggingAPIClient(wrapping: RealAPIClient())
let response = cleint.fetchData(endpoint: "/user")
```

**Pros:**
- Behavior is composable, can chain multiple decorators
- Avoids subclass explosion
- Enhances single responsibility by separating concerns (API logic vs logging)
- Fully transparent to client

**Cons:**
- Can lead to many small classes if overused
- Layering decorators can get confusing to debug
- Avoids runtime indirection

**Real world Examples:**
- `NSAttributedString` - Decorates text with attributes
- SwiftUI `.modifier()`  - Decorates view with a new behavior
- `CALayer` tree - Decorations like shadows, borders, masks
- `URLProtocol` subclasses - Decorate requests/responses with logic (mocks, logs)
- Custom logger wrappers - Decorate `Logger` or `Analytics` services

### Interview Questions
- Add retry and logging to your network layer without modifying it
- Add Analytics to feature interactions without touching view code
- Design a middleware stack for your service calls

---
## Composite
The Composite Pattern is a structural design pattern that allows recursive containment and lets you treat individual objects and compositions of objects uniformly by modeling them using a tree structure. eg. `UIView` can contain other `UIView`s
>Part whole hierarchy with the same interface

| Role      | Responsibility                                      |
| --------- | --------------------------------------------------- |
| Component | Base interface, common to both leaf and container   |
| Leaf      | End node in a structure, performs actual work       |
| Composite | Can hold children, delegates or aggregates behavior |

**Real World Analogies:**

| Scenario                                  | Composite Role                              |
| ----------------------------------------- | ------------------------------------------- |
| `UIView` and `UIViewController hierarchy` | UIKit is a native composite system          |
| SwiftUI's `View` trees                    | Declarative composite eg. `VStack`, `Group` |
| Menus, file systems, scene graphs         | Nodes with children vs leaf elements        |
| Chat threads with replies                 | Messages as composite of nested messages    |

**Example - File System Representation:**
```swift
/// Protocol
protocol FileComponent {
	var name: String { get }
	func size() -> Int
	func display(indent: String)
}

/// Leaf
struct File: FileComponent {
	let name: String
	let fileSize: Int

	func size() -> Int { fileSize }

	func display(indent: String) {
		print("\(indent)- File: \(name) (\(fileSize) KB)")
	}
}

/// Composite
class Folder: FileComponent {
	let name: String
	private var content: [FileComponent] = []

	init(name: String) { self.name = name }

	func add(_ item: FileComponent) {
		contents.append(item)
	}

	func size() -> Int {
		contents.reduce(0) { $0 + $1.size() }
	}

	func display(indent: String = "") {
		print("\(indent)+ Folder: \(name) (\(size()) KB)")
		for item in contents {
			item.display(indent: indent + " ")
		}
	}
}

/// Usage
let doc1 = File(name: "Resume.pdf", fileSize: 400)
let doc2 = File(name: "CoverLetter.pdf", fileSize: 250)

let docsFolder = Folder(name: "Documents")
docsFolder.add(doc1)
docsFolder.add(doc2)

let img1 = File(name: "Selfie.jpg", fileSize: 1200)
let img2 = File(name: "Food.png", fileSize: 800)

let picsFolder = Folder(name: "Pictures")
picsFolder.add(img1)
picsFolder.add(img2)

let root = Folder(name: "iCloud Drive")
root.add(docsFolder)
root.add(picsFolder)

root.display()
```

```markdown
/// Output
+ Folder: iCloud Drive (2650 KB)
  + Folder: Documents (650 KB)
    - File: Resume.pdf (400 KB)
    - File: CoverLetter.pdf (250 KB)
  + Folder: Pictures (2000 KB)
    - File: Selfie.jpg (1200 KB)
    - File: Food.png (800 KB)
```

**Pros:**
- Uniform treatment of simple (leaf) and complex (composite) objects
- Natural fit for tree like structures
- Recursive rendering, parsing, event handling
- Supports open-ended nesting

**Cons:**
- Complex navigation - If you need to go up the tree or track parent, extra logic is needed
- Memory risk - Infinite nesting can blow up memory
- Mutation safety - Need immutability or thread safety for shared trees

**Real Use Cases:**
- `UIView` / `CALayer` hierarchy - Composite rendering tree
- SwiftUI `View` structs (`VStack`, `Group`, etc) - Pure functional composite
- JSON parsing - Recursive tree structure with nodes and arrays
- SceneKit / SpriteKit - Composite node graph for rendering and input

### Interview Questions
Design a chat thread where messages can have replies, which can in turn have replies
Explain:
- `Message` protocol
- `TextMessage` (leaf), `ThreadedMessage` (composite)
- Use recursion for render and nesting

---
## Bridge
The Bridge Pattern decouples an abstraction from its implementation independently. Think of it as splitting a giant inheritance hierarchy into two orthogonal dimensions:

![[Bridge Pattern.excalidraw]]

You create:
- Abstraction side - the API you expose to clients.
- Implementation side - interchangeable concrete workers behind the scenes.

**Use Cases:**

| Problem                                                                     | Benefit                                                                            |
| --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Support both UIKit and SwiftUI for a design-system component library        | One abstraction (`Button`), two implementations (`UIKitButton`, `SwiftUIButton`)   |
| Ship the same feature with local SQLite on-device but CloudKit on iCloud    | Toggle implementation without touching business logic                              |
| Allow multiple payment gateways (Razorpay, Stripe, Apple Pay) under one API | Swap implementations per region or feature flag                                    |
| Render map view with Apple Maps or Google Maps based on user preference     | Abstraction (`MapView`), implementations (`AppleMapsAdapter`, `GoogleMapsAdapter`) |

**Example - MapView Bridge:**
```swift
/// Implementation Protocol
protocol MapRenderer {       // Implementation layer
	func setCenter(lat: Double, lon: Double)
	func setMarker(lat: Double, lon: Double)
	func present(on view: UIView)
}

/// Concrete Implementations
final class AppleMapRenderer: MapRenderer {
	private let map = MKMapView()

	func setCenter(lat: Double, lon: Double) {
		map.setCenter(CLLocationCoordinate2D(latitude: lat, longitude: lon), animated: false)
	}

	func addMarker(lat: Double, lon: Double, title: String) {
		let pin = MKPointAnnotation()
		pin.coordinate = .init(latitude: lat, longitude: lon)
		pin.title = title
		map.addAnnotation(pin)
	}

	func present(on view: UIView) {
		map.frame = view.bounds
		view.addSubview(map)
	}
}

final class GoogleMapRenderer: MapRenderer {
	private let map = GMSMapView()

	func setCenter(lat: Double, lon: Double) {
		map.camera = GMSCameraPosition(latitude: lat, longitude: lon, zoom: 12)
	}

	func addMarker(lat: Double, lon: Double, title: String) {
		let marker = GMSMarker(position: .init(latitude: lat, longitude: lon))
		marker.title = title
		marker.map = map
	}

	func present(on view: UIView) {
		map.frame = view.bounds
		view.addSubview(map)
	}
}

/// Abstraction Layer
class MapView {         // Abstraction
	private let renderer: MapRenderer

	init(renderer: MapRenderer) {
		self.renderer = renderer
	}

	func showBangalore() {
		renderer.setCenter(lat: 12.9716, lon: 77.5946)
	}

	func addEatStreetMarker() {
		renderer.addMarker(lat: 12.971781, ;pn: 77.641151, title: "Indiranagar Eat Street")
	}

	func attach(to view: UIView) {
		renderer.present(on: view)
	}
}

/// Usage
let renderer: MapRenderer = useGoogleMaps ? GoogleMapRenderer() : AppleMapRenderer()

let mapView = MapView(renderer: renderer)
mapView.showBangalore()
mapView.addEatStreetMarker()
mapView.attach(to: self.view)
```

**Pros:**
- Separate release cycles for abstraction & implementation
- Easy A/B or region‑based toggling
- Test the abstraction with stub implementation
- Combines well with DI + Strategy for runtime swaps

**Cons:**
- Slight indirection cost (negligible)
- Extra boilerplate (solve with protocols + default extensions)
- If both hierarchies grow unchecked, still can bloat, keep interfaces lean
- Be mindful of feature drift between implementation

### Interview Questions
- Design a video player that can output via AVPlayer on iOS and VLCCore on macOS - Use Bridge
- We need themeable components that render with CoreGraphics offline and Metal online - Bridge the renderer
- How to share a networking layer between watchOS and iOS, but use URLSession vs SessionActor - Bridge the transport

---
## Flyweight
The Flyweight Pattern is a structural design pattern used to minimize memory usage by sharing as much data as possible between similar objects. It's ideal for scenarios where you have many objects that share common state, and only a small portion of their state is unique
>Share intrinsic state, externalize what varies

| Term              | Meaning                                                 |
| ----------------- | ------------------------------------------------------- |
| Flyweight         | Shared object containing intrinsic (shared) data        |
| Context / Client  | Provides extrinsic (per-instance) data to the flyweight |
| Flyweight Factory | Reuses or creates new flyweights as needed              |

**Use Cases:**

| Scenario                      | Application                                                     |
| ----------------------------- | --------------------------------------------------------------- |
| Emoji renderer in chat app    | All emojis share glyph and vector data                          |
| Map oin system                | Pins share icon, size, animations; only lat/lon differs         |
| Attributed text rendering     | Fonts, colors, and styles shared across runs                    |
| List of seats in a cinema app | Seat type & color as flyweights; only ID & status are unique    |
| Document editor               | Characters share font metrics; only position and content differ |

**Example - Character Rendering Template:**
```swift
/// Flyweight
struct GlyphFlyweight {
	let character: Character
	let font: UIFont
	let color: UIColor

	func draw(at point: CGPoint) {
		let attr: [NSAttributedString.Key: Any] = [
			.font: font,
			.foregroundColor: color
		]
		let str = NSAttributedString(string: String(character), attributes: attr)
		str.draw(at: point)
	}
}

/// Flyweight Factory
final class GlyphFactory {
	static let shared = GlyphFactory()

	private var pool: [String: GlyphFlyweight] = [:]  // key = char + font name + color hex

	func flyweight(for characer: Character, font: UIFont, color: UIColor) -> GlyphFlyweight {
		let key = "\(character)-\(font.fontName)-\(color.hex)"
		if let cached = pool[key] {
			return cached
		}

		let newGlyph = GlyphFlyweight(character: character, font: font, color: color)
		pool[key] = newGlyph
		return newGlyph
	}
}

/// Client
struct RenderedGlyph {
	let flyweight: GlyphFlyweight
	let position: CGPoint
	
	func render() {
		flyweight.draw(at: position)
	}
}

/// Usage
let font = UIFont.systemFont(ofSize: 14)
let color = UIColor.label

let sentence = "Hello Bangalore"
let rendered: [RenderedGlyph] = []

for (index, ch) in sentence.enumerated() {
	let fly = GlyphFactory.shared.flyweight(for: ch, font: font, color: color)
	let pos = CGPoint(x: index * 10, y: 50)
	rendered.append(RenderedGlyph(flyweight: fly, position: pos))
}

for glyph in rendered {
	glyph.render()
}
```

**Pros:**
- Massive memory savings - Especially in large UI lists, renderers, maps
- Better performance - Fewer allocations, smaller GC/memory churn
- Enables object pooling - Reuse same model for read-only/shared cases

**Cons:**
- Harder to debug - Tooling eg. print flyweight cache size
- Inflexible if shared state becomes mutable - Strictly enforce immutability for flyweights
- Context-dependent logic - Must pass all extrinsic data correctly each time

**Analogy:**
>Fonts in iOS: A single `UIFont` object can be reused across thousands of text draws. You wouldn't create a new font for each word, you reuse the flyweight

### Interview Questions
Design a seat map for a cinema hall where each seat has a type (Premium, Regular, Couple), but only position and booking status are unique.
Breakdown:
- Flyweight: SeatType (icon, color, pricing)
- Extrinsic: seatID, row/column, isBooked
- Factory: `SeatTypeFactory`