## Rendering Pipeline
1. UIView/SwiftUI View → Core Animation → Render Server → GPU
2. iOS uses a separate process (Render Server) to offload rendering work from app process
3. A frame has 16.67ms (for 60fps) to complete: layout, rendering, and compositing

### Breakdown per frame
- Main Thread: Layout, state updates, view hierarchy updates
- Core Animation (CA): Layer tree construction, implicit animations
- Render Server: Rasterizes to framebuffer (GPU)
Any slowness in any of these = dropped frames = jank

---
## Common Bottlenecks
Main Thread Overload
- Heavy computations or sync calls
- Layout recalculations in `layoutSubviews`, `viewDidLayoutSubviews`
- Complex SwiftUI body recompositions due to state changes
Overdraw
- Multiple opaque layers rendered on top of each other
- iOS has to paint unnecessary pixels
Expensive Animations
- Animating non-GPU friendly properties: frame, bounds, transform (on CPU) instead of opacity, position (on GPU)
- Complex keyframe animations without offloading to Core Animation
Non-Batched Updates
- Continuous `setNeedsLayout`, `setNeedsDisplay` calls from timers, gestures, etc

## Tools & Techniques
Instruments: Core Animation
- **Color Blended Layers**: Overdraw
- **Color Hits Green and Misses Red**: GPU compositing issues
- **FPS Monitor**: Dropped frames, frame time spikes

Instruments: Time Profiler
- Trace bottlenecks in layout, display update cycles

SwiftUI Debugging
- Use `View.body` console logs and Xcode’s View Debugger to spot excessive recomputations
- `@State`, `@Binding`, `@ObservedObject` mismanagement

## Optimization Techniques
For UIKit
- Use CALayers directly for static parts
- Prefer UIView.animate(withDuration:) which uses CA and GPU
- Avoid repeatedly creating/destroying views, reuse instead
- Set `opaque = true` where applicable

For SwiftUI
- Split complex views: so recompositions don't hit whole trees
- Use `.drawingGroup()` or force GPU rendering
- Use `EquatableView`, `.id()`, or `@Equatable` models to prevent unnecessary redraws

---
### Architecture Level Performance Patterns

| Strategy                         | Notes                                                           |
| -------------------------------- | --------------------------------------------------------------- |
| Frame skipping Animations        | Don't animate every frame, use CADisplayLink + delta timing     |
| Throttle state updates           | Avoid triggering state changes mid-frame, especially in SwiftUI |
| Async Rendering                  | Use `UIGraphicsImageRenderer` in background for heavy drawing   |
| Layer backed rendering           | Offload static content to `CATiledLayer` or `CAShapeLayer`      |
| Prefer Core Animation Transforms | They run on GPU and don't block the main thread                 |

### Android Comparison

| Aspect             | iOS                    | Android                                |
| ------------------ | ---------------------- | -------------------------------------- |
| Rendering Enginer  | Core Animation + Metal | RenderThread + Skia                    |
| Main Thread Role   | UI updates + layout    | Same                                   |
| Offloading         | Render Server + GPU    | RenderThread (separate from UI thread) |
| Overdraw Detection | Core Animation tools   | GPU overdraw debug overlay             |
| Composition        | Optimized via Metal    | Optimized via SurfaceFlinger           |

### What Architectural thinking should bring
- Know **when** to invest in animation vs static UX
- Define animation architecture: shared easing curves, animation controllers, interruptibility
- Lead design system integration: reusable components that don't regress performance
- Measure UX metrics: interaction latency, frame stability, animation jank rate

### Interview Angle
- "We achieved 60fps by deferring heavy layout to post-animation"
- "I introduced a view-pooling mechanism to prevent layout churn during scroll"
- "Used `drawingGroup()` to offload SwiftUI render to GPU"