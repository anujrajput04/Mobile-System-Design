## Circuit Breakers
A state machine that stops calling an unhealthy service for a while to give it time to recover

**States**:
- 🔴 **Open**: No requests are sent. Only probe/health checks
- 🟡 **Half-Open**: Some requests are allowed through
- 🟢 **Closed**: All request go though as normal

**Use Case**:
Prevents cascading failures in microservices

**Libraries/Tools**:
- iOS/Android: Needs to be implemented manually
- Backend: Hysterix (Netflix), Resilience4j (Java), Polly (.NET)