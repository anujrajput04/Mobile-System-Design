## Model-View-Intent Architecture
**MVI** is a **unidirectional data flow** architecture where:
- The **View** renders a state
- The user triggers **Intents** (**Actions**)
- The **Model** (or **Reducer**) processes them into a new **State**
- The **View observes State** and re-renders

>Think: `Intent → Model → State → View`
>The cycle repeats on every user interaction

![[MVI.excalidraw]]


![[MVI.webp]]

**Principles:**

| Concept             | Description                                              |
| ------------------- | -------------------------------------------------------- |
| Unidirectional      | Data flows in one direction, preventing state confusion  |
| Immutable State     | Each UI screen is a function of a single source of truth |
| Explicit Intents    | All user actions are well defined and traceable          |
| Pure State Reducers | Logic produces a new state from old state + intent       |

**Components:**

| Component        | Role                                               |
| ---------------- | -------------------------------------------------- |
| View             | Renders state, forwards user actions as `Intent`s  |
| Intent           | User event or system-triggered event               |
| Model / Reducer  | Processes intent, computes new `State`             |
| State            | Entire snapshote of screen-specific UI state       |
| Store (optional) | Holds current state, dispatches intents to reducer |

**Example - Login Screen:**
```swift
/// Intent
enum LoginIntent {
	case emailChanged(String)
	case passwordChanged(String)
	case submitTapped
}

/// State
struct LoginState {
	var email: String = ""
	var password: String = ""
	var isLoading: Bool = false
	var errorMessage: String? = nil
	var isLoggedIn: Bool = false
}

/// Reducer
func reduce(state: LoginState, intent: LoginIntent) -> LoginState {
	var newState = state
	switch intent {
		case .emailChanged(let email):
			newState.email = email
			newState.errorMessage = nil
		case .passwordChanged(let password):
			newState.password = password
			newState.errorMessage = nil
		case .submitTapped:
			if state.email.isEmpty || state.password.isEmpty {
				newState.errorMessage = "Missing credentials"
			} else {
				newState.isLoading = true
				// async login simulation can trigger new intents
			}
	}
	return newState
}

/// ViewModel (Store)
@MainActor
final class LoginViewModel: ObservableObject {
	@Published private(set) var state = LoginState()

	func send(intent: LoginIntent) {
		state = reduce(state: state, intent: intent)
	}
}

/// SwiftUI View
struct LoginView: View {
	@StateObject var viewModel = LoginViewModel()

	var body: some View {
		VStack {
			TextField("Email", text: Binding(
				get: { viewModel.state.email },
				set: { viewModel.send(intent: .emailChanged($)) }
			))

			SecureField("Password", text: Binding(
				get: { viewModel.state.password },
				set: { viewModel.send(intent: .passwordChanged($)) }
			))

			if let error = viewModel.state.errorMessage {
				Text(error).foregroundColor(.red)
			}

			Button("Login") {
				viewModel.send(intent: .submitTapped)
			}
			.disabled(viewModel.state.isLoading)
		}
	}
}
```

**Pros:**
- Predictable UI - View is pure function of state
- Easily testable - Reducers are pure, no side effects
- No race conditions - State changes flow linearly
- Error handling is explicit - Error = state field, not exception

**Cons:**
- Boilerplate - Use codegen or structured enums
- Async logic hard to inline - Use effect handlers / middlerware
- Can over-engineer small screens - Apply only where complexity justifies

**Use Cases:**
- Authentication flows
- Form validation
- Complex UI states (load → error → data)
- Cross-screen state syncing
- Offline-first screens (sync/retry)