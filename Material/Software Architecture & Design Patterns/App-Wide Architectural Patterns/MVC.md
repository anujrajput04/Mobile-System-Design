# Model-View-Controller Architecture
MVC is an architectural pattern that separates an application into three core components

| Component  | Responsibility                                                              |
| ---------- | --------------------------------------------------------------------------- |
| Model      | Manages app data and business logic                                         |
| View       | Handles UI and user output                                                  |
| Controller | Mediates between Model and View, handling user input and coordination logic |
>Separate concerns to avoid tight coupling between UI and data

![[MVC.excalidraw]]

**Native UIKit Pattern:**
In UIKit, MVC is the default architecture promoted by Apple, but often becomes "Massive View Controller" because the `UIViewController` ends up doing too much

**Breakdown of MVC:**
```swift
/// Model
struct User {
	let name: String
	let age: Int
}

/// View
class UserView: UIView {
	let nameLabel: UILabel()
	let ageLabel: UILabel()

	func configure(with user: User) {
		nameLabel.text = user.name
		ageLabel.text = "\(user.age)"
	}
}

/// Controller
class UserViewController: UIViewController {
	var user: User
	let userView = UserView()

	override func viewDidLoad() {
		super.viewDidLoad()
		view.addSubview(userView)
		userView.configure(with: user)
	}
}
```

**Responsibilities:**

| Component  | Contains                                 | Should Avoid                        |
| ---------- | ---------------------------------------- | ----------------------------------- |
| Model      | Business logic, Persistence, API parsing | UI, Animation code                  |
| View       | Layout, Colors, Animations, Presentation | Data mutation, Business logic       |
| Controller | Navigation, Event Handling, Glue logic   | Business logic, View layout details |

**Real Examples:**

| Feature                  | MVC Mapping                |
| ------------------------ | -------------------------- |
| UITableViewCell + Data   | View + Model               |
| Core Data / Realm models | Model                      |
| UIViewController         | Controller (often bloated) |
| Storyboard/XIB UI        | View (Visual layout)       |
| Delegates, IBActions     | Controller input handlers  |

Pros:
- Separation of concerns - Isolates UI from business logic
- Reusability - Models and views can be reused across controllers
- Ease of testing - Model can be unit-tested in isolation
- Straightforward for simple apps - Ideal for 1-3 screen apps

**Cons:**
- Massive View Controller - Controller ends up doing too much (UI, navigation, logic)
- Poor scalability - Tight coupling makes reuse & refactoring harder
- No strict boundaries - Easy to violate separation due to flexibility

**Mitigation:**
- Move logic to Service layer - Keep controllers thin
- Use Presenters/ViewModels - Decouple view formatting
- Use Coordinators - Extract navigation out of controller
- Prefer Composition over Inheritance - Avoid fat base controllers

**Anti-Pattern:**
```swift
/// In a bad MVC, everything is jammed in ViewController:
class HomeVC: UIViewController {
	var users: [User] = []
	var tableView: UITableView!

	override func viewDidLoad() {
		super.viewDidLoad()
		fetchData() // should be in a service
		setupTableView() // should be isolated
	}

	func fetchData() {
		URLSession.shared.dataTask(with: url) { data, _, _ in
			// parsing JSON here too
		}.resume()
	}
}
```

### Interview Questions
Design an app that shows a user profile from a backend. Walk me through your MVC separation.
Proceed with:
- Model: `User` struct from API
- View: `UserView` with labels
- Controller: Fetch user, send to view
- Service layer: Optional to separate API call

