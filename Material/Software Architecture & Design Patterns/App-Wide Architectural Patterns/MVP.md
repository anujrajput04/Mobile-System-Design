## Model-View-Presenter Architecture
MVP separates application into:
- **Model** - data and business logic
- **View** - UI, passive, forwards events
- **Presenter** - core logic, updates the View and reacts to user input
>The View is dumb. The Presenter is smart

**Responsibilities:**

| Component | Role                                                   |
| --------- | ------------------------------------------------------ |
| Model     | Provides data via services, repositories, APIs         |
| View      | Displays data, forwards UI events to presenter         |
| Presenter | Mediator - pulls data from Model, pushes state to View |

![[MVP.excalidraw]]

**Data Flow:**
```
User → View → Presenter → Model
		↑          ↓
	(passive)   updates
```
>The View never updates iteself
>Presenter is unit-testable
>View is injected via a protocol for full decoupling

**Example - Login Flow:**
```swift
/// View Protocol
protocol LoginView: AnyObject {
	func showLoading()
	func hideLoading()
	func showError(_ message: String)
	func navigateToHome()
}

/// Presenter
final class LoginPresenter {
	weak var view: LoginView?
	let authService: AuthService

	init(view: LoginView, authService: AuthService) {
		self.view = view
		self.authService = authService
	}

	func didTapLogin(email: String, password: String) {
		view?.showLoading()
		authService.login(email: email, password: password) { [weak self] result in
			self?.view?.hideLoading()
			switch result {
				case .success:
					self?.view?.navigateToHome()
				case .failure(let error):
					self?.view?.showError(error.localizedDescription)
			}
		}
	}
}

/// ViewController
class LoginViewController: UIViewController, LoginView {
	private var presenter: LoginPresenter!

	override func viewDidLoad() {
		super.viewDidLoad()
		presenter = LoginPresenter(view: self, authService: AuthServiceImpl())
	}

	func loginButtonTapped() {
		presenter.didTapLogin(email: emailField.text ?? "", password: passwordField.text ?? "")
	}

	func showLoading() { /* UI start spinner*/ }
	func hideLoading() { /* UI stop spinner*/ }
	func showError(_ message: String) { /* Alert */ }
	func navigateToHome() { /* Push Home VC*/ }
}
```

**Pros:**
- Testable Presenter - UI logic testable without UI
- Separation of concerns - View only renders, Presenter only handles logic
- Dependency injection friendly - Swappable views and services
- No massive ViewController - Keeps UIKit/SwiftUI files slim

**Cons:**
- Boilerplate - Many protocols + wiring
- Async chaining - Presenter can become bloated if not careful
- Harder in SwiftUI - SwiftUI encourages MVVM-style unidirectional flow

**Use Cases:**

| Use Case                                        | Why MVP fits                                  |
| ----------------------------------------------- | --------------------------------------------- |
| UIKit-based apps                                | Works well with ViewController/view hierarchy |
| Screens with heavy UI logic or state management | Clear separation helps                        |
| Need test coverage for all flows                | Presenter tests are easy, fast                |

**Real Use Cases:**
- Login flows
- Multi-step forms (tax forms, onboarding)
- Screens with conditional UI blocks
- Offline flows with retry logic