# MVP Spec

This folder contains the **build spec** for Aspen’s MVP: the concrete systems, boundaries, and contracts needed to implement the MVP described in [`mvp-overview.md`](../mvp-overview.md).

This spec is intentionally incremental:

- If something is unknown or under-specified, we record it as a problem in [`problems/`](../problems/) (using [`problem-template.md`](../../problem-template.md)) and link it from the relevant spec section.
- Specs should prefer **contracts and invariants** over implementation details, but include enough detail to unblock engineering.

## Index

- [`systems.md`](systems.md) — system decomposition and responsibilities
- [`platform.md`](platform.md) — platform kernel responsibilities + API surface (MVP scope)
- [`storage-and-indexes.md`](storage-and-indexes.md) — authoritative store + derived SQLite/search rebuildability
- [`ingestion.md`](ingestion.md) — URL and file capture pipelines (reference mode only)
- [`macos-app.md`](macos-app.md) — macOS app UX, capture surfaces, and UI flows
- [`testing-and-invariants.md`](testing-and-invariants.md) — acceptance tests and invariants (MVP “proofs”)
