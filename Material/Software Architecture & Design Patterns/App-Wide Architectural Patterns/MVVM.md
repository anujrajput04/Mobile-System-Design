## Model-View-ViewModel Architecture
**MVVM** separates concerns by introducing **ViewModel** between the View and Model, designed to transform model data into UI-ready state. It enables data binding, reactive UI updates, and testable business logic.
>ViewModel is the translator between business logic and UI

| Component | Role                                                                        |
| --------- | --------------------------------------------------------------------------- |
| Model     | Business logic, data layer, networking, persistence                         |
| View      | UI layer (SwiftUI `View` or UIKit `UIViewController`)                       |
| ViewModel | UI logic holder - transforms raw data to UI state, handles user interaction |

**Data Flow (Unidirectional):**
```
View  →  ViewModel  →  Model
			↓       ↑
		  Render   Notify
		    ↓
		  View
```

![[MVVM.excalidraw]]

**Example - Login Screen:**
```swift
/// Model
struct UserCredentials {
	let email: String
	let password: String
}

protocol AuthService {
	func login(_ credentials: UserCredentials) async throws -> Bool
}

/// ViewModel
@MainActor
final class LoginViewModel: ObservableObject {
	@Published var email = ""
	@Published var password = ""
	@Published var isLoading = ""
	@Published var errorMessage: String? = nil
	@Published var isLoggedIn = false

	private let authService: AuthService

	init(authService: AuthService) {
		self.authService = authService
	}

	func loginTapped() {
		guard !email.isEmpty, !password.isEmpty else {
			errorMessage = "All fields are required"
			return
		}

		isLoading = true
		Task {
			do {
				let success = try await authService.login(UserCredentials(email: email, password: password))
				isLoggedIn = success
			} catch {
				errorMessage = "Login failed"
			}
			isLoading = false
		}
	}
}

/// View (SwiftUI)
struct LoginView: View {
	@StateObject private var vm = LoginViewModel(authService: RealAuthService())

	var body: some View {
		VStack(spacing: 12) {
			TextField("Email", text: $vm.email)
			SecureField("Password", text: $vm.password)

			if let error = vm.errorMessage {
				Text(error).foregroundColor(.red)
			}

			Button("Login") {
				vm.loginTapped()
			}
			.disabled(vm.isLoading)

			if vm.isLoggedIn {
				Text("✅ Logged in")
			}
		}
		.padding()
	}
}
```

**Pros:**
- Reactive UI - View reflects state instantly via `@Published`, Combine, or SwifUI bindings
- Decoupled logic - ViewModel has no knowledge of UI framework, which becomes testable
- Mockable/testable - You can inject fake services into the ViewModel
- Fits SwiftUI naturally - Binding & observable object work seamlessly

**Cons:**
- ViewModel can become bloated - Especially when too many UI states are tracked
- No official navigation handling - Often handled inconsistently
- Overkil for simple UIs - Too much boilerplate for tiny screens

**UIKit with MVVM:**
Still valid. Replace `@Published` with delegates, KVO, or RxSwift/Combine observers. Bind view properties in `UIViewController`

**Use Cases:**
- SwiftUI & Combine Apps - Onboarding, chat, dashboard UIs
- Complex UI logic - State-heavy flows (eg. checkout, search)
- Real-time updates - Live prices, location, IoT data