# Feature 0001 — Change Intelligence

## Summary

Change Intelligence is an AI-powered capability that compares a PR diff against one or more requirement sources, identifies gaps between the intended behavior and the implementation, and generates targeted test proposals. It closes the feedback loop between product intent and engineering execution — enabling QA engineers to answer "what was this change supposed to do, and did it actually do that?" before manual testing begins.

Phase 1 uses manual text input (pasted PR diffs and requirement text) and does not require GitHub API integration (D-004). See `product-spec.md` for the full feature overview.

---

## Status

| Field | Value |
|-------|-------|
| ID | 0001 |
| Phase | 1 |
| Status | In Progress — M1 Complete, M2 Specification Ready |
| Owner | [OPEN] |
| M0 | COMPLETE — 2026-07-26 |
| M1 | COMPLETE |
| M2 | Specification Complete — Ready for Implementation |
| M3–M14 | [OPEN] |

---

## Contents

| File | Purpose |
|------|---------|
| `product-spec.md` | Problem statement, goals, and feature overview |
| `user-experience.md` | User journeys, personas, and UX principles |
| `requirements-model.md` | Functional and non-functional requirements |
| `ai-workflow.md` | AI agent roles, workflows, and prompt strategy |
| `data-model.md` | Entities, relationships, and storage decisions |
| `api-design.md` | API surface — endpoints, contracts, error handling |
| `acceptance-criteria.md` | Testable acceptance criteria for each requirement |
| `implementation-plan.md` | Engineering phases, milestones, and task breakdown (M0–M14) |
| `decisions.md` | Design decisions D-001–D-022 and their rationale |
| `milestones/` | Per-milestone detailed specifications |
| `milestones/m2-analysis-persistence-foundation.md` | M2 full specification — schema, API contracts, 36 acceptance criteria |

---

## Related

- Architecture ADRs: `architecture/adr/` (link specific ADRs when created)
- Blueprint volume: [OPEN]
