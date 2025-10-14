## View-Interactor-Presenter-Entity-Router Architecture
**VIPER** is an acronym for a highly modular architecture that splits responsibilities across five components:
>View → Interactor → Presenter → Entity → Router

It evolved from Clean Architecture to suit iOS development, emphasizing testability, separation of concerns, and scalability

![[VIPER.excalidraw]]

| Component          | Responsibility                                                  |
| ------------------ | --------------------------------------------------------------- |
| View               | UI layer, receives user actions & displays presenter state      |
| Interactor         | Business logic and use cases (fetching, validating, processing) |
| Presenter          | UI logic, processes data from Interactor and updates view       |
| Entity             | Model/data structures used by the Interactor                    |
| Router (Wireframe) | Handles navigation logic between scenes                         |

**Data Flow:**
```
User → View → Presenter → Interactor → Entity
         ↑        ↓           ↑
      Render    Output  ←   Result
         ↓
      Router (on navigation)
```

All communication is protocol based, enabling full mocking and separation.

**Example - Displaying List of Articles:**
```swift
/// View (UI Layer)
protocol ArticleListView: AnyObject {
	func showArticles(_ articles: [ArticleViewModel])
	func showError(_ message: String)
}

/// Presenter (View Model)
final class ArticleListPresenter {
	weak var view: ArticleListView?
	var interactor: ArticleListInteractorInput?
	var router: ArticleListRouterInput?

	func viewDidLoad() {
		interactor?.fetchArticles()
	}
}

extension ArticleListPresenter: ArticleListInteractorOutput {
	func didFetchArticles(_ articles: [Article]) {
		let vm = articles.map { ArticleViewModel(title: $0.title) }
		view?.showArticles(vm)
	}

	func didFail(error: Error) {
		view?.showError(error.localizedDescription)
	}
}

/// Interactor (Use Case)
protocol ArticleListInteractorInput {
	func fetchArticles()
}

protocol ArticleListInteractorOutput: AnyObject {
	func didFetchArticles(_ articles: [Article])
	func didFail(error: Error)
}

final class ArticleListInteractor: ArticleListInteractorInput {
	weak var output: ArticleListInteractorOutput?

	func fetchArticles() async {
		Task {
			do {
				let articles = try await NetworkService().loadArticles()
				output?.didFetchArticles(articles)
			} catch {
				output?.didFail(error: error)
			}
		}
	}
}

/// Entity (Request & Response Models)
struct Article {
	let id: UUID
	let title: String
}

/// Router (Wireframe)
protocol ArticleListRouterInput {
	func navigateToDetail(for article: Article)
}

final class ArticleListRouter: ArticleListRouterInput {
	weak var viewController: UIViewController?

	func navigateToDetail(for article: Article) {
		let detailVC = ArticleDetailModule.build(with: article)
		viewController?navigationController?.pushViewController(detailVC, animated: true)
	}
}

/// Module Assembler
final class ArticleListModule {
	static func build() -> UIViewController {
		let vc = ArticleListViewController()
		let presenter = ArticleListPresenter()
		let interactor = ArticleListInteractor()
		let router = AritcleListRouter()

		vc.presenter = presenter

		presenter.view = vc
		presenter.interactor = interactor
		presenter.router = router

		interactor.output = presenter
		router.viewController = vc

		return vc
	}
}
```

**Pros:**
- Testable in isolation - Each piece is protocol driven, easily mockable
- Clean separation - UI, logic, data, and navigation are all decoupled
- Scales for teams and large apps - Each module is self-contained
- Improved maintenance - New devs can understand modules independently

**Cons:**
- Boilerplate heavy - Many files and protocols per screen
- Slower iteration - Simple features can feel over-engineered
- Steep learning curve - Harder for junior devs to adopt alone
- Harder with SwiftUI - Needs adaptation, SwiftUI favors MVVM

**Use Cases:**
- Multi-module large apps - VIPER modularization fits cleanly
- Apps with multiple teams working in parallel - Encapsulation avoids cross-coupling
- Legacy MVC refactor - Better boundaries & test coverage