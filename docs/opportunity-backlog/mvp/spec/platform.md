# Platform Kernel (MVP Spec)

This document describes the platform kernel we need to ship the MVP: responsibilities, boundaries, and a minimal API surface.

Source constraints:
- Desktop-first, portable platform library. (See [`0011-technical-foundations.md`](../../../design-decisions/0011-technical-foundations.md).)
- Apps are lenses; platform owns data/identity/crypto. (See [`overview.md`](../../../architecture/overview.md).)

---

## Goals (MVP)

- Provide a single-device, single-user platform that is “real” with respect to:
  - identity keys
  - device keys
  - space keys (Inbox)
  - encrypted authoritative storage
- Expose a stable interface to the macOS app for:
  - capture (URL, file reference)
  - timeline querying
  - incremental search
  - subscriptions for updates
- Keep seams that allow later multi-device sync without rewrites of the authoritative model.

---

## Responsibilities (MVP)

### Identity + device bootstrap

On first launch:
- generate user identity key
- generate device key
- create Inbox space (non-deletable) and its key material

Notes:
- Recovery and device pairing are out of scope for MVP UX, but key material must be stored in a way that is compatible with future expansion. See:
  - [`key-management.md`](../../../architecture/key-management.md)
  - [`device-and-recovery.md`](../../../architecture/device-and-recovery.md)
  - [`key-unlocking-and-local-key-storage.md`](../problems/key-unlocking-and-local-key-storage.md)

### Authoritative operations

The platform is the only component that can:
- encrypt/decrypt authoritative payloads
- append operations to the space log
- read/replay operations for rebuild

### Derived state maintenance

The platform is responsible for:
- projecting authoritative operations into SQLite (derived views)
- maintaining (or updating) local-only search indexes
- rebuilding derived state on demand

### Subscriptions

The platform emits change notifications such that the app can:
- update the timeline when new events are appended
- update search results incrementally

---

## Minimal API surface (conceptual)

This is intentionally conceptual (names not final) — it defines what the macOS app needs from the platform.

### Lifecycle / state
- `platform_init(app_data_dir) -> PlatformHandle`
- `platform_state() -> { locked | unlocked }`
- `platform_unlock(...)` (mechanism TBD; see problem doc)
- `platform_lock()`

### Capture
- `capture_url(raw_url, surface_metadata) -> CaptureResult`
- `capture_file(path, surface_metadata) -> CaptureResult`

CaptureResult includes:
- whether this created a new item vs appended a re-share event
- the new timeline event id
- enough metadata to update the UI immediately

### Querying
- `timeline_list(limit, cursor, filter?) -> [TimelineEvent]`
- `search(query, limit, cursor) -> [TimelineEvent]` (or event ids + ranking)

### Subscriptions
- `subscribe_timeline() -> stream<EventChange>`
- `subscribe_search_index_state() -> stream<IndexChange>`

---

## Open questions (tracked as problems)

- How do we define the platform API boundary and embed it in a macOS app (Rust ↔ Swift, threading model, error surfaces)?
  - [`platform-api-and-ffi-boundary.md`](../problems/platform-api-and-ffi-boundary.md)
- What is the minimal record envelope/versioning strategy for durability?
  - [`record-schema-and-versioning.md`](../problems/record-schema-and-versioning.md)
- What is the CRDT surface for “current revision” selection (even if MVP runs single-device)?
  - [`crdt-surface-and-conflict-semantics.md`](../problems/crdt-surface-and-conflict-semantics.md)
