## Exponential Backoff
Wait times between retries increase exponentially

**Formula**: `delay = base * (2 ^ retryCount)`
Example: If base is 500ms: Retry 1 → 500ms, Retry 2 → 1000ms, Retry 3 → 2000ms

**Use Case**:
Prevents overwhelming the server, especially in large scale distributed systems

**Variants**:
- **Full Jitter**: `random(0, base * (2 ^ retryCount))`, adds randomness to avoid thundering herd problem
- **Decorrelated Jitter**: Ensures variability while bounding max delay