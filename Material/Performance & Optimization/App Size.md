## App Size
Optimizing App Size is crucial for performance, download speed, user adoption (especially in low connectivity regions like Tier 2/3 cities) and runtime memory usage. It's not just about compressing assets, it's about making architectural, tooling, and build pipeline decisions to manage growth and technical debt.

### iOS App Size Optimization
#### Build Configuration & Compiler Optimizations
- User `Whole Module Optimization` in release builds
- Enable bitcode, depending on Apple's policy, deprecated for some targets
- Strip debug symbols from the release builds
- Set `Dead Code Stripping` to `YES` in release builds

#### Framework & Binary Management
- Modularize features using dynamic frameworks only if shared, otherwise prefer static libraries to  avoid App Thinning overhead
- Use XCFrameworks to share binary builds across platforms
Example:
```swift
// Prefer this
static let MyFeature = StaticLibrary()

// Instead of
import MyFeature.framework // increases binary size unnecessarily
```

#### Assets Optimization
- Use vector images (PDF) over raster images (PNG, JPEG) when possible
- Run assets through tools like
	- `imageoptim` or `tinypng`
	- `sips` for resizing in pipelines
- Xcode Asset Catalog:
	- Enable `Preserve Vector Data` only when absolutely needed
	- Use `AppIcon.appiconset` efficiently, remove unnecessary sizes

#### App Thinning
- Slicing: Devices download only assets relevant to their screen resolution
- Bitcode (deprecated): Allow Apple to re-optimize your app
- On Demand Resources: Load features/media only when needed
```swift
let tag = "Level2Assets"
let resourceRequest = NSBundleResourceRequest(tags: [tag])
resourceRequest.beginAccessingResources { error in
	// Download assets only when needed
}
```

#### Swift Specifics
- Avoid heavy generics and inline closures in frequently reused code, they increase binary size due to specialization
- Avoid embedding large static strings, assets or dictionaries in code, move to server fetch or on demand resources
- Disable Swift standard libraries embedding if targeting iOS 12+ `EMBEDDED_CONTENT_CONTAINS_SWIFT = NO`

#### Third party Libraries
- Audit dependencies regularly
	- Remove unused SDKs
	- Prefer static linking
	- Avoid overlapping libraries (eg. Firebase + Analytics + Crashlytics + GoogleSignIn, many of these duplicate dependencies)
- Tools: CocoaPods, Carthage, SPM with XCFrameworks

#### Localization
- Remove unnecessary `lproj` folders if localizations aren't used
- Avoid bloating `Localizable.strings` with unused keys

### Android Considerations
| Concern                | iOS               | Android                               |
| ---------------------- | ----------------- | ------------------------------------- |
| Vector assets          | PDF               | VectorDrawable (SVG)                  |
| App Thinning           | App Slicing + ODR | Android App Bundle + Dynamic Delivery |
| Debug Symbols          | DWARF             | Proguard / R8                         |
| Static Linking         | Static libs       | `minifyEnabled`, `shrinkResources`    |
| Localization stripping | `lproj`           | `resConfig` in Gradle                 |
| Resource compression   | PNG8, imageoptim  | `aapt2`, WebP, Lottie                 |
| Frameworks             | .framework/.a     | .aar/.jar                             |
| On-demand features     | ODR               | Dynamic Feature Modules               |
#### Tooling
| Tool                    | Purpose                                    |
| ----------------------- | ------------------------------------------ |
| `Xcode Archive` Summary | See compressed/uncompressed size breakdown |
| `App Store Connect`     | See thinned size across devices            |
| `size` / `otool`        | Inspect binary sections                    |
| `AppSize` (3rd-party)   | Visual breakdown of IPA content            |
| `Bloaty`                | Binary size profiler (advanced)            |
### Strategic Considerations
- **CI Checks**: Add size regression checks on CI eg. alert if `.ipa` grows more than X MB
- **A/B Testing with ODR**: Defer game levels or multimedia heavy content to ODR and track impact
- **Evaluate SDK Size vs Value**: eg. full Firebase SDK may add 15–25 MB, does it justify the size?
- **Make Data Informed Decisions**: Track install drop off based on size using analytics