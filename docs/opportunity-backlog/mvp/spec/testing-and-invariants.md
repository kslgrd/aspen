# Testing and Invariants (MVP Spec)

This document turns MVP success criteria into concrete checks.

---

## Acceptance checks (MVP)

### A) SQLite rebuildability

Given:
- an existing local store with at least N captured items (URLs + files)

When:
- SQLite derived DB is deleted
- the app/platform is restarted

Then:
- the timeline reconstructs the same events (count + identities + ordering rule)
- search returns results consistent with the derived extraction inputs

### B) Re-sharing produces new events

Given:
- a URL (or file) already captured

When:
- the same URL (or file) is captured again

Then:
- a new timeline event is appended (history visible)

### C) Offline behavior

Given:
- the network is unavailable

Then:
- capture and search still work for local items
- URL capture behavior is explicit:
  - either it errors clearly (if fetch is required), or it stores a pending capture state (TBD; decide)

---

## Invariants to preserve

- Apps never bypass the platform to access authoritative store or keys.
- Authoritative data is always encrypted at rest.
- Derived state can be discarded and rebuilt without data loss.

---

## Open questions (tracked)

- What behavior should URL capture have when offline (fail fast vs queue/pending)?
  - [`url-fetch-and-summary-extraction.md`](../problems/url-fetch-and-summary-extraction.md)
