# ADR-0003: Permissions are defined at the space level only

## Status
Accepted

## Context

Aspen is a local-first, end-to-end encrypted platform that supports:
- peer-to-peer sync
- CRDT-based collaboration
- multiple apps sharing a single data substrate
- frictionless sharing with strong privacy guarantees

Fine-grained permission systems (per-record, per-field ACLs) introduce:
- complex UX
- difficult cryptographic enforcement
- metadata leakage risks
- fragile sync semantics under concurrency

Aspen prioritizes clarity, predictability, and enforceability over maximal flexibility.

---

## Decision

All permissions in Aspen are defined **at the space level only**.

Each space defines:
- who can read data
- who can write data
- who can administer the space

Roles:
- **Viewer**: read access
- **Editor**: read and write access
- **Owner**: administrative access (membership, sharing, key rotation)

There are **no per-record, per-field, or per-link permissions**.

---

## Consequences

### Positive

- **Simple mental model**
  - Users reason about access in terms of spaces, not objects.
- **Cryptographic enforceability**
  - Space keys cleanly enforce access.
- **Predictable sync**
  - All peers in a space see the same authoritative data.
- **Lower metadata leakage**
  - No permission edges to infer from partial visibility.
- **Cross-app consistency**
  - All apps operate under the same permission model.

### Negative

- **Reduced granularity**
  - Some sharing patterns require restructuring data into spaces.
- **Up-front modeling**
  - Apps must design around space boundaries intentionally.

These tradeoffs are accepted.

---

## Privacy and security implications

- Possession of a space key implies access to all data in that space.
- There is no attempt to partially decrypt space contents.
- Permission checks are enforced:
  - cryptographically (via keys)
  - logically (via role checks on operations)

If a user can read a space, they can read everything in it.

---

## Relationship to overlay spaces

The absence of fine-grained permissions necessitates **overlay spaces**:

- Cross-space relationships cannot be stored in content spaces without leaking existence.
- Overlays provide a separate permission boundary for relationships.
- Overlay spaces follow the same permission model as content spaces.

This preserves privacy without introducing ACL complexity.

---

## Implications for application design

Applications must:

- treat spaces as the primary unit of visibility
- avoid relying on per-record access control
- model sensitive relationships via overlays or separate spaces
- surface space boundaries clearly in the UI

Apps that require per-item permissions must either:
- create additional spaces, or
- be considered out of scope for Aspen.

---

## Implications for sync and CRDTs

- All operations in a space are visible to all members.
- No conditional replication is required.
- CRDT convergence is simplified by uniform visibility.

Permission enforcement occurs before accepting operations.

---

## Alternatives considered

### Per-record permissions
Rejected:
- complex UX
- difficult to enforce cryptographically
- introduces metadata leaks
- complicates sync and CRDT semantics

### Attribute-based access control (ABAC)
Rejected:
- opaque to users
- brittle under evolution
- incompatible with local-first replication

### Hybrid models
Rejected:
- increases conceptual overhead
- undermines clarity of authority boundaries

---

## Related documents

- ADR-0001: Authoritative data is append-only operations and blobs
- ADR-0002: SQLite is derived local state only
- `docs/architecture/data-model.md`
- `docs/architecture/security-model.md`

---

## Notes

Space-level permissions are a deliberate constraint.

They trade expressive power for:
- safety
- simplicity
- and enforceability

Any feature that assumes per-record permissions is incompatible with Aspen.