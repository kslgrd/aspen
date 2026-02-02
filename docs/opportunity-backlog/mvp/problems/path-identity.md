# Problem: Path identity for local files

**Problem statement:**  
When I re-share or revisit a local file, I want Aspen to recognize it as the
same item, so I can keep a coherent history, but paths change across moves,
renames, and devices.

**Evidence**

- Internal design constraints (reference-mode file items).
- Known OS-specific behavior around file identity.
- Cross-device behavior is required later for sync and collaboration.

---

## Opportunities

- How might we keep file identities stable across renames, moves, and devices without merging distinct files?

---

## Hypotheses

- We believe tracking a file identity separate from its path will achieve stable histories across renames.
- We believe a lightweight fingerprint (content hash + metadata) will achieve reliable re-binding across devices.
- We believe explicit user re-linking flows will achieve correctness when automated identity confidence is low.

**Success metrics**

- Renamed files remain linked to prior history in 90%+ of cases.
- False merges stay below 1% in a test corpus.
- Manual re-linking resolves ambiguous cases within 2 steps.

---

## Possible Concepts (MVE)

- **Hypothesis:** Separate file identity from path  
  **MVE:** Simulate rename/move events on a small corpus  
  **Effort:** low  
  **Signal:** continuity rate without manual fixes

- **Hypothesis:** Fingerprint for re-binding  
  **MVE:** Build a basic fingerprint prototype across two devices  
  **Effort:** medium  
  **Signal:** re-link success vs. false merges

- **Hypothesis:** Explicit re-linking flow  
  **MVE:** Add a “link to existing item” action in a prototype UI  
  **Effort:** low  
  **Signal:** ability to resolve ambiguity without breaking history

---

## Experiments

### [Experiment description]

- **Result:** pending
- **Evidence:** (none yet)
- **Decision:** (none yet)

---

## Notes

- Current model mentions "canonical path," which is fragile across changes.
- No chosen strategy for stable file identity.
- This is explicitly out of scope for MVP implementation, but needs exploration.
- Any chosen identity strategy will affect “re-share” semantics and may also impact UX (how users resolve ambiguity).

## Links

- [mvp-overview.md](../mvp-overview.md)
- [ADR-0007](../../../design-decisions/0007-external-sources-have-stable-identities-and-revisions.md)
