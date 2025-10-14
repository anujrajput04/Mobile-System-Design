Behavioral Design Patterns focus on how objets interact and communicate with each other. These patterns:
- Define the flow of control
- Delegate responsibility between objects
- Make object interactions flexible, decoupled, and dynamic

>It's not about how objects are created (Creational) or structured (Structural) but how they behave together.

| Area                        | Behavioral Pattern Use                     |
| --------------------------- | ------------------------------------------ |
| MVC/MVVM/MVI                | Delegation, Observers, State               |
| App-level architecture      | Strategy, Command, Chain of Responsibility |
| UI Event Handling           | Responder Chain, Observer                  |
| User Preferences / Settings | Memento, Command, State                    |
| Feature Flag Systems        | Strategy, Interpreter                      |

---
## Observer
The Observer Pattern is a Behavioral Design Pattern where an object (Subject) maintains a list of dependents (Observers) and notifies them automatically of state changes
>"When one object changes, all its dependents are updated"

- Subject - Holds the state, notifies observers of changes
- Observer - Reacts to updates pushed by the subject

**User Cases:**
- UI updates on data change - TableView reloads on model update
- NotificationCenter - Observers listen for global/system events
- Combine/ReactiveCocoa/RxSwift - Declarative, reactive observation pipelines
- KVO (Key-Value Observing) - Observing properties in UIKit/Core Data
- SwiftUI - Uses `@Observable`, `@State`, `@Published` (built-in observer mechanisms)

**Example - Manual Observer Implementation:**
```swift
// Define Observer Protocol
protocol Observer: AnyObject {
	func didUpdate(message: String)
}

// Subject Class
class MessagePublisher {
	private var observers = [Observer]()

	func addObserver(_ observer: Observer) {
		observers.append(observer)
	}

	func removeObserver(_ observer: Observer) {
		observers.removeAll { $0 === observer }
	}

	func notifyObservers(with message: String) {
		for observer in observers {
			observer.didUpdate(message: message)
		}
	}

	func updateMessage(_ msg: String) {
		print("MessagePublisher updated message to: \(msg)")
		notifyObservers(with: msg)
	}
}

// Concrete Observer
class Logger: Observer {
	func didUpdate(message: String) {
		print("Logger received message: \(message)")
	}
}

// Usage
let publisher = MessagePublisher()
let logger = Logger()

publisher.addObserver(logger)
publisher.updateMessage("Hello Observer!)
```

**Real World Examples:**
- `NotificationCenter` - Observers register for system events (`keyboardWillShow`)
- `Combine` - `@Published`, `sink`, `assign`, etc
- `KVO` - `observer(\.property)` for property changes
- `SwiftUI` - View observes state via `@ObservedObject`, `@EnvironmentObject`, etc
- `CoreData` - `NSFetchedResultsController` notifies of DB changes

**Pros:**
- Decouples sender and receiver
- Allows multiple observers for the same subject
- Easy to extend without touching existing code
- Supports event driven design

**Cons:**
- Risk of retain cycles, especially in closures or NotificationCenter
- Can become hard to debug in large systems with many observers
- Manual observer management needs explicit add/remove
- Overuse leads to tight coupling on message/event formats

### Interview Questions
- Design a notification system where multiple views react to shared model changes
- How would you update multiple modules when a user logs in or logs out?
- Can you show how NotificationCenter or Combine follows Observer pattern?

---
## Chain of responsibility
The Chain of Responsibility is a Behavioral Design Pattern that lets you pass requests along a chain of handlers. Each handler decides wheter to process the request or pass it to the next handler in the chain.
>"Who can handle this?" Not me - next in line"

- Each handler has a reference to the next handler
- The request travels down the chain until some handler processes it (or none do)
- Helps in decoupling the sender and receiver of the request

**Use Cases:**
- Event propagation in views - UIKit `responder chain` passes events from child to parent
- Middleware pipelines - Logging 
Auth → Validation → Retry
- Request handling - Chaining filters/interceptors for API requests
- UI gesture recognizers - Chain of responder views (eg. `touchesBegan`)
- Custom error handling - Delegating to the next error resolver if one fails

