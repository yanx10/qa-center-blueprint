# Product Roadmap

This document describes the phase-level plan for QA Center. Each phase is a coherent unit of value that can be piloted, evaluated, and shipped independently.

---

## Existing Platform Baseline

Before reading the phases below, note that QA Center is an actively used platform. The following capabilities already exist and are not part of any roadmap phase — they are the baseline:

- Test Cases, Test Suites, Test Runs, My Work
- Releases and Release Quality Reports
- Development Gates
- AI Studio (PRD-to-test-case generation)
- MCP Server
- Settings (projects, users, platforms, Jira, Anthropic, thresholds, templates, MCP keys)

All roadmap phases are additive extensions to this baseline.

---

## Phase 1 — Change Intelligence Foundation

See [`phase-1.md`](phase-1.md) for detailed scope.

**Goal:** Enable QA to compare a PR diff against requirements, receive a structured analysis, and generate targeted test proposals — without disrupting any existing workflow.

**Status:** Milestone 0 Complete — Blueprint Reconciliation in Progress  
**Target:** [OPEN] — set after blueprint reconciliation is reviewed and approved

**Milestones:**

| # | Milestone | Status |
|---|-----------|--------|
| M0 | Existing System Discovery | COMPLETE — 2026-07-26 |
| — | Blueprint Reconciliation | IN PROGRESS — 2026-07-26 |
| M1 | Isolated Feature Shell | NEXT — blocked on reconciliation approval |
| M2 | Manual Input Analysis | [OPEN] |
| M3 | Requirement Comparison | [OPEN] |
| M4 | Risk and Regression Analysis | [OPEN] |
| M5 | Manual Test Generation | [OPEN] |
| M6 | Playwright Proposals | [OPEN] |
| M7 | Existing Context Integration | [OPEN] |
| M8 | Persistence and Auditability | [OPEN] |
| M9 | Controlled Team Pilot | [OPEN] |

**Key constraint:** Nothing in Phase 1 removes, renames, or modifies existing QA Center capabilities.

---

## Phase 2 — Change Intelligence Expansion

**Goal:** [OPEN] — determined by Milestone 10 expansion decisions, which are based on Phase 1 pilot data.

**Status:** [FUTURE] — Not planned until pilot findings are reviewed.

**Candidates for Phase 2** (not commitments — each requires a decision record):

| Candidate | Depends On |
|-----------|-----------|
| GitHub API integration — auto-fetch PR diffs | Phase 1 pilot demand |
| Jira auto-linking — auto-fetch story from branch name | Phase 1 pilot demand |
| MCP tools for Change Intelligence | Phase 1 pilot demand |
| Development Gate integration — surface CI findings in Gates 3/4 | Phase 1 pilot demand |
| Release Quality Report aggregation | Phase 1 pilot demand |
| Repository-aware automation generation | Phase 1 pilot demand |

---

## Phase 3 — [OPEN]

**Goal:** [OPEN]  
**Status:** [FUTURE] — Not planned until Phase 2 outcomes are known.

---

## Roadmap Principles

1. **Ship incrementally.** Each phase delivers standalone value to real users. No phase is a purely internal stepping stone.
2. **Defer complexity.** Features that require Phase N infrastructure belong in Phase N or later, not in Phase N-1.
3. **Validate before building.** Acceptance criteria and pilot evidence drive the decision to expand, not assumptions.
4. **Protect the baseline.** Every phase must leave all existing QA Center capabilities fully functional.
5. **Pilot before scaling.** Phase 1 ends with a controlled pilot. Phase 2 does not begin without written pilot findings.
