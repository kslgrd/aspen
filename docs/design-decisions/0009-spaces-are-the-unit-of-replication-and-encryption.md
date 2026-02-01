# ADR-0009: Spaces are the unit of replication and encryption

## Status
Accepted

## Context

Aspen is a local-first, end-to-end encrypted system that supports:
- peer-to-peer synchronization
- offline operation
- explicit sharing
- space-level permissions
- overlay spaces for relationships

To reason clearly about:
- who can see what
- what data is replicated where
- how encryption keys are managed
- how peers synchronize efficiently

the system requires a single, consistent boundary for visibility, encryption, and replication.

That boundary must be simple, enforceable, and understandable to users.

---

## Decision

A **space** is the fundamental unit of:

- replication
- encryption
- visibility
- permission enforcement

All authoritative data belongs to exactly one space and is:

- encrypted with that space’s key material
- replicated only among members of that space
- inaccessible to devices without that space’s keys

There is no replication or encryption outside the context of a space.

---

## What this means in practice

### Replication

- Devices replicate data **per space**, not globally.
- A device syncs only the spaces it is a member of.
- There is no partial replication within a space.

If a device has access to a space, it receives the full authoritative history of that space.

---

### Encryption

- Each space has its own encryption keys.
- All authoritative data in a space is encrypted with those keys.
- Possession of the space key implies access to all data in the space.

There is no attempt to selectively decrypt subsets of a space.

---

### Visibility

- Visibility is binary at the space level:
  - either a user can see the space
  - or they cannot
- There are no hidden records inside a space.
- There is no “awareness” of data outside accessible spaces.

This prevents accidental metadata leakage.

---

## Relationship to permissions

This decision reinforces ADR-0003:

- Permissions are defined at the space level only.
- Roles (Viewer, Editor, Owner) apply to the entire space.
- Permission enforcement is cryptographic and logical.

Spaces define *what* you can see.
Roles define *what* you can do.

---

## Overlay spaces

Overlay spaces are not special cases.

They follow the same rules:
- their own encryption keys
- their own replication scope
- their own membership

This makes overlays composable, private, and predictable.

---

## Implications for sync

- Sync protocols operate on a per-space basis.
- Anti-entropy reconciliation is scoped to spaces.
- Peer state tracking is per space.

This simplifies:
- sync correctness
- peer discovery
- failure handling

---

## Implications for storage and indexing

- Authoritative data is grouped by space.
- Derived state (indexes, views) may aggregate across spaces **locally**, but:
  - never changes replication behavior
  - never changes encryption boundaries

Local aggregation does not imply shared visibility.

---

## What is explicitly forbidden

- Replicating individual records outside their space
- Sharing encryption keys across spaces
- “Partial” space membership
- Cross-space replication shortcuts

Any such design violates Aspen’s security and privacy model.

---

## Consequences

### Positive

- **Clear trust boundaries**
- **Simple mental model**
- **Predictable encryption**
- **Efficient replication**
- **Strong privacy guarantees**

### Negative

- **Coarse granularity**
  - Sharing requires creating or joining spaces.
- **Data duplication**
  - The same item may appear in multiple spaces via reference or snapshot.

These tradeoffs are accepted.

---

## Alternatives considered

### Record-level replication
Rejected:
- complex permission enforcement
- metadata leakage
- brittle sync semantics

### Global replication with filtering
Rejected:
- implicit trust in infrastructure
- opaque visibility rules
- incompatible with local-first operation

### Multiple overlapping replication scopes
Rejected:
- difficult to reason about
- error-prone
- undermines user trust

---

## Related documents

- ADR-0003: Permissions are defined at the space level only
- ADR-0004: Cross-space relationships are stored in overlay spaces
- ADR-0008: Derived intelligence is local-only and non-authoritative
- `docs/architecture/security-model.md`
- `docs/architecture/sync-model.md`

---

## Notes

Spaces are the backbone of Aspen.

If this invariant is weakened, nearly every other architectural decision becomes harder to reason about.