**Example - Middleware for API Request:**
```swift
// Define Handler Protocol
protocol RequestHandler: AnyObject {
	var next: RequestHandler? { get set }
	func handle(request: URLRequest)
}

// Banse Handler with Default Chaining
extension RequestHandler {
	func passToNext(request: URLRequest) {
		next?.handle(request: request)
	}
}

// Concrete Handlers
class AuthHandler: RequestHandler {
	var next: RequestHandler?

	func handle(request: URLRequest) {
		if request.url?.path.contains("/secure") == true {
			print("AuthHandler: Authenticated request")
			passToNext(request: request)
		} else {
			print("AuthHandler: Skipping auth for public request")
			passToNext(request: request)
		}
	}
}

class LoggingHandler: RequestHandler {
	var next: RequestHandler?

	func handle(request: URLRequest) {
		print("LoggingHandler: Request to \(request.url?.absoluteString ?? "")")
		passToNext(request: request)
	}
}

class FinalHandler: RequestHandler {
    var next: RequestHandler?

    func handle(request: URLRequest) {
        print("FinalHandler: Executing network call to \(request.url?.absoluteString ?? "")")
    }
}

// Setup Chain
let auth = AuthHandler()
let logger = LoggingHandler()
let finalHandler = FinalHandler()

auth.next = logger
logger.next = finalHandler

let secureRequest = URLRequest(url: URL(string: "https://api.app.com/secure/data")!)
auth.handler(request: secureRequest))
```

**Pros:**
- Clean separation of concerns
- Flexible reorderable chains (plug-in/out behavior)
- Easily extendable without modifying existing handlers
- Promotes open/closed principle

**Cons:**
- Can be hard to trace request flow in long chains
- Risk of request not being handled at all
- Requires each handler to be aware of the protocol/contract
- Debugging across handlers can be non trivial

### Interview Questions
- Design an extensible middleware pipeline for network requests
- Create a handler for different types of user input events
- How would you intercept requests and conditionally block or transform them

---
## Strategy
Strategy Pattern is a Behavioral Design Pattern that defines a family of algorithms, encapsulates each one, and makes them interchangeable. It lets the algorithm vary independently from clients that use it.
>Encapsulate what varies. Swap logic without touching client code

| Role             | Purpose                                 |
| ---------------- | --------------------------------------- |
| Context          | Uses a strategy to perform an operation |
| Strategy         | Defines a common interface              |
| ConcreteStrategy | Implements the logic                    |

**Use Case:**
- Feature toggles: Choose behavior at runtime
- Different sort/filter logic: Swap comparator dynamically
- Payment gateways: Stripe vs Razorpay vs Apple Pay
- Image loading libraries: URLSession, SDWebImage, Custom loader
- UI Layout logic: Grid vs List vs Staggered Layouts
- Validation logic: Name/Email/Password strategies

**Example - Sorting:**
```swift
// Define protocol
protocol SortingStrategy {
	func sort(_ items: [Int]) -> [Int]
}

// Concrete Strategies
class AscendingSort: SortingStrategy {
	func sort(_items: [Int]) -> [Int] {
		return items.sorted()
	}
}

class DescendingSort: SortingStrategy {
	func sort(_items: [Int]) -> [Int] {
		return items.sorted(by: >)
	}
}

// Context
class Sorter {
	private var strategy: SortingStrategy

	init(strategy: SortingStrategy) {
		self.strategy = strategy
	}

	func setStrategy(_ strategy: SortingStrategy) {
		self.strategy = strategy
	}

	func sort(_ items: [Int]) -> [Int] {
		return strategy.sort(items)
	}
}

// Usage
let ascendingSorter = Sorter(strategy: AscendingSort())
print(ascendingSorter.sort([5, 3, 1])) // [1, 3, 5]

let descendingSorter = Sorter(strategy: DescendingSort())
print(descendingSorter.sort([5, 3, 1])) // [5, 3, 1]
```

**Pros:**
- Eliminates conditionals (if-else, switch) for logic variations
- Makes algortihm interchangeable at runtime
- Makes the system open for extension, closed for modification
- Great for testing each strategy in isolation

**Cons:**
- Adds extra boilerplate (protocol + concrete types)
- Can become overkill if strategies are too simple
- Context must manage which strategy is set (external responsibility)

