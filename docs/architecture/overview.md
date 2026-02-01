# Architecture Overview

This document describes the high-level architecture of Aspen: its major components, boundaries, and invariants.

It explains *how the system is shaped*, not how individual features are implemented.

---

## Architectural goals

Aspen’s architecture is shaped by a small set of non-negotiable goals:

- Local-first operation
- End-to-end encrypted data
- No required backend services
- Peer-to-peer collaboration
- Support for multiple apps sharing one data substrate
- Clear, enforceable privacy boundaries

These goals constrain the design more than convenience or performance concerns.

---

## High-level shape

At a high level, Aspen consists of:

- a **platform kernel** that owns data, identity, and sync
- a set of **apps** that provide UI and domain behavior
- a clear split between **authoritative data** and **derived state**

```
┌────────────┐
│   Apps     │  ← UI + domain logic
└─────┬──────┘
      | Platform API
┌─────▼──────┐
│  Platform  │  ← identity, crypto, storage, sync
└─────┬──────┘
      │
┌─────▼──────┐
│ Data Store │  ← encrypted ops + blobs
└────────────┘
```

Apps never bypass the platform. The platform never interprets app semantics.

---

## Platform vs apps

### Platform responsibilities

The Aspen platform is responsible for:

- User identity and device identity
- Key management and encryption
- Space membership and permissions
- CRDT-based replication
- Persistence of authoritative data
- Change notification (subscriptions)
- Building and maintaining derived local indexes

The platform exposes **capabilities**, not raw data access.

### App responsibilities

Apps are responsible for:

- UI and user interaction
- Defining record schemas they own
- Rendering and editing records
- Initiating platform actions (create record, link, share space)

Apps **do not**:

- manage encryption keys
- implement sync protocols
- enforce permissions
- read or write raw storage directly

---

## Authoritative data model

Authoritative data is the only data that is replicated between devices.

It consists of:

### 1. Operation logs

- Append-only
- Encrypted
- CRDT-compatible
- Scoped to a space
- Signed by device keys

Operations describe _intent_ (e.g. “add revision”, “create link revision”), not materialized state.

### 2. Blobs

- Encrypted binary content
- Content-addressed
- Referenced by operations
- Stored separately from ops

Blobs may represent:

- file snapshots
- screenshots
- web captures
- large text payloads

---

## Derived state

Derived state exists purely to make the system usable and fast.

Examples:

- SQLite materialized views
- Full-text search indexes
- Vector similarity indexes
- Graph projections and layouts

Properties of derived state:

- Local-only
- Rebuildable
- Not encrypted for transport
- Not replicated

If a derived index is deleted, the system must be able to recover by replaying authoritative data.

---

## Spaces as the primary boundary

A **space** is the fundamental unit of:

- visibility
- permissions
- encryption
- replication

All authoritative data belongs to exactly one space.

Permissions are defined only at the space level:

- Viewer
- Editor
- Owner

There are no per-record or per-field permissions. This constraint simplifies sync, encryption, and reasoning about leaks.

---

## Overlays (relationship layers)

Cross-space relationships introduce a privacy risk if stored directly.

Aspen addresses this with **overlay spaces**.

- An overlay is a space whose contents describe relationships between items in other spaces.
- Overlays use the same permission and encryption model as content spaces.
- Users without access to an overlay cannot infer the existence of those relationships.

Examples:

- Cross-space links
- Private relationship annotations
- Shared “connection layers” between otherwise independent spaces

---

## Identity and trust

- Identity is **per user**, not per space.
- First contact uses Trust On First Use (TOFU).
- Identity continuity is enforced:
  - identity key changes are treated as security events
- Devices have their own keys, signed by the user identity.

This allows:

- low-friction sharing
- strong guarantees after initial trust
- device rotation without constant re-verification

Detailed mechanics are described in the security model document.

---

## Sync model overview

- CRDT-based
- Peer-to-peer first
- No required servers
- Relays allowed only as encrypted transport

Key properties:

- Append-only logs enable anti-entropy reconciliation
- Real-time streams are opportunistic, not authoritative
- Devices may join and leave arbitrarily

Sync never operates on SQLite or other derived state.

---

## Multi-app support

Multiple apps can coexist on top of the same platform by:

- Namespacing record types by app
- Sharing core primitives (spaces, links, refs)
- Using overlays for cross-app relationships

This enables:

- intentional integration
- shared context
- avoidance of duplicated storage and indexing

Apps remain loosely coupled through the platform.

---

## Architectural invariants

The following must always hold:

- Authoritative data is append-only
- SQLite is a cache, not a source of truth
- All replicated data is encrypted
- Permissions are enforced cryptographically
- Cross-space links never leak existence

Violating these invariants indicates a design error.
