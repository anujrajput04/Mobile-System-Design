## Redux Architecture
Redux is a **predictable state container** and architectural pattern based on **unidirectional data flow**, originally from JavaScript but widely applicable, including in iOS using Swift or SwiftUI
>A single immutable state tree, changed only via pure reducers, driven by actions

Redux fits well when managing complex UI, background sync, error states, and offline-first flows.

![[Redux.excalidraw]]

| Component             | Description                                                   |
| --------------------- | ------------------------------------------------------------- |
| State                 | The single source of truth for you app/screen                 |
| Action                | A plain value describing *what happened*                      |
| Reducer               | A pure function `(State, Action) -> State`                    |
| Store                 | Holds current state, dispatches actions, notifies subscribers |
| Middleware (optional) | Intercepts actions runs side effects like API calls           |

**Data Flow:**
```
User → View → dispatch(Action)
                   ↓
               Reducer
                   ↓
             New Immutable State
                   ↓
              View Re-renders
```

**Example - Login Flow:**
```swift
/// State
struct LoginState {
	var email: String = ""
	var password: String = ""
	var isLoading: Bool = false
	var isLoggedIn: Bool = false
	var errorMessage: String?
}

/// Actions
enum LoginAction {
	case emailChanged(String)
	case passwordChanged(String)
	case loginTapped
	case loginSucceeded
	case loginFailed(String)
}

/// Reducer
func loginReducer(state: LoginState, action: LoginAction) -> LoginState {
	var newState = state
	switch action {
		case .emailChanged(let email):
			newState.email = email
		case .passwordChanged(let password):
			newState.password = password
		case .loginTapped:
			newState.isLoading = true
			newState.errorMessage = nil
		case .loginSucceeded:
			newState.isLoading = false
			newState.isLoggedIn = true
		case .loginFailed(let message):
			newState.isLoading = false
			newState.errorMessage = message
	}

	return newState
}

/// Store
final class Store: ObservableObject {
	@Published private(set) var state: LoginState
	private let reducer: (LoginState, LoginAction) -> LoginState

	init(initialState: LoginState, reducer: @escaping (LoginState, LoginAction) -> LoginState) {
		self.state = initialState
		self.reducer = reducer
	}

	func dispatch(_ action: LoginAction) {
		state = reducer(state, action)
	}
}

/// SwiftUI View
struct LoginView: View {
	@ObservedObject var store: Store

	var body: some View {
		VStack {
			TextField("Email", text: Binding(
				get: { store.state.email },
				set: { store.dispatch(.emailChanged($0)) }
			))
			SecureField("Password", text: Binding(
				get: { store.state.password },
				set: { store.dispatch(.passwordChanged($0)) }
			))
			if let error = store.state.errorMessage {
				Text(error).foregroundColor(.red)
			}

			Button("Login") {
				store.dispatch(.loginTapped)
				Task {
					// simulate async call
					let success = await FakeAPI.login(email: store.state.email, password: store.state.password)
					if success {
						store.dispatch(.loginSucceeded)
					} else {
						store.dispatch(.loginFailed("Invalid credentials"))
					}
				}
			}
		}
	}
}

/// Middleware
/// Used for async or side-effectful logic (networking, logging, analytics):
typealias Middleware = (LoginState, LoginAction) -> Void

// Logging
let logger: Middleware = { state, action in
	print("Dispatched action: \(action)")
}
```

**Pros:**
- Predictability - State changes are traceable and reproducible
- Centralized state - No scattered local view states
- Testable - Reducers are pure functions, easy to test
- Undo/Redo support - Easy to implement via state snapshots
- Debuggable - Every action is logged and replayable

**Cons:**
- Boilerplate-heavy - Especially for simple features
- Verbose for trivial state changes - Every field update is an action
- Async flows (thunks/middleware) add complexity - Needs extra structuring or libraries
- Overkill for small projects - Not ideal for tiny UIs or local-only state

**Use Cases:**
- Apps with complex UI state flows - Excellent
- Background sync or offline-first - Centralized control
- Real-time feeds or websocket data - Predictable state handling
- Shared state across many screens - Global store works well
- Apps build using SwiftUI + Combine - Swift syntax fits well