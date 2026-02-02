# ADR-0011: Technical Foundations and Build Strategy

## Status
Accepted

## Context

Aspen is a local-first, privacy-preserving application designed to operate across multiple user devices with the following requirements:

- Full offline functionality
- Concurrent editing across devices, including while offline
- No locks, leases, or exclusive coordination
- Durable local persistence using SQLite
- End-to-end encrypted synchronization
- Peer-to-peer operation by default (no central server)
- Optional “always-on” mode to improve availability, with explicit privacy trade-offs

A major open question is how to **build and package Aspen** in a way that:

- preserves these guarantees on desktop and mobile platforms
- respects mobile background execution constraints
- avoids treating browser storage or web runtimes as a durability boundary
- allows long-term evolution without a forced rewrite

This ADR captures the foundational technical decisions that follow from these constraints.

---

## Decision Summary

Aspen will be built as a **desktop-first system**, with mobile support as a **full-featured but opportunistic participant**, and an optional **always-on instance** to improve availability.

The system will be structured around a **portable platform** that implements sync, storage semantics, and cryptography, with **apps** responsible for UI, OS integration, and domain behavior.

The platform will be explored as a **Rust-based library**, compiled and embedded into platform applications via stable interfaces.


---

## Technical Foundations

### 1. Local-first execution

- All reads and writes occur against local storage.
- The application functions fully without network connectivity.
- Networking is used only to exchange changes between devices.

**Rationale**
Offline-first is not an optimization; it is a requirement driven by mobile constraints, privacy goals, and UX predictability.

---

### 2. Change-based synchronization

- All mutations are recorded as immutable changes.
- Devices exchange changes rather than full database state.
- Local databases are derived projections of these changes.

**Rationale**
This supports partial sync, interrupted sync, recovery, and deterministic convergence across devices.

---

### 3. Concurrent editing without coordination

- The same item may be edited on multiple devices concurrently.
- No locks, leases, or check-out semantics are used.
- Conflicts are resolved via deterministic merge behavior.

**Rationale**
Locks are incompatible with offline-first behavior and hostile to real-world mobile usage. Mergeable operations move complexity from users to the system.

---

### 4. SQLite as derived local storage

- Each device maintains a local SQLite database.
- SQLite is treated as derived state, not the source of truth.
- Databases may be rebuilt from recorded changes if required.

**Rationale**
SQLite provides strong local performance and query capability while allowing recovery, repair, and schema evolution.

---

### 5. Capability-based device behavior

Devices are differentiated by **capability and availability**, not authority:

- **Complete devices**  
  Devices that have synced all known data.

- **Always-on devices**  
  Devices expected to be reachable for long periods, improving availability for others.

- **Opportunistic devices**  
  Devices (typically mobile) that may be offline or unreachable for extended periods but are fully capable of creating and editing data.

A user may have multiple complete or always-on devices simultaneously.

**Rationale**
This avoids implicit hierarchy and reflects real-world multi-device usage.

**Notes**
- “Complete” is a *situational* state: devices may be temporarily incomplete due to intermittent connectivity.
- The system’s semantic model remains “replicate authoritative data per space”; this ADR does not introduce partial synchronization *within* a space.

---

## Build and Runtime Strategy

### 6. Desktop-first packaging

- Desktop platforms (macOS, Windows, Linux) are the primary environment for:
  - durable local storage
  - long-lived execution
  - strong key custody
  - “always-on” behavior when configured by the user

**Rationale**
Desktop operating systems are the only platforms that reliably support these capabilities without artificial constraints.

---

### 7. Mobile as full-featured, opportunistic

- Mobile applications maintain local state and SQLite databases.
- All editing and creation works offline.
- Synchronization occurs when the OS permits (foreground and limited background windows).

**Rationale**
This aligns with iOS and Android execution models without degrading functionality.

---

### 8. Platform implementation (recommended: Rust)

The platform system (sync logic, cryptography, storage semantics, change processing) should be implemented as a **portable native library** with a stable API boundary.

**Recommended approach: Rust-based platform**

- Compiled for desktop and mobile targets.
- Embedded via FFI into native platform applications.
- UI and OS integration implemented per-platform.

**Rationale**
- Strong memory-safety guarantees for crypto and sync-heavy code.
- Excellent portability across desktop and mobile.
- No runtime dependency or embedded VM.
- Mature ecosystems for SQLite, cryptography, and networking.

This approach minimizes long-term rewrite risk while preserving platform-appropriate UX and guarantees.

Other languages (e.g., Kotlin Multiplatform, Go, Swift) were considered, but Rust offers the best balance of portability, safety, and long-term viability for the platform problem space.

---

### 9. Helper processes and background execution

- Desktop builds may use helper processes for separation of concerns (sync, indexing).
- The architecture must not *require* a permanently running background daemon to remain correct.
- Distribution constraints (e.g., Mac App Store vs direct download) are treated as an orthogonal concern.

**Rationale**
This preserves flexibility in packaging and avoids premature commitment to a distribution model.

---

## Optional Always-On Mode

### 10. Always-on instances as a convenience trade-off

- Aspen may optionally run on a device intended to be continuously available.
- This may include user-owned hardware or a cloud-hosted instance.
- Always-on instances improve availability but do not gain special authority.

Cloud-hosted instances explicitly trade privacy for convenience and are opt-in.

**Clarification: “always-on peer” vs “relay/mailbox”**
- An **always-on peer device** participates in sync as a normal peer. It holds space keys and can decrypt data for the spaces it’s authorized for. This is the larger privacy trade-off if hosted outside user-controlled hardware.
- A **relay/mailbox** improves reachability but does not hold space keys and cannot decrypt or authoritatively store user data. This is closer to a transport optimization than an always-on peer.

---

## Explicitly Rejected Alternatives

- Web-only platform used as the primary vault
- Lock- or lease-based coordination
- Single authoritative device models
- Architectures that assume continuous connectivity

---

## Consequences

### Positive
- Strong offline guarantees
- Credible durability and crypto posture
- Realistic mobile support without degraded UX
- Clear path to improved availability without hidden trade-offs

### Negative
- Increased implementation complexity
- Need for careful API and FFI design
- Multi-platform development effort is unavoidable

These trade-offs are accepted as necessary to meet the product’s goals.

---

## Related documents

For the underlying invariants and semantics this ADR builds on, see:

- `docs/architecture/overview.md`
- `docs/architecture/sync-model.md`
- `docs/architecture/sync-topology.md`
- `docs/architecture/key-management.md`
- `docs/architecture/device-and-recovery.md`

Related ADRs:
- `docs/design-decisions/0001-authoritative-data-is-append-only-ops-and-blobs.md`
- `docs/design-decisions/0002-sqlite-is-derived-local-state-only.md`
- `docs/design-decisions/0005-per-user-identity-with-tofu-and-continuity.md`
- `docs/design-decisions/0008-derived-intelligence-is-local-only-and-non-authoritative.md`
- `docs/design-decisions/0009-spaces-are-the-unit-of-replication-and-encryption.md`