**Real World Use Cases:**
- API Environment - `ProductionEnvironmentStrategy`, `MockEnvironmentStrategy`
- Feature Flags - `ABTestingStrategy`, `DefaultStrategy`
- Layout Manager - `ListLayout`, `GridLayout`, `StaggeredLayout`
- Authentication - `BiometricAuth`, `PasswordAuth`, `OTPAuth`
### Interview Questions
- Design a flexible payment flow for multiple providers
- You have dynamic sort/filter logic across the app. How do you manage it?
- Support different validation strategies per screen. How would you implement that?

---
## Mediator
The Mediator Pattern is a behavioral design pattern that introduces a central coordinator to manage communication between multiple objects, keeping them from talking to each other directly
>Stop the group chat chaos - everyone DM the coordinator instead

**Roles:**
- Mediator: Central hub that routes or orchestrates requests between colleagues
- Colleague: Any class that wants to interact with others through the mediator

**Scenarios:**
- Coordinator pattern for navigation: A single coordinator directs screens, avoiding VC-to-VC coupling
- Complex feature modules: eg. Chat, ChatMediator routes messages, typing indicators, presence updates
- Decoupled feature toggles: ToggleMediator decides which modules should respond to a flag
- Form validation across fields: FormMediator coordinates field states and enables/disables submit button

**Example - Coordinator as Mediator:**
```swift
/// Protocol
protocol AuthCoordinator {
	func showLogin()
	func showSignup()
	func didLoginSuccessfully(user: User)
}

/// Concrete Mediator
final class AppAuthCoordinator: AuthCoordinator {
	private let navigationController: UINavigationController
	init(nav: UINavigationController) { self.navigationController = nav }

	func showLogin() {
		let loginVC = LoginViewController(coordinator: self)
		navigationViewController.pushViewController(loginVC, animated: true)
	}

	func showSignup() {
		let signupVC = SignupViewController(coordinator: self)
		navigationViewController.pushViewController(signupVC, animated: true)
	}

	func didLoginSuccess(user: User) {
		let dashboard: DashboardViewController(user: User)
		navigationViewController.setViewControllers([dashboard], animated: true)
	}
}

/// Colleague (LoginVC)
final class LoginViewController: UIViewController {
	weak var coordinator: AuthCoordinator?

	init(cordinator: AuthCoordinator) {
		self.coordinator = coordinator
		super.init(nibName: nil, bundle: nil)
	}

	required init?(coder: NSCoder) { fatalError() }

	@objc private func loginTapped() {
		// validate, call API, etc
		let user = User(id: 42, name: "Douglas")
		coordinator?.didLoginSuccess(user: user)
	}
}
```

**Result:**
- View Controllers never reference each other directly
- All flows are centralized in `AppAuthCoordinator`, simplifying testing and future refactors.

**Pros:**
- Loose coupling - Colleagues don't need to know each other
- Single source of truth for workflow logic
- Easier testing - Mediator can be swapped with a mock
- Clear separation of concerns - UI objects focus on UI, mediator handles orchestration

**Cons:**
- Mediator bloat - can turn into a god object if you dump all logic there
- Indirection overhead - simple interactions might become verbose
- Single point of failure - bugs in mediator break entire flow

**Ways to implement:**
1. Coordinator pattern - pure Swift class, often used for navigation
2. Combine subject or RxSwift `PublishedSubject` - mediator as a stream hub routing events
3. `Actor` based mediator in Swift Concurrency - actor owns the shared state and methods
4. Service bus - mediator as an in-memory event bus module
Pick based on complexity:
- Small flow → classic coordinator
- Heavy concurrency / many event types → actor or Combine hub

**When to use:**
- More than two objects need to collaborate **and** that collaboration changes frequently
- You’re seeing “ViewController A imports ViewController B” spaghetti
- Multiple feature modules must stay isolated but still coordinate (chat ↔ presence ↔ typing)

**Example - Actor-based Mediator for High-Volume Chat Streams:**
Scenario:
A Bangalore based food delivery app adds real-time group chat between customers, riders, and support. Each chat room must handle:
- Hundreds of concurrent message events
- Typing indicators and presence updates
- Spam throttling and analytics taps
- Strict main-thread isolation for UI rendering
Using one `ChatHub` actor as the Mediator keeps all peers decoupled while protecting shared state.

