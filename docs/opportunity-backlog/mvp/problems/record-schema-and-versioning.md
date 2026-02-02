# Problem: Record schema and versioning for multi-app durability

**Problem statement:**  
When multiple apps evolve over time on top of the same Aspen substrate, I want old data to remain usable,
so the system stays durable across versions, but record schemas and semantics will change and can break
older clients or future lenses if we don’t define a versioning strategy early.

**Evidence**

- Aspen’s architecture assumes multiple apps are lenses over shared data. (See [`overview.md`](../../../architecture/overview.md) and [`0010-applications-are-lenses-not-owners.md`](../../../design-decisions/0010-applications-are-lenses-not-owners.md).)
- The data model defines records and revisions but avoids storage encodings and schema evolution mechanics. (See [`data-model.md`](../../../architecture/data-model.md).)
- Earlier feedback identified schema/versioning as a missing piece even for MVP, because it prevents re-platforming later.

---

## Opportunities

- How might we evolve record types safely across app and platform versions, without central coordination and without breaking old data?

---

## Hypotheses

- We believe every record type including an explicit schema/version marker will achieve forward-compatibility by allowing readers to interpret or ignore unknown shapes.
- We believe treating revisions as self-describing (type + version + optional fields) will achieve safe evolution without requiring synchronized deployments.
- We believe a minimal migration strategy ("read old, write new") will achieve practical evolution for MVP without building a full migration framework.

**Success metrics**

- Older clients can ignore unknown fields without corrupting derived views.
- Newer clients can read historical records without manual intervention.
- Schema changes do not require rewriting authoritative history.

---

## Possible Concepts (MVE)

- **Hypothesis:** Explicit schema/version on records
  **MVE:** Define a minimal record envelope and implement one backwards-compatible schema bump
  **Effort:** low
  **Signal:** old data remains readable after a change

- **Hypothesis:** "Read old, write new" migrations
  **MVE:** Implement a derived-index rebuild that can handle both versions
  **Effort:** medium
  **Signal:** SQLite rebuild succeeds across mixed history

---

## Experiments

### [Experiment description]

- **Result:** pending
- **Evidence:** (none yet)
- **Decision:** (none yet)

---

## Notes

- This is the seam between "apps as lenses" and "append-only authoritative history".
- Even if MVP ships as a single lens, versioning should be cheap to include from day one.

## Links

- [`mvp-overview.md`](../mvp-overview.md)
- [`data-model.md`](../../../architecture/data-model.md)
- [`overview.md`](../../../architecture/overview.md)
- [`0010-applications-are-lenses-not-owners.md`](../../../design-decisions/0010-applications-are-lenses-not-owners.md)
