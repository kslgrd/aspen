# ADR-0010: Applications are lenses over shared data, not owners of it

## Status
Accepted

## Context

Aspen is designed as a platform that supports:
- multiple applications sharing a single data substrate
- long-lived user data that outlasts individual tools
- local-first operation with peer-to-peer sync
- explicit, enforceable privacy boundaries

Traditional applications tend to:
- own their data
- define exclusive schemas
- trap information inside app-specific silos

This model conflicts with Aspen’s goals of composability, longevity, and user ownership.

---

## Decision

In Aspen, **applications are lenses over shared data**, not owners of it.

- Applications provide views, interactions, and domain logic.
- The platform owns:
  - storage
  - identity
  - encryption
  - sync
  - permissions
- Data is owned by the user and governed by spaces, not by apps.

Applications do not have exclusive control over the data they create.

---

## What “lens” means

An application lens:
- defines record types it understands
- renders and edits those records
- creates and consumes links
- proposes actions and transformations

An application lens does **not**:
- bypass platform rules
- assume exclusive access to records
- control replication or encryption
- enforce its own permission model

Multiple applications may present different views over the same underlying data.

---

## Record ownership and responsibility

- Record types are namespaced by application.
- The application that defines a record type is responsible for:
  - its schema
  - its semantics
  - its UI behavior

The platform is responsible for:
- persistence
- identity and authorship
- access control
- sync and convergence

No application “owns” a space.

---

## Cross-application interaction

Because applications share a substrate:

- One application may link to records defined by another.
- Derived views may aggregate data across applications locally.
- Applications may coexist without explicit coordination.

This enables:
- intentional integration
- reuse of context
- avoidance of duplicated storage and indexing

Applications remain loosely coupled.

---

## Implications for extensibility

This decision enables:

- adding new applications without migrating existing data
- retiring applications without losing information
- experimenting with alternative interfaces over the same data
- long-term survivability of user knowledge

Data is more durable than any one application.

---

## Implications for user experience

Users should experience Aspen as:
- one coherent system
- multiple ways of working with the same information

Switching applications should not:
- fragment data
- require duplication
- require re-sharing or re-permissioning

Applications are interchangeable views, not silos.

---

## Enforcement

The platform enforces this decision by:

- mediating all data access through platform APIs
- preventing direct storage access by applications
- enforcing space-level permissions consistently
- isolating derived state per application

Applications that require exclusive data control are out of scope.

---

## Consequences

### Positive

- **Strong data ownership**
- **Longevity of information**
- **Low coupling between apps**
- **Composable integrations**
- **Reduced lock-in**

### Negative

- **Higher design discipline**
  - Applications must tolerate shared ownership.
- **Limited assumptions**
  - Apps cannot rely on exclusive schemas or invariants.

These tradeoffs are accepted.

---

## Alternatives considered

### Application-owned databases
Rejected:
- creates silos
- complicates sync
- undermines user ownership

### App-specific permission models
Rejected:
- inconsistent UX
- increased complexity
- weakens platform guarantees

### Strong app isolation
Rejected:
- prevents integration
- duplicates storage and indexing
- contradicts Aspen’s goals

---

## Related documents

- ADR-0001: Authoritative data is append-only operations and blobs
- ADR-0003: Permissions are defined at the space level only
- ADR-0008: Derived intelligence is local-only and non-authoritative
- ADR-0009: Spaces are the unit of replication and encryption
- `docs/architecture/overview.md`

---

## Notes

Applications may come and go.

The data, and the relationships between it, must endure.