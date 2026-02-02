# Problem: Platform API and FFI boundary (Rust ↔ macOS app)

**Problem statement:**  
When we build the MVP as a macOS app on top of a portable platform library, I want the boundary to be stable and ergonomic,
so we can ship quickly without painting ourselves into a corner, but Rust ↔ Swift/Objective‑C integration choices can easily
leak complexity into the app, constrain threading, or force rewrites later.

**Evidence**

- ADR-0011 recommends a portable platform library (likely Rust) embedded into platform apps. (See [`0011-technical-foundations.md`](../../../design-decisions/0011-technical-foundations.md).)
- The architecture requires apps to never bypass the platform for authoritative data, keys, or storage. (See [`overview.md`](../../../architecture/overview.md).)
- The MVP needs subscriptions (timeline/search updates), capture APIs, and “locked/unlocked” states; these imply async boundaries and error surfaces.

---

## Opportunities

- How might we define a minimal platform API that supports MVP needs while keeping the boundary future-proof for additional apps and multi-device sync?

---

## Hypotheses

- We believe defining a small, explicit API surface (capture, query, subscribe, rebuild, lock/unlock) will achieve fast iteration
  without coupling the app to internal storage/crypto details.
- We believe choosing one concurrency model at the boundary (e.g., async callbacks / streams with backpressure) will achieve predictable UI behavior
  without deadlocks or re-entrancy bugs.

**Success metrics**

- macOS app contains no direct access to keys or storage; all flows use the platform API.
- Platform calls are safe to invoke from UI thread(s) without blocking (or provide explicit non-blocking APIs).
- Subscription/event delivery is deterministic and testable (no “sometimes the timeline updates” bugs).

---

## Possible Concepts (MVE)

- **Hypothesis:** Minimal API surface first  
  **MVE:** Define the MVP API in a doc + implement a stubbed in-memory platform that the macOS app can call end-to-end  
  **Effort:** medium  
  **Signal:** UI can be built without knowing storage internals

- **Hypothesis:** Concurrency clarity at the boundary  
  **MVE:** Pick one event/subscription mechanism and prototype timeline updates under load (rapid captures)  
  **Effort:** medium  
  **Signal:** no UI hangs; updates are ordered and consistent

---

## Experiments

### [Experiment description]

- **Result:** pending
- **Evidence:** (none yet)
- **Decision:** (none yet)

---

## Notes

- This problem should be resolved before substantial macOS UI work, to avoid rewrites.
- The boundary should account for:
  - data types and versioning
  - error semantics (recoverable vs fatal)
  - logging/diagnostics surfaces

## Links

- [`platform.md`](../spec/platform.md)
- [`0011-technical-foundations.md`](../../../design-decisions/0011-technical-foundations.md)
- [`overview.md`](../../../architecture/overview.md)
