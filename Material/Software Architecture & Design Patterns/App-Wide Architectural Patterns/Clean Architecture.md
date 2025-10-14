**Clean Architecture**, coined by Robert C Martin (Uncle Bob), is a layered architecture that emphasizes independence of:
- Frameworks (UIKit, SwiftUI, Firebase)
- UI
- Database
- Device
>The most important part of your app is not the UI, but the business rules. Protect them

![Clean Architecture by Robert C Martin](https://youtu.be/Nltqi7ODZTM)
![[Clean.excalidraw]]

**Diagram:**
```
--------------
|   UI Layer |               ← SwiftUI / UIKit / CLI
--------------
	   ↑
-----------------------
|  Presentation Layer |      ← ViewModels / Presenters / Controllers
-----------------------
       ↑
-------------------       
|  Use Case Layer |          ← Application Business Logic
-------------------
       ↑
------------------------       
|  Domain (Entitites)  |     ← Core Business Rules (Framework agnostic)
------------------------
```

**Principles:**
- Dependency Rule - Code dependencies must always point inward, toward core logic
- Independence - UI, DB, and frameworks can change without affecting business rules
- Interface Segregation - Outer layers define protocols, inner layers implement them
- Testability - Inner logic is fully unit-testable without UI or DB involved

**Example - News App (SwiftUI + Clean Architecture):**
```swift
/// Entity (Domain Layer)
struct Article {
	let id: UUID
	let title: String
	let summary: String
}

/// Use Case
protocol ArticleRepository {
	func fetchArticles() async throws -> [Article]
}

final calss FetchArticlesUseCase {
	let repo: ArticleRepository

	init(repo: AritcleRepository) {
		self.repo = repo
	}

	func execute() async throws -> [Article] {
		try await repo.fetchArticles()
	}
}

/// ViewModel (Presentation Layer)
@MainActor
final class ArticleListViewModel: ObservableObject {
	@Published var articles: [Article] = []
	@Published var errorMessage: String?

	private let fetchArticles: FetchArticlesUseCase

	init(fetchArticles: FetchArticlesUseCase) {
		self.fetchArticles = fetchArticles
	}

	func load() async {
		do {
			articles = try await fetchArticles.execute()
		} catch {
			errorMessage = "Failed to load articles"
		}
	}
}

/// SwiftUI View (UI Layer)
struct ArticleListView: View {
	@StateObject var vm: ArticleListViewModel

	var body: some View {
		List(vm.articles, id: \.id) { article in
			VStack(alignment: .leading) {
				Text(article.title).font(.headline)
				Text(article.summary).font(.subheadline)
			}
		}
		.task {
			await vm.load()
		}
	}
}

/// Data Layer (Infrastructure)
final class RemoteArticleRepository: ArticleRepository {
	func fetchArticles() async throws -> [Article] {
		// Simulated network logic
		return [Article(id: UUID(), title: "Clean Architecture", summary: "Layered design in Swift.")]
	}
}
```

**Pros:**
- Highly testable - Core logic tested without DB, UI, or network
- Swappable UIs - Easily switch from SwiftUI to UIKit to CLI
- Framework-agnostic - Core use cases don't rely on Combine, SwiftUI, etc
- Scales well - Ideal for large apps with many domains & workflos

**Cons:**
- More files, more boilerplate - Avoid in very simple apps
- Learning curve for junior devs - Must train people in architecture boundaries
- Async boundaries can get verbose - Especially when passing closures/callbacks

**Use Cases:**
- Multi-feature apps (eg. ride share, fintech) - Clean separation, scalibility
- Domain-rich apps (eg. healthcare, banking) - Isolated business rules
- Want testable logic separate from frameworks - Perfect match
- You expect team growth or handovers - Enforces discipline & boundaries