```swift
// Core Types
/// Messages flowing through the system
struct ChatEvent: Sendable {
	enum Kind { case text(String), typingStart, typingStop, presence(Bool) }
	let roomID: String
	let userID: String
	let kind: Kind
	let timestamp: UInt64 = DispatchTime.now().uptimeNanoseconds
}

/// Protocol every frontend module adopts (iOS view, watchdog bot, analytics sink)
protocol ChatSink: AnyObject, Sendable {
	func receive(_ event: ChatEvent) async
}


/// Weak wrapper for ChatSink
final class WeakSink {
    weak var value: (any ChatSink)?
    init(_ value: any ChatSink) { self.value = value }
}

// The Mediator: `ChatHub` Actor
actor ChatHub {
	// MARK: - Registrations
	private var sinks: [String: WeakSink] = [:]  // key = sinkID
	private var rooms: [String: Set<String>] = [:]     // roomID → sinkIDs

	/// Register any module that wants events
	func register(_ sink: ChatSink, for sinkID: String, rooms roomIDs: [String]) {
		sinks[sinkID] = WeakSink(sink)
		for id in roomIDs { rooms[id, default: []].insert(sinkID) }
	}

	/// Deregister when VC deallocates
	func deregister(sinkID: String) {
		sinks.removeValue(forKey: sinkID)
		for (room, members) in rooms {
			rooms[room] = members.subtracting([sinkID])
		}
	}

	// MARK: - Event Dispatch
	func post(_ event: ChatEvent) async {
		guard let listeners = rooms[event.roomID] else { return }
		await withTaskGroup(of: Void.self) { group in
			for sinkID in listeners {
				if let sink = sinks[sinkID]?.value {
					group.addTask { await sink.receive(event) }
				}
			}
		}
	}
}
```

Keys:
- Actor isolation ensures `sinks` and `rooms` mutate safely
- `withTaskGroup` fans out delivery concurrently for high throughput
- UI threads never touch shared mutable state

```swift
/// UI Module consuming the Mediator
final class ChatViewModel: ObservableObject, ChatSink {
	@MainActor @Published var messages: [ChatEvent] = []

	init(hub: ChatHub, roomID: String) {
		Task { await hub.register(self, for: UUID().uuidString, rooms: [roomID]) }
	}

	nonisolated func receive(_ event: ChatEvent) async {
		await MainActor.run { [weak self] in
			guard case .text = event.kind else { return }
			self?.messages.append(event)
		}
	}

	func send(text: String, hub: ChatHub, roomID: String, userID: String) {
		Task {
			let event = ChatEvent(roomID: roomID, userID: userID, kind: .text(text))
			await hub.post(event)
		}
	}
}
```

---
## Command
A Command encapsulates all the information needed to perform an action - the what, who, when, and how - inside an object
That object can be queued, logged, undone, retried, serialized, or executed later without the caller knowing any implementation details.
>Turn a method call into a data object

| Role            | Responsibility                                |
| --------------- | --------------------------------------------- |
| Command         | Declares an `execute()` interface             |
| ConcreteCommand | Stores receiver reference + action parameters |
| Receiver        | Knows how to perform the actual work          |
| Invoker         | Holds commands, decides when to run them      |
| Client          | Assembles everything and triggers execution   |

| Scenario                                                        | Command Advantage                                                                                        |
| --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Undo/Redo in drawing or note apps                               | Store `DrawStrokeCommand`, `EraseCommand`; push to Undo stack                                            |
| Offline job queue for creating expense transactions (Splitwise) | `CreateTransactionCommand`, `DeleteTransactionCommand` serialized to disk, replayed when network resumes |
| Analytics batching                                              | `TrackEventCommand` objects aggregated, compressed, flushed on WiFi                                      |
| In-app purchases                                                | Wrap "buy", "restore", "consume" into commands to support retry / rollback                               |
| Push-notification actions                                       | Map buttons to specific commands rather than giant switches                                              |

