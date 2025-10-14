## Feature Flags & Remote Config
A **Feature Flag** (or Feature Toggle) is a conditional switch in code used to **enable or disable features are runtime**, without deploying new builds. It supports **A/B testing**, **gradual rollout**, **canary releases**, and **quick rollback of faulty features**.

**How It Works:**
```swift
if FeatureFlags.shared.isEnabled(.newOnboarding) {
	showNewOnboarding()
} else {
	showOldOnboarding()
}
```

The flag's value may come from:
- Local config
- Remote server (dynamic flags)
- User defaults
- Experiment platform (eg. LaunchDarkly, Firebase Remote Config)

| Component      | Description                        |
| -------------- | ---------------------------------- |
| Flag Enum      | Defines all available feature keys |
| Flag Manager   | Singleton to resolve flag values   |
| Backend Config | Stores overrides per user or group |

**Example - Local Flag Setup:**
```swift
/// Define Flags
enum Feature: String {
	case newOnboarding
	case searchRedesign
}

/// Flag Manager
final class FeatureFlags {
	static let shared = FeatureFlags()

	private var flags: [Feature: Bool] = [
		.newOnboarding: false,
		.searchRedesign: true
	]

	func isEnabled(_ feature: Feature) -> Bool {
		flags[feature] ?? false
	}

	func set(_ feature: Feature, enabled: Bool) {
		flags[feature] = enabled
	}
}

/// Usage
if FeatureFlags.shared.isEnabled(.newOnboarding) {
	onboardingService.startNewFlow()
} else {
	onboardingService.startLegacyFlow()
}

/// Remote Feature Flags (Firebase Remote Config)
RemoteConfig.remoteConfig().fetchAndActivate {_, _ in
	let enabled = RemoteConfig.remoteConfig().configValue(forKey: "searchRedesign").boolValue
	FeatureFlags.shared.set(.searchRedesign, enabled: enabled)
}
```

**Pros:**
- Safe Deployment - Release broken/incomplete code behind a switch
- A/B Testing - Compare behavior across user groups
- Gradual rollout - Enable for 1%, then 10%, then 100%
- Fast rollback - Turn off feature instantly without hotfix
- Developer toggles - Enable beta/dev UI without App Store release

**Cons:**
- Flag bloat - Should audit and delete stale flags from time to time
- Complex logic - Don't nest flags too deeply
- Risk of inconsistency - Ensure consistency between client/server config
- Testing complexity - Test both flag states in CI

**Best Practices:**
- Use **typed enums**, not raw strings
- Default to **safe (off)** for new flags
- Always **expire flags** after release
- Keep UI/UX consistent across states
- Don't **gate security or critical paths** using flags alone

**Tooling:**

| Tool                   | Usage                                 |
| ---------------------- | ------------------------------------- |
| Firebase Remote Config | Remote dynamic flag control           |
| LaunchDarkly           | Scalable feature flag management      |
| Optimizely             | A/B testing + flagging                |
| UserDefaults           | Manual dev flags                      |
| Backend config APIs    | Team-built toggles with API responses |
