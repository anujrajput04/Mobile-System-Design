## ObjectBox
ObjectBox is a high performance, object oriented database designed as an alternative to SQLite/Room/Realm. Unlike Room, which sits on top of SQLite, ObjectBox uses its own NoSQL engine built for speed and efficiency on mobile and IoT devices

### Features
- **Object-Oriented**
    - Works directly with objects instead of rows/columns.
    - No need to write SQL queries.
- **Performance**
    - Extremely fast reads/writes (benchmarks show ~10× faster than SQLite/Room in many cases).
    - Lightweight memory footprint.        
- **Relations**
    - Supports one-to-one, one-to-many, many-to-many as native object links.
- **Schema Management**
    - Schema migrations are mostly automatic; no manual migration code.        
- **Multiplatform**
    - Works on Android, iOS, Linux, macOS, Windows, even embedded/IoT. 
- **Reactive Data**
    - Supports reactive queries (like Room’s Flow/LiveData).