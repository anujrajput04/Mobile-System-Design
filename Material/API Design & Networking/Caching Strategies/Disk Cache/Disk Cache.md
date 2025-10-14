## Disk Cache
Persists data across app launches. Used for:
- API responses
- Images
- JSON blobs

**Libraries**:
- iOS: Custom (eg. Using `FileManager` or `URLCache`)
- Android: `OkHttp` includes disk caching automatically via `Cache`