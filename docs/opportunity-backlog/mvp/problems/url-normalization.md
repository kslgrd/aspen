# Problem: URL normalization and identity

**Problem statement:**  
When I save or re-share a URL, I want Aspen to recognize the same page,
so I can maintain a coherent history, but small URL variations can
cause duplicates or accidental merges.

**Evidence**

- Internal design constraints (reference-mode URL items).
- Common URL variants include tracking params, redirects, and canonical tags.
- Re-sharing the “same” URL is a first-class event in the MVP.

---

## Opportunities

- How might we keep URL identities stable over time without merging distinct resources?

---

## Hypotheses

- We believe a canonicalization ruleset with explicit exclusions will achieve stable identity without unsafe merges.
- We believe storing both raw and normalized URLs will achieve auditability without compromising identity stability.
- We believe per-domain overrides will achieve higher accuracy than a single global ruleset.

**Success metrics**

- Duplicate URL captures reduced by 80% without increasing false merges.
- Re-sharing the same URL results in a revision event in 95%+ of cases.
- False merges remain below 1% in a test corpus.

---

## Possible Concepts (MVE)

- **Hypothesis:** Canonicalization ruleset with exclusions  
  **MVE:** Build a small URL test corpus and run a ruleset prototype  
  **Effort:** low  
  **Signal:** false merge rate vs. duplicate rate

- **Hypothesis:** Store raw + normalized URLs  
  **MVE:** Implement dual storage for a small sample and audit conflicts  
  **Effort:** low  
  **Signal:** ability to resolve ambiguity without manual cleanup

- **Hypothesis:** Per-domain overrides  
  **MVE:** Add overrides for 5 high-traffic domains and compare outcomes  
  **Effort:** medium  
  **Signal:** reduction in false merges and duplicates

---

## Experiments

### [Experiment description]

- **Result:** pending
- **Evidence:** (none yet)
- **Decision:** (none yet)

---

## Notes

- Known to be tricky (redirects, tracking params, canonical tags, etc.).
- We should preserve raw URLs regardless of normalization decisions.
- This is explicitly out of scope for MVP implementation, but needs exploration.
- The normalization result is likely to become part of the long-lived identity model for URL sources, so we should treat changes as migrations.

## Links

- [`mvp-overview.md`](../mvp-overview.md)
- [`0007-external-sources-have-stable-identities-and-revisions.md`](../../../design-decisions/0007-external-sources-have-stable-identities-and-revisions.md)
