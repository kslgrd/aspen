# Opportunity Backlog

This directory is where we do product discovery for Aspen.

We use a lightweight Opportunity Backlog workflow:

- Define problems (in user language)
- Reframe as opportunities ("How might we...")
- Propose hypotheses + success metrics
- Design minimum viable experiments (MVEs)
- Run experiments and record decisions

## Structure

- `problem-template.md` is the canonical template for new problems.
- Each project gets its own folder (current: `mvp/`).
- Each project folder contains:
  - `[project]-overview.md` (high-level context and goals)
  - `problems/` (one file per problem; problems that change together live together)

## Naming problems

Use short, descriptive filenames (e.g., `url-normalization.md`).

## How to use a problem doc

Each problem doc should be kept current as we learn.

- **Problem statement:** Use the syntax "When..., I want to..., so I can..., but...".
- **Evidence:** Concrete signals (user feedback, observations, analytics, constraints) with links where possible.
- **Opportunities:** One or more "How might we..." statements (avoid embedding solutions).
- **Hypotheses:** Multiple are fine. Pair them with clear success metrics.
- **Possible Concepts (MVE):** The smallest experiment that can produce a signal (effort + what we learn).
- **Experiments:** Log what we actually tried and the outcome (pending/validated/invalidated/inconclusive).
- **Notes/Links:** Capture important context and pointers to relevant docs/ADRs.
