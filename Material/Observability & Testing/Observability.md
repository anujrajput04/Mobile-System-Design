## Observability
Observability is how well you can understand the internal state and behavior of a system by looking at its external outputs — logs, metrics, and traces. 

It bridges:
- **Client (mobile app)**: telemetry, analytics, crash reporting, network tracing.
- **Backend services**: logs, metrics, tracing pipelines.
- **End-to-end correlation**: seeing the full path of a user action (tap → API call → backend DB).

### Components of Observability

| Concept             | Mobile View                                                   | Backend View                                        | Purpose                                |
| ------------------- | ------------------------------------------------------------- | --------------------------------------------------- | -------------------------------------- |
| **Logging**         | Structured logs for app events, errors, lifecycle states      | Application logs per request                        | Debug and trace issues                 |
| **Metrics**         | App startup time, ANR count, FPS, memory, battery drain       | Request latency, throughput, error rate             | Health monitoring and performance KPIs |
| **Tracing**         | Span a single user action across app → network → service → DB | Distributed tracing (OpenTelemetry, Jaeger, Zipkin) | Find bottlenecks in user flows         |
| **Crash Reporting** | Firebase Crashlytics, Sentry, Instabug                        | N/A                                                 | Capture unhandled exceptions           |
| **Analytics**       | Custom events (Mixpanel, Amplitude, Firebase Analytics)       | Business metrics (conversion, engagement)           | Product health and feature usage       |

### How Observability Applies to Mobile Architecture
#### Client-Side
- Use **structured logging frameworks** (`os_log`, SwiftyBeaver, CocoaLumberjack).
- **Sampling**: avoid spamming network/logs — batch and upload logs only on WiFi or at session end.
- **Trace propagation**: include trace IDs in HTTP headers → helps link frontend traces with backend traces.
- **Health signals**: expose metrics like crash-free users %, cold start time, scroll jank (FPS drops).

#### Cross-Platform Alignment
- Android teams may use Timber for logging, LeakCanary for leak detection — ensure parity with iOS tools.
- Shared observability contracts: standardized trace IDs, error codes, API request IDs.
#### Backend Link
- Use OpenTelemetry or Datadog to tie together mobile trace IDs with backend requests.
- Example: user taps “Pay” → traceID `abc123` → appears in both app log and backend service log.

### At Staff/Architect Level - You Should Be Able To:
1. **Design an Observability Architecture**:
    - SDKs → batching layer → ingestion API → centralized data lake or monitoring tool.
2. **Define KPIs**:
    - “App startup < 2s”, “Crash-free sessions > 99.9%”, “API latency < 300ms”.
3. **Build Guardrails**:
    - Log privacy, GDPR compliance, crash deduplication.
4. **Enable developers**:
    - Provide unified logging + tracing SDKs for all apps.
5. **Collaborate with backend/SRE**:
    - To standardize trace propagation and dashboards.

### Testing in Mobile Systems
```
UI / End-to-End Tests
-----------------------
Integration Tests
-----------------------
Unit Tests (Foundation)
```

### Unit Tests
- **iOS:** XCTest / Quick+Nimble.
- Test view models, data models, business logic.
- Use **dependency injection** and **protocol mocks**.
- Avoid heavy frameworks in unit tests, run them fast and frequently.

### Integration Tests
- Test interactions between modules (eg. Networking + Persistence).
- Use **mock servers** (eg. Mountebank, WireMock) or **Localhost JSON fixtures**.
- **TestFlight builds + API mocks** to verify app-level integration before backend is ready.

### UI Tests / End-to-End Tests
- **iOS:** XCTest UI, XCUITest, KIF.
- **Android:** Espresso.
- **Cross-platform:** Appium, Detox, Maestro.
- Focus on _critical paths_ (login, purchase flow, onboarding).
- Run in **CI** on real/simulated devices (Firebase Test Lab, BrowserStack, AWS Device Farm).

### Performance Testing
- Use **Xcode Instruments** for CPU/memory profiling.
- Automate using **XCTest metrics API** or **custom telemetry**.
- Define thresholds (eg. “scroll FPS should not drop below 55fps”)

### At Staff/Architect Level - You Should:
1. Define **test strategy & coverage goals** (eg. 70% unit coverage, 90% for core modules).
2. Enable **shift-left testing**, push tests to earlier in CI/CD.
3. Build **testable architectures**:
    - MVVM, Clean Architecture, dependency injection.
4. Integrate **automated test pipelines** (CI/CD + fastlane).
5. Maintain **stability & observability of test runs** (flaky test dashboards, retries).
6. Align **test types** across platforms for parity (Android/iOS both validate core flows).

### Backend Testing Knowledge for a Mobile Architect
You should understand:
- **Contract testing** (eg. Pact), ensures backend API doesn’t break mobile clients.
- **Load testing**, helps predict API scalability.
- **Feature flag testing**, A/B tests with controlled rollout.