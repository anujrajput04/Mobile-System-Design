## Reactive Programming
**Reactive Programming** is a declarative paradigm where code reacts automatically to data changes, asynchronous events, or time-based updates over a stream. Instead of asking *what's the value now*, your code says *do this when the value changes*. It fits well for user interactions, network responses, form validations, or UI updates
>It's about pushing data, not pulling it

| Concept                | Description                                         |
| ---------------------- | --------------------------------------------------- |
| Observable / Publisher | Emits a stream of values over time                  |
| Observer / Subscriber  | Listens and reacts to emitted values                |
| Operators              | Transform, filter, merge or combine streams         |
| Schedulers             | Manage threading: background vs main queue          |
| Subjects               | Both observable and observer - can emit and receive |
**Libraries:**

| Library       | Description                                 |
| ------------- | ------------------------------------------- |
| Combine       | Native framework since iOS 13               |
| RxSwift       | Mature third-party library inspired by RxJS |
| ReactiveSwift | From ReactiveCocoa ecosystem                |
| SwiftUI       | Built on top of Combine                     |

**Example - SwiftUI & Combine:**
```swift
/// Simple Publisher + Subscriber
import Combine

let namePublisher = Just("Anuj")
let subscription = namePublisher
	.sink { value in 
		print("Received:", value)
	}

/// TextField Reactive Validation (Combine + UIKit)
class ViewModel {
	@Published var email: String = ""
	var isValid: AnyPublisher<Bool, Never> {
		$email
			.map { $0.contains("@") }
			.eraseToAnyPublisher()
	}
}

/// Debounce API Call
$searchText
	.debounce(for: .milliseconds(300), scheduler: RunLoop.main)
	.removeDuplicates()
	.sink { query in
		apiService.search(query)
	}
```

**Pros:**
- Declarative - Define what should happen, not how
- Composability - Combine data sources and operators fluently
- Handles async naturally - Streams, network, UI events, timers, etc
- Less state & fewer bugs - Reduces manual binding/update mess
- Powerful operators - Filter, merge, retry, zip, combineLatest, etc

**Cons:**
- Steep learning curve - Complex syntax and mental model
- Harder debugging - Stack traces are abstract
- Overuse risk - Unnecessary streams add complexity
- Retain cycles - Must manage `sink`, `assign`, or `weak self` carefully

**When to use:**

| Scenario                     | Fit                                      |
| ---------------------------- | ---------------------------------------- |
| Live form validation         | Great use case                           |
| Text input search throttling | Common with `debounce()`                 |
| Multi-source data binding    | With `combineLatest()`                   |
| WebSocket or polling updates | Stream-based design fits well            |
| Analytics / event tracking   | Reactive tap pipelines                   |
| SwiftUI state propagation    | Built in via `@State`, `@Published`, etc |

**Combine Operators Cheatsheet:**

| Operator           | Usage                              |
| ------------------ | ---------------------------------- |
| `map`              | transform values                   |
| `filter`           | drop values not meeting condition  |
| `debounce`         | delay to prevent spam (eg. typing) |
| `combineLatest`    | merge multiple publishers          |
| `flatMap`          | chain async publishers             |
| `removeDuplicates` | ignore repeating values            |
| `retry`            | reattempt on failure               |

**Example - Combine + SwiftUI:**
```swift
struct ContentView: View {
	@ObservedObject var vm = WeatherViewModel()

	var body: some View {
		VStack {
			TextField("City", text: $vm.city)
			Text(vm.weatherInfo)
		}
	}
}

final class WeatherViewModel: ObservableObject {
	@Published var city: String = ""
	@Published var weatherInfo: String = ""

	private var cancellables = Set<AnyCancellable>()

	init() {
		$city
			.debounce(for: .milliseconds(400), scheduler: RunLoop.main)
			.removeDuplicates()
			.sink { [weak self] city in
				self?.fetchWeather(for: city)
			}
			.store(in: &cancellables)
	}

	func fetchWeather(for city: String) {
		// API Call → update weatherInfo
	}
}
```

**Reactive vs Imperitive:**

| Imperative                    | Reactive                          |
| ----------------------------- | --------------------------------- |
| `if changed { do something }` | `on change → automatically react` |
| Pull-based                    | Push-based                        |
| Manual binding                | Auto propagation                  |
| Lots of boilerplate           | Fluent, concise streams           |