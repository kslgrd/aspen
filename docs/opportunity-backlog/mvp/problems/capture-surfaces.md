# Problem: Capture surfaces and consistency

**Problem statement:**  
When I’m capturing something in different contexts, I want the experience to feel consistent and reliable, so I can trust Aspen as my inbox, but capture surfaces behave differently and fragment the workflow.

**Evidence**

- The MVP relies on capture as the primary action.
- Planned capture surfaces already include copy/paste, and drag-and-drop files.
- Future plans include share sheets and OS-level share targets, which risk diverging behaviors if not aligned early.
- Capture needs to behave identically in terms of: item identity, revision creation, timeline events, and indexing.

---

## Opportunities

- How might we make capture consistent across surfaces without slowing users down or over-complicating the workflow?

---

## Hypotheses

- We believe a shared capture contract across all surfaces will achieve consistent outcomes without extra user effort.
- We believe a minimal required metadata set will achieve reliable indexing without adding friction.
- We believe capture metadata (source, timestamp, surface) will achieve better auditability without confusing users.

**Success metrics**

- Users report consistent capture outcomes across surfaces in 90%+ of tests.
- Capture failures drop below 2% in MVP usage.
- Manual corrections for capture metadata drop below 5% of captures.

---

## Possible Concepts (MVE)

- **Hypothesis:** Shared capture contract  
  **MVE:** Prototype copy/paste using the same contract
  **Effort:** medium
  **Signal:** outcome parity across surfaces

- **Hypothesis:** Minimal required metadata  
  **MVE:** Define required fields and test capture completeness  
  **Effort:** low
  **Signal:** missing-metadata rate

- **Hypothesis:** Capture metadata improves auditability  
  **MVE:** Add source + surface tags in timeline for a small cohort  
  **Effort:** low
  **Signal:** reduction in “how did this get here?” confusion

---

## Experiments

### [Experiment description]

- **Result:** pending
- **Evidence:** (none yet)
- **Decision:** (none yet)

---

## Notes

- Initial capture surfaces: copy/paste, drag/drop.
- In-app share sheets and browser extensions are future scope.
- Capture should be fast and default to Inbox in MVP.
- Capture is expected to expand (extensions, share sheets, system targets). We should treat capture as a contract early to avoid divergent semantics.

## Links

- [mvp-overview.md](../mvp-overview.md)
