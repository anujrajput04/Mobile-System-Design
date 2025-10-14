#### What is a Leak?
A **memory leak** occurs when allocated memory is never released because the owning object is still retained, often unintentionally. Over time, this leads to:
- Increased memory usage
- Crashes due to memory pressure, especially on lower end devices
- Sluggish UI due to GC pressure or memory churn

---
#### Detection Workflow
##### Step 1: Use the Xcode Memory Graph Debugger 
1. Run the app via Xcode with the **Debug Memory Graph** button
2. Navigate through the app flow where you suspect a leak
3. Click the button again to generate a memory graph snapshot
4. Look for:
    - ViewControllers that should’ve deallocated but are still alive
    - Strong reference chains where weak/unowned was expected
    - Objects with unexpectedly high retain counts

Common sign: Orphaned ViewControllers that are not in the VC hierarchy but still in memory

##### Step 2: Instruments > Leaks
1. Run Instruments → Choose **Leaks** template
2. Interact with the app and simulate real user flows
3. **Watch the purple diamonds**, these are confirmed leaks
4. Click on each leak → "Extended Detail" → **Backtrace or Retain Cycle**
5. Trace where the object was allocated and who’s holding it

Use this especially when you notice long session memory growth or frequent crash logs showing `EXC_BAD_ACCESS`

##### Step 3: Instruments > Allocations
1. Use when leaks aren’t detected, but memory usage seems suspicious
2. Track:
    - **Number of live instances** over time
    - **Persistent allocations**
    - `vm_regions` and large object allocations (like images, data blobs)

**Tip**: Set a **heap snapshot baseline**, run flows, then compare snapshots

---
#### Memory Leak Automation in CI
For high quality teams, leak detection is not manual only
##### Xcode Previews
- Memory Graph snapshots can be captured via unit/UI tests with test flows
- Use `XCTestCase` teardown hooks to assert that objects deallocate

```swift
func testLeak() {
	weak var weakVC: SomeViewConroller?
	autoreleasepool {
		let vc = SomeViewController()
		weakVC = vc
	}
	XCTAssertNil(weakVC)
}
```

##### Tools
- FBMemoryProfiler
- HeapInspector
- In house tooling using `os_signpost` and Instruments automation

---
#### When to Check

| Symptom                           | What to Check                                             |
| --------------------------------- | --------------------------------------------------------- |
| VC never deallocates              | Retain cycles in closures, timers, delegates              |
| Growing memory after scrolling    | Image/data caching, collection view cell reuse            |
| Crash on backgrounding            | Memory pressure callbacks missed                          |
| No leaks but RAM keeps increasing | Autorelease pools not being drained, large blob retention |
| App hangs after rapid switching   | Combine or NotificationCenter leaks                       |

#### System Level + Android/Backend Awareness

| Platform | Leak Trigger                                                          | Detection                              |
| -------- | --------------------------------------------------------------------- | -------------------------------------- |
| Android  | Context leaks (eg. activity held in singleton)                        | LeakCanary                             |
| Backend  | Retained objects in long-lived processes (WebSocket sessions, caches) | Heap dumps, memory profiler            |
| iOS      | Closures, VCs, Combine                                                | Instruments, Xcode, Unit test teardown |

#### Interview Angle
>"We use combination of Xcode Memory Graph, Instruments Leaks and Allocations, and test driven leak detection to catch regressions early. We have conventions like `[weak self]` in closure based APIs and unit tests that validate VC deallocation. For critical flows, we benchmark snapshot memory before/after and graph retained object counts. On Android, we lean on LeakCanary similarly. Our design process always accounts for object lifetime, especially in coordinators, Combine pipelines, and caching layers"

#### Leak Diagnosis
You're given this crash log:
```assembly
EXC_BAD_ACCESS (code=1, address=0x...)
```

Flow:
1. Reproduce the steps before the crash
2. Attach instruments > Allocations and Leaks
3. Navigate > capture snapshot > check retained object tree
4. Identify: is there a VC or closure not deallocating?
5. Use the retain cycle graph to fix the closure/delegate/timer