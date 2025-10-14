## In-Memory Cache
Used store frequently accessed data during app runtime.

#### iOS: `NSCache<KeyType, ObjectType>`
- Auto-purges on memory pressure
- Thread-safe, faster than Dictionary

```swift
let imageCache = NSCache<NSString, UIImage>()
imageCache.setObject(image, forKey: "profileImage")
```

#### Android: `LruCache<K, V>`
- Least recently used eviction strategy

```kotlin
val cache = LruCache<String, Bitmap>(cacheSize)
cache.put("profileImage", bitmap)
```