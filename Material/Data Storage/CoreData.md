## CoreData
CoreData is Apple's object graph and persistence framework. It is not just a database wrapper, it manages:
- An in-memory object graph
- Persistence to disk (via SQLite, binary, XML, or in-memory stores)
- Change tracking
- Undo/redo support
- Faulting and lazy loading
- Data model versioning and migrations

It's ORM + object graph manager + persistence layer with SQLite often as the default backing store

### Implementation
A Core Data stack typically involves:
1. **NSManagedObjectModel**: schema definition
2. **NSPersistentStoreCoordinator**: coordinates between model and store
3. **NSManagedObjectContext**: scratchpad for objects, supports undo, change tracking
4. **Persistent Store**: where data lives
```swift
let container = NSPersistentContainer(name: "MyModel")
container.loadPersistentStores { _, error in
	if let error { fatalError("Error: \(error)") }
}

let context = container.viewContext
let entity = MyEntity(context: context)
entity.name = "Sample"
try? context.save()
```

### When CoreData is ideal
- Complex object graphs (eg. social graph, hierarchical data)
- Rich model relationships (one-to-one, one-to-many, many-to-many)
- Undo/redo or complex change tracking
- Lazy loading for large datasets
- Integration with iCloud (via NSPersistentCloudKitContainer for sync)
- Apple ecosystem only apps where ORM convenience > portability

### System Design Interview Relevance
- How Core Data works under the hood (object graph, contexts, persistent store)
- Threading model (contexts are thread confined, use `perform`/`performAndWait`)
- Batch operations, batch insert/fetch/update for performance
- Faulting & caching, how Core Data optimizes memory
- Migrations, lightweight vs mapping model
- Sync, CloudKit integration patterns & limitations
- Trade offs when considering Realm/SQLite instead

### Threading Model
- `NSManagedObjectContext` is NOT thread safe
- Use one context per queue
- Common pattern:
	- Main context for UI
	- Background context for heavy writes/fetches
	- Merge changes into main context
```swift
container.performBackgroundTask { bgContext in
	// Heavy work here
	try? bgContext.save()
}
```

### Migrations
Lightweight Migration:
- Add optional attributes
- Add relationships without breaking existing data
- Enabled by:
```swift
options: [NSMigratePersistentStoresAutomaticallyOption: true, NSInferMappingModelAutomaticallyOption: true]
```

Manual Mapping Model Migration:
- When renaming attributes, changing data types, or major schema changes

### Interview Prompt
"Design an offline-first task management app with complex relationships and iCloud sync"

Expected Core Data points:
- Use NSPersistentCloudKitContainer to sync between devices
- Entities: Task, Project, Tag (many-to-many relationships)
- Background context for network sync to avoid blocking UI
- Lightweight migration for adding new attributes
- Batch fetch for large datasets
- Faulting to limit memory usage