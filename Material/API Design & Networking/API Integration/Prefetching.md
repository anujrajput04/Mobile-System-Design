## Prefetching
**Prefetching** is a technique used in mobile apps to improve responsiveness and perceived performance by **proactively fetching data before it is needed**. Instead of waiting for the user to request data (eg. by navigating to a screen to a certain point), the app anticipates what the user might need next and downloads it in advance.

**Use Cases**:
- List Scrolling: Fetch next page of feed when user reaches 70% of current list
- Navigating Between Screens: Prefetch profile data when user taps on a username
- Media Content: Preload thumbnails, audio, or video segments before they are requested
- Search Autocomplete: Fetch predictions as the user types

**Things to watch out for**:
- **Overfetching**: Wastes bandwidth or memory
- **Stale Data**: Prefetched data may be outdated when used
- **Battery & Data Usage**: Impacts user experience if not throttled or conditional
- **Security**: Avoid prefetching sensitive/private data unnecessarily

**Best Practices**:
- Use heuristics based on **user behavior** and **analytics** to guide prefetch decisions
- Combine with **caching** to avoid network hits if already available
- Throttle aggressively on metered networks or low battery
- Cancel or deprioritize unnecessary fetches if UI intent changes

---
### iOS Implementation Strategies
1. **Using URLSession with Operation Queues**
	- Combine prefetching with background `URLSession` tasks or priority queues.
2. **UICollectionView/UITableView**: `prefetchDataSource`
```swift
func collectionView(_ collectionView: UICollectionView, prefetchItemsAt indexPaths:[IndexPath]) {
	// Predictive fetch when nearing certain index
}
```
3. **Combine or async/await + Task**
	- Predict and launch async fetch calls based on UI state, like preloading detail screen data on hover or selection

---
### Android Implementation Strategies

1. **RecyclerView: `RecyclerView.OnScrollListener`**
	- Trigger network prefetch when nearing bottom:

```kotlin
if (visibleItemCount + pastVisibleItems >= totalItemCount - THRESHOLD) {
	// Trigger prefetch
}
```
2. **Coroutines + ViewModel**
	- Use coroutine scopes to launch speculative API calls on idle time or future navigation intents.
3. **WorkManager / Prefetch APIs (Jetpack)**
	- For background prefetching when app is idle, connected, and charging.

---
### Prefetching vs Caching vs Background Sync

| **Feature** | **Prefetching**                        | **Caching**                 | **Background Sync**                     |
| ------- | ---------------------------------- | ----------------------- | ----------------------------------- |
| When?   | _Before_ data is explicitly needed | _After_ data is fetched | _Periodically or reactively_        |
| Goal    | Reduce perceived latency           | Avoid repeat calls      | Keep local data fresh in background |
| Trigger | Predictive/Heuristic/User action   | On-demand API calls     | Timer or push notifications         |
