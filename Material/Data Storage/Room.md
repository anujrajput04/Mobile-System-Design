## Room
Room is Android's persistence library (from Jetpack) that provides an abstraction layer over SQLite, similar to Core Data on iOS. It simplifies database work while still letting you take advantage of the full power of SQLite

### Concepts
1. Entities
	- Annotated classes (`@Entity`) that represent database tables
	- Each instance = a row in a table
2. Data Access Objects (DAOs)
	- Interfaces or abstract classes annotated with `@Dao`
	- Contain methods (`@Insert`, `@Update`, `@Delete`, `@Query`) that define database interactions
	- Compile-time checked SQL, safer than raw SQLite
3. Database
	- Abstract class annotated with `@Database`
	- Holds the database configuration and serves as the main access point
	- Extends `RoomDatabase`

### Benefits
- Type safety: Queries are validated at compile time
- Boilerplate reduction: No need to write raw SQLite helper classes
- Integration with LiveData/Flow: Updates from DB can directly notify UI
- Migration support: Built-in APIs to handle schema changes
- Testability: DAOs are plain interfaces, easy to mock/unit test

### Example
```kotlin
@Entity(tableName = "users")
data class User(
	@Primary val id: Int,
	val name: String,
	val age: Int
)

@Dao
interface UserDao {
	@Insert
	suspend fun insert(user: User)
	
	@Query("SELECT * FROM users WHERE id = :id")
	suspend fun getUser(id: Int): User?
}

@Database(entities = [User::class], version = 1)
abstract class AppDatabase: RoomDatabase() {
	abstract fun userDao(): UserDao
}

/// Usage
val db = Room.databaseBuilder(
	context,
	AppDatabase::class.java,
	"app_database"
).build()

val userDao = db.userDao()
userDao.insert(User(1, "Anuj", 33))
```

### Compared to others
- Room vs Core Data (iOS): Same layer of abstraction on SQLite
- Room vs Realm: Room uses SQLite under the hood. Realm is its own engine. Room is more "standard" and Android native
- Room vs SQLite: Room is safer, cleaner, less boilerplate