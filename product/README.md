# Product

This directory is the product management source of truth for QA Center.

It captures what we are building, in what order, and why — independent of the narrative documentation in `docs/` and the engineering detail in `architecture/`.

---

## Contents

| File / Directory | Purpose |
|-----------------|---------|
| `roadmap.md` | Phase-level view of the product timeline |
| `backlog.md` | Prioritized feature backlog |
| `phase-1.md` | Detailed scope and goals for Phase 1 |
| `features/` | One directory per feature, containing the full feature definition |

---

## Feature Numbering

Features are numbered sequentially: `NNNN-feature-name`.

Each feature directory contains the complete definition of that feature — product spec, UX, requirements, AI workflow, data model, API design, acceptance criteria, implementation plan, and decisions.

---

## Relationship to Other Directories

- `docs/` — narrative volumes explaining the vision and capabilities
- `architecture/` — engineering-level design artifacts (ADRs, data models, API contracts)
- `product/` — product management artifacts (what, when, why, acceptance criteria)

Product and architecture artifacts for the same feature should cross-reference each other but live in their respective directories.