**Example - Offline Order Queue:**
```swift
/// Command Protocol
protocol OrderCommand: Codable {
	func execute(on receive: OrderService) async throws
}

/// Receiver - Real implementation
struct OrderService {
	func placeOrder(id: String, items: [String]) async throws { /* Implementation */ }
	func cancelOrder(id: String) async throws { /* Implementation */ }
}

/// Concrete Commands
struct PlaceOrder: OrderCommand {
	let id: String
	let items: [String]
	func execute(on receive: OrderService) async throws {
		try await receiver.placeOrder(id: id, items: items)
	}
}

struct CancelOrder: OrderCommand {
	let id: String
	func execute(on receive: OrderService) async throws {
		try await receiver.cancelOrder(id: id)
	}
}

/// Invoker - Command queue
actor OrderQueue {
	private var queue: [OrderCommand] = []
	private let service = OrderService()

	func add(_ cmd: OrderCommand) async {
		queue.append(cmd)
		try? await persist()
	}

	func flush() async {
		for cmd in queue {
			do {
				try await cmd.execute(on: service)
			} catch {
				// keep it in queue for retry
				continue
			}
			queue.removeFirst()
			try? await persist()
		}
	}

	private func persist() async throws {
		let data = try JSONEncoder().encode(queue)
		try data.write(to: URL.documents.appendingPathComponent("order.queue"))
	}
}
```

**Highlights:**
1. `OrderCommand` is *Sendable* and *Codable*. Can cross threads and app restarts
2. `OrderQueue` actor guarantees thread-safe, high concurrency handling
3. Subcommands are easy to extend eg. `UpdateOrderAddress`
4. Undo can be added by pairing each command with an inverse command

**Pros:**
- Decouples invoker from receiver logic
- Supports batching, retry, undo, logging, transactions
- Uniform "task" abstraction fits GCD, OperationQueue, actors
- Easy to compose higher order features eg. analytics, metrics, encryption

**Cons:**
- More classes, hide complexity behind protocol + default implementation
- Needs persistence and versioning if commands change shape
- Overkill for trivial actions, stick to closures instead
- Requires disciplined error handling so queue doesn't stall

**Variations:**
1. Macro Command - bundles multiple commands (useful for "place order + send SMS + log analytics")
2. Asynchronous Command - returns a `Task`/`Future` for integration with Swift Concurrency
3. Reactive Command - exposes Combine publisher so UI can bind enabled/disabled state

### Interview Questions
- "Walk me through undo/redo for a rich-text editor". Show command objects + dual stacks (`undo`, `redo`)
- "Design network resilient operations for a metro ticketing app". Explain serialization, replay, explonential back-off in the invoker
- "How would you add auditing to critical flows without touching business code". Wrap every command in `LoggingDecorator` that records before/after states

---
## Memento
The Memento Pattern is a behavioral design pattern that allows you to capture and restore an object's state without exposing its internal structure
>Snapshot now, revert later

| Role       | Responsibilit                                       |
| ---------- | --------------------------------------------------- |
| Originator | The object whose state you want to save and restore |
| Memento    | A snapshot of the Originator's state (immutable)    |
| Caretaker  | Stores and manages mementos, doesn't interpret them |

| Use Case                                   | Role                                 |
| ------------------------------------------ | ------------------------------------ |
| Undo/Redo in drawing or editing apps       | Save canvas/document state           |
| Restore draft state in forms or emails     | Save fields into a memento           |
| Save/Restore view state on background/kill | Use `Codable` to persist UI state    |
| Onboarding flow checkpoints                | Allow user to return to prior step   |
| Game saves / Level checkpoints             | Snapshot player state and game world |

