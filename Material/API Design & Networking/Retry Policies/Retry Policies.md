## Retry Policies
A retry policy defines how a system handles transient failures like timeouts, rate limits, temporary unavailability when calling external systems, APIs, or services. A good retry strategy ensures resilience without overwhelming the system

**Components of Retry Policy**:
- Max Attempts: Total number of retry attempts, including the first one
- Delay Strategy: Time gap between retries (fixed, exponential, jitter, etc)
- Timeout per Attempt: How long to wait before considering a single attempt as failed
- Retryable Errors: Define which error codes/exceptions are safe to retry (eg. 5xx, timeouts)
- Circuit Breaker: Block retries temporarily when failures spike, to protect systems

#### Common Retry Strategies
![[Fixed Delay]]

![[Exponential Backoff]]

![[Linear Backoff]]

![[Circuit Breakers]]

![[Retry after OAuth Token Refresh]]

#### Combined Retry Strategy (Example)
1. API call fails with 500 (server error)
2. Retry with exponential backoff and jitter
3. On 401, trigger token refresh, retry after new token
4. After 3 failed retries or if latency > threshold, open circuit
5. After cooldown, half-open state allows 1 probe request

#### When NOT to Retry
- 4xx errors: Authentication errors, bad requests, validation failures
- Idempotent violations: If the operation causes side effects (eg. money transfer)
- Rate limits: Unless accompanied by a `Retry-After` header

**Sample Policy for HTTP Client**:
```json
{
	"max_attempts": 5,
	"base_delay_ms": 500,
	"strategy": "exponential_backoff_with_jitter",
	"retry_on": ["timeout", "5xx", "connection_reset"],
	"max_total_retry_time_ms": 10000
}
```

#### Best Practices
- Only retry idempotent operations (GET, PUT, safe POST)
- Add retry budget caps to prevent client explosion
- Respect server headers like `Retry-After`
- Monitor and log each retry attempt with reason
- Combine with circuit breakers to fail fast when upstream is overwhelmed