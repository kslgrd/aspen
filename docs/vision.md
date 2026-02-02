# Vision

Aspen is a local-first platform for building applications that respect user agency, privacy, and ownership of data.

It exists to make it practical to build software where:

- data lives with the user,
- collaboration does not require surrendering trust to a central service, and
- multiple tools can coexist on top of a shared, interoperable foundation.

Aspen is not a single app. It is a shared root system.

---

## The problem Aspen is addressing

Most modern applications are built around centralized services:

- Data lives on servers by default.
- Collaboration depends on always-on infrastructure.
- Privacy is optional, retrofitted, or framed as a feature rather than a right.
- Each app is a silo, even when it overlaps heavily with others.

Local-first tools exist, but they often:

- break down under collaboration,
- rely on opaque sync services,
- or trap users in single-purpose applications with no shared substrate.

Aspen exists to explore a different shape of software.

---

## Core beliefs

### 1. Data should belong to the user

Users should be able to:

- run software fully offline,
- inspect and back up their data,
- move between devices without vendor lock-in.

This implies:

- no required backend services,
- no server-side indexing or interpretation of user data,
- clear separation between authoritative data and derived views.

---

### 2. Privacy is a right, not a premium feature

Aspen treats privacy as a baseline constraint:

- All user data is encrypted end-to-end.
- Infrastructure may exist, but it must not be able to read user data.
- Trust boundaries must be explicit and enforceable cryptographically.

This does not mean “perfect secrecy.” It means users are never asked to trust the platform operator with their content.

---

### 3. Collaboration should be safe and easy

Most permission systems expose internal complexity to users: ACLs, roles, inheritance, edge cases.

Aspen takes a different stance:

- permissions are coarse but predictable,
- sharing is explicit,
- trust is established once and maintained through identity continuity.

The system should make it clear _who_ you are sharing with and _what that implies_, without requiring users to understand cryptography or security models.

---

### 4. Local-first is a foundation, not an optimization

Aspen is designed so that:

- the first device works alone,
- sync is additive,
- collaboration emerges from replication, not central coordination.

This pushes complexity into conflict resolution, identity verification, and data modeling, but results in systems that are more resilient and more respectful of user autonomy.

---

### 5. Apps should be lenses, not silos

Rather than building one monolithic app, Aspen aims to provide:

- a shared data substrate,
- common primitives (identity, spaces, links),
- and clear boundaries between platform and app concerns.

Integration should be intentional and explicit, not accidental.

---

## The Aspen metaphor

Aspen trees often appear as many independent trunks, but are in fact a single organism connected by a shared root system.

Aspen the platform is designed the same way:

- many apps at the surface,
- one shared, interconnected foundation underneath.

Apps can evolve independently. The substrate remains coherent.