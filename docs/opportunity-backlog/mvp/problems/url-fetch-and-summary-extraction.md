# Problem: URL fetch and bounded summary extraction

**Problem statement:**  
When I paste a URL into Aspen, I want it to capture a useful offline summary quickly and reliably,
so I can search and skim later, but web fetching and content extraction are messy (redirects, paywalls,
dynamic pages, huge documents) and can easily become slow, flaky, or privacy-risky.

**Evidence**

- MVP requires: on add, fetch page, extract bounded summary, and persist as encrypted blob. (See [`mvp-overview.md`](../mvp-overview.md).)
- URL normalization is already identified as a separate identity problem, but extraction quality and performance can make the MVP feel broken even if identity is correct.
- Offline-first constraints imply we must define what happens when a URL can’t be fetched (network down, blocked, etc.).

---

## Opportunities

- How might we produce a consistently “good enough” bounded summary for search and skimming without building a full web archiving system?
- How might we make failures legible (and recoverable) without adding complex background crawling?

---

## Hypotheses

- We believe a conservative extraction pipeline (HTML → readability → text normalization → truncate by tokens/bytes) will achieve useful summaries
  while keeping implementation scope bounded.
- We believe storing both extraction metadata (e.g., fetch time, content type, final URL) and the summary blob will achieve debuggability
  without making derived state authoritative.

**Success metrics**

- ≥90% of pasted URLs produce a non-empty summary in <3 seconds on a typical connection (excluding extreme cases).
- Failures are explicit and actionable (retry, view raw URL, etc.), with <5% “silent missing content” reports.
- Summaries are bounded (size and time) and safe to index locally.

---

## Possible Concepts (MVE)

- **Hypothesis:** Conservative extraction pipeline  
  **MVE:** Build a small corpus of 50 URLs and run an extractor prototype; measure extraction success, latency, and summary usefulness  
  **Effort:** medium  
  **Signal:** success/latency distribution; qualitative usefulness ratings

- **Hypothesis:** Define offline behavior early  
  **MVE:** Choose one offline policy (fail fast vs pending capture state) and test with 10 users/dev sessions  
  **Effort:** low  
  **Signal:** reduced confusion; clear mental model

---

## Experiments

### [Experiment description]

- **Result:** pending
- **Evidence:** (none yet)
- **Decision:** (none yet)

---

## Notes

- This problem is about extraction semantics and UX expectations, not URL identity. (See `url-normalization.md`.)
- MVP explicitly avoids background crawling/refresh; extraction should be on-demand at capture time unless we decide on a small pending state.

## Links

- [`mvp-overview.md`](../mvp-overview.md)
- [`ingestion.md`](../spec/ingestion.md)
- [`url-normalization.md`](url-normalization.md)
- [`0007-external-sources-have-stable-identities-and-revisions.md`](../../../design-decisions/0007-external-sources-have-stable-identities-and-revisions.md)
