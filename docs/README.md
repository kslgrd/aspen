# Aspen Docs

Aspen is a **local-first platform** for building apps that respect user agency, privacy, and data ownership. It’s not a single app—it's also a shared substrate (identity, encryption, storage, sync) that multiple apps can build on top of. See: [`vision`](vision.md).

---

## Architecture

Aspen is shaped by a few invariants that show up everywhere in the docs:

- **Local-first is the default**: every device works fully offline; sync is additive.
- **All replicated data is end-to-end encrypted**: infrastructure may help with transport, but it can’t read or interpret content.
- **Spaces are the primary boundary** for visibility, permissions, encryption, and replication.
- **Authoritative data is append-only** (ops + blobs); **derived state** (SQLite indexes, projections) is local-only and rebuildable.
- **Cross-space relationships must not leak**: links across spaces live in **overlay spaces**.

### Reading path (recommended order)

1) **System shape and invariants**
   - [`architecture/overview.md`](architecture/overview.md) — platform vs apps, authoritative vs derived, spaces + overlays, core invariants.

2) **What the data _is_**
   - [`architecture/data-model.md`](architecture/data-model.md) — users/devices/spaces, records + revisions, blobs, links + link revisions.

3) **What the system guarantees**
   - [`architecture/security-model.md`](architecture/security-model.md) — threat model, TOFU + identity continuity, space-scoped encryption, enforcement.

4) **How data moves**
   - [`architecture/sync-model.md`](architecture/sync-model.md) — CRDT convergence, append-only logs, blobs, anti-entropy reconciliation.
   - [`architecture/sync-topology.md`](architecture/sync-topology.md) — peer-to-peer, transport neutrality, optional relays/mailboxes.

5) **How keys and devices work in real life**
   - [`architecture/key-management.md`](architecture/key-management.md) — identity/device/space keys, storage + unlock models, adding devices, rotation.
   - [`architecture/device-and-recovery.md`](architecture/device-and-recovery.md) — device lifecycle, revocation, realistic recovery constraints.

6) **What “delete” means**
   - [`architecture/deletion.md`](architecture/deletion.md) — semantic deletion vs physical erasure, shared-space behavior, redaction/rotation limits.

---

## Documentation structure

- [`architecture/`](architecture/) — the conceptual model (invariants, boundaries, “how it’s shaped”).
- [`design-decisions/`](design-decisions/) — ADRs: the “why” behind specific constraints and choices.
- [`opportunity-backlog/`](opportunity-backlog/) — product thinking: problems, MVP notes, and prioritization.
- [`journal/`](journal/) — day-to-day notes and discovery.

