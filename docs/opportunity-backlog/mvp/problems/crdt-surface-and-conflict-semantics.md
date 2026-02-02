# Problem: CRDT surface and conflict semantics

**Problem statement:**  
When multiple edits happen concurrently (today across future devices/peers), I want the system to converge deterministically,
so collaboration and offline use are safe, but the docs currently describe CRDTs conceptually without specifying the exact
CRDT surface or how "current revision" is selected.

**Evidence**

- Sync model states CRDT-based convergence and deterministic replay as core properties. (See [`sync-model.md`](../../../architecture/sync-model.md).)
- Data model describes records and revisions, and implies a "current revision pointer" concept. (See [`data-model.md`](../../../architecture/data-model.md).)
- Earlier review feedback highlighted this as an ambiguity that will affect implementation, testing, and UX.

---

## Opportunities

- How might we define a minimal, testable CRDT surface for revisions and links that preserves the append-only model and yields predictable user-visible behavior?

---

## Hypotheses

- We believe defining an explicit "current" selection rule (e.g., deterministic tie-break) will achieve testability
  without requiring global ordering.
- We believe limiting CRDT semantics to a small set of data types (revision sets + pointers) will achieve clarity
  without over-engineering.

**Success metrics**

- Given the same set of operations, all replicas derive the same effective state.
- Conflicts are representable and debuggable (history preserved; selection rule explainable).
- The model supports future collaboration without rewriting the authoritative model.

---

## Possible Concepts (MVE)

- **Hypothesis:** Explicit "current" selection rule
  **MVE:** Write a small spec + property tests that prove convergence under reordering/duplication
  **Effort:** medium
  **Signal:** deterministic outcomes across randomized operation orders

- **Hypothesis:** Minimal CRDT surface
  **MVE:** Implement only revision-set convergence + pointer selection for one record type
  **Effort:** medium
  **Signal:** end-to-end sync simulation passes invariants

---

## Experiments

### [Experiment description]

- **Result:** pending
- **Evidence:** (none yet)
- **Decision:** (none yet)

---

## Notes

- MVP is single-device, but choosing a clear model early reduces future rework.
- This doc is about semantics and testability, not transport protocol.

## Links

- [`mvp-overview.md`](../mvp-overview.md)
- [`sync-model.md`](../../../architecture/sync-model.md)
- [`data-model.md`](../../../architecture/data-model.md)
