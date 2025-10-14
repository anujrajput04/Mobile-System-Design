## Linear Backoff
Wait time increases linearly

**Formula**: `dealy = base * retryCount`
Example: Base = 1s → Retry 1: 1s, Retry 2: 2s, Retry 3: 3s..

**Use Case**:
Simpler cases where rapid growth in wait time isn't needed

**Drawback**:
Not ideal under high contention or server stress