**Example - Data Form Recovery:**
```swift
/// Originator
struct AddressFormModel {
	var name: String
	var address: String
	var pincode: String

	func createSnapshot() -> AddressFormSnapshot {
		return AddressFormSnapshot(name: name, address: address, pincode: pincode)
	}

	mutating func restore(from snapshot: AddressFormSnapshot) {
		self.name = snapshot.name
		self.address = snapshot.address
		self.pincode = snapshot.pincode
	}
}

/// Memento - Snapshot struct
struct AddressFormSnapshot: Codable {
	let name: String
	let address: String
	let pincode: String
}

/// Caretaker - Draft Saver
struct FormDraftManager {
	private let fileURL = URL.documents.appendingPathComponent("address_form.json)

	func save(_ snapshot: AddressFormSnapshot) {
		let data = try? JSONEncoder().encode(snapshot)
		try? data?.write(to: fileURL)
	}

	func restore() -> AddressFormSnapshot? {
		guard let data = try? Data(contentsOf: fileURL) else { return nil }
		return try? JSONDecoder().decode(AddressFormSnapshot.self, from: data)
	}

	func clear() {
		try? FileManager.default.removeItem(at: fileURL)
	}
}

/// Usage
var form = AddressFormModel(name: "Douglas", address: "Galaxy Far Away", pincode: "424242")
let draftManager = FormDraftManager()

/// Save draft
let snapshot = form.createSnapshot()
draftManager.save(snapshot)

/// Later: Restore
if let saved = draftManager.restore() {
	form.restore(from: saved)
}
```

**Real Use Cases:**
- UITextField autosave - memento for keyboard dismissal restore
- SwiftUI undoable state - wrap model in `@StateObject`, add `undo()` using memento
- Core Data draft edits - temporary in-memory memento object
- Game pause/resume - store game engine state in JSON memento

**Pros:**
- State saved without violating encapsulation
- Enables undo, rollback, autosave, versioning
- Works well with Codable and SwiftUI's State

**Cons:**
- Caretaker must manage memory eg. too many snapshots = bloat
- If originator has complex nested state, memento grows large
- Versioning: if structure changes, old mementos may break decoding

### Interview Questions
"Design a form that lets users discard changes and revert to the last saved state"
Answer flow:
- Define a `FormModel` as originator
- Capture a snapshot using `createSnapshot`
- Store to disk or memory in a Caretaker
- User `restore(from:)` on cancel

---
## Interpreter
The Interpreter Pattern defines a grammar and uses a structure of classes (one per grammar rule) to interpret sentences in that grammar.
>Model a language as a tree of expressions and evaluate it

It is a behavioral design pattern typically used for parsing and evaluating simple domain specific languages (DSL)

| Role                  | Responsibility                                  |
| --------------------- | ----------------------------------------------- |
| AbstractExpression    | Defines the `interpret()` interface             |
| TerminalExpression    | implemnents atomic grammar eg. literal          |
| NonTerminalExpression | Combines expressions eg. AND, OR, NOT           |
| Context               | Holds state like variables, input, symbol table |

**Where it's used in iOS:**

| Use Case                                       | Role of Interpreter                                         |
| ---------------------------------------------- | ----------------------------------------------------------- |
| Custom filtering DSLs in Notes/Tasks app       | Parse and apply filter rules                                |
| Search query parsing (`title:foo AND tag:bar`) | Convert to NSPredicate or SQL                               |
| Feature flag conditions                        | `region == "IN" AND version >= 2.0`                         |
| Animation timeline DSL                         | `move(10).rotate(90).fadeOut()`                             |
| UI Testing DSLs                                | Parse user steps from text eg. `tap "Login"`, `type "Anuj"` |

**Example - Basic Boolean Expression Parser:**
```swift
/// Define Protocol
protocol Expression {
	func interpret(context: [String: Bool]) -> Bool
}

/// Terminal Expressions
struct Variable: Expression {
	let name: String
	func interpret(context: [String: Bool]) -> Bool {
		return context[name] ?? false
	}
}

/// Non Terminal Expressions
struct And: Expression {
	let left: Expression
	let right: Expression

	func interpret(context: [String: Bool]) -> Bool {
		return left.interpret(context: context) && right.interpret(context)
	}
}

struct Or: Expression {
	let left: Expression
	let right: Expression

	func interpret(context: [String: Bool]) -> Bool {
		return left.interpret(context: context) || right.interpret(context)
	}
}

struct Not: Expression {
    let expr: Expression
    func interpret(context: [String: Bool]) -> Bool {
        return !expr.interpret(context: context)
    }
}

/// Usage - Evaluate DSL Rule
let expr = And(
	left: Variable(name: "isLoggedIn"),
	right: Not(expr: Variable(name: "isBanned"))
)

let context: [String: Bool] = [
	"isLoggedIn": true,
	"isBanned": false
]

let result = expr.interpret(context: context) // true
```

**Pros:**
- Encapsulates grammar rules as classes
- Adds extensibility for new operations
- Enables custom DSLs without parser generator
- Promotes separation between parsing and execution

**Cons:**
- Results in many small classes - complex for large grammars
- Slow for large inputs. Recursive evaluation per node
- Hard to debug due to nested evaluations

**Real World iOS Examples:**

| Area                 | Interpreter Role                                  |
| -------------------- | ------------------------------------------------- |
| Spotlight search     | Interpret text rules for filtering                |
| Siri commands        | Parsing voice to intent trees under the hood      |
| Feature gating       | Rules like `user.age > 25 AND app.version >= 2.0` |
| Custom animation DSL | Parse + run sequences using expressions           |

### Interview Questions
You are given a string like `isLoggedIn AND NOT isBanned` - how would you evaluate it?
Break down:
- Tokenization
- AbstractExpression tree
- Context as dictionary
- Recursive evaluation

---
## Visitor
The Visitor Pattern lets you define new operations on a group of objects without changing their classes. You do this by letting a Visitor object "visit" each element and perform work based on its type.
>Add new behavior to existing class hierarchies without modifying them

It's a behavioral design pattern useful when you need to perform unrelated operations across a fixed structure.

| Role            | Purpose                                              |
| --------------- | ---------------------------------------------------- |
| Element         | Base type in your object structure (e.g. Node, View) |
| ConcreteElement | Actual class that accepts a visitor                  |
| Visitor         | Declares visit methods for each concrete element     |
| ConcreteVisitor | Implements different logic per element               |

**iOS Use Cases:**

| Use Case                              | Visitor Role Example                            |
| ------------------------------------- | ----------------------------------------------- |
| Analytics/Event Logging across views  | Visitor logs data without polluting view logic  |
| AST traversal in custom interpreters  | Visitor interprets or prints DSL nodes          |
| Document export (PDF, HTML, Markdown) | Visitor generates different output formats      |
| UI validation                         | Visitor checks each view's rules recursively    |
| Code generation / form builders       | Visitor walks a schema to build SwiftUI/UIViews |

**Example - Form Field Visitor:**
```swift
/// Element Protocol
protocol FormField {
	func accept(visitor: FormFieldVisitor)
}

/// Concrete Elements
struct TextField: FormField {
	let label: String
	let value: String

	func accept(visitor: FormFieldVisitor) {
		visitor.visit(textField: self)
	}
}

struct SwitchField: FormField {
	let label: String
	let isOn: Bool

	func accept(visitor: FormFieldVisitor) {
		visitor.visit(switchField: self)
	}
}

/// Visitor Protocol
protocol FormFieldVisitor {
	func visit(textField: TextField)
	func visit(switchField: SwitchField)
}

// Concrete Visitor - Validation
final class FieldValidator: FormFieldVisitor {
	var errors: [String] = []

	func visit(textField: TextField) {
		if textField.value.trimmingCharacters(in: .whitespace).isEmpty {
			errors.append("'\(textField.label)' cannot be empty")
		}
	}

	func visit(switchField: SwitchField) {
		if !switchField.isOn {
			errors.append("'\(switchField.label)' must be turned on")
		}
	}
}

/// Usage
let fields: [FormField] = [
	TextField(label: "Name", value: ""),
	SwitchField(label: "Terms & Conditions", isOn: false)
]

let validator = FieldValidator()
fields.forEach { $0.accept(visitor: validator) }

print(validator.errors) // ["'Name' cannot be empty", "'Terms & Conditions' must be turned on"]
```

**Pros:**
- Adds operations without modifying original classes
- Encourages Open/Closed Principle
- Ideal for batch operations eg. report generation, schema export
- Centralizes logic that's scattered across types

**Cons:**
- Breaks encapsulation - visitor can access internal fields
- Tight coupling - new element types require updating all visitors
- Verbose - requires a lot of boilerplate for each new type
- Best when element set is stable, but operations are variable

### Interview Questions
Imagine a schema with many form fields, and you want to add different behavior like validation, rendering, or analytics without changing the field types.
- Model each field as an `Element`
- Define a Visitor for each concern
- Plug in different visitors based on mode: validate, log, preview, save, etc