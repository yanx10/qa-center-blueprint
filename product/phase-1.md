# Phase 1 — Change Intelligence Foundation

## Overview

**Goal:** Deliver a working, production-safe Change Intelligence capability for a controlled pilot group without disrupting any existing QA Center workflows.

**Status:** Milestone 0 Complete — Blueprint Reconciliation in Progress  
**Target date:** [OPEN] — to be set after blueprint reconciliation is reviewed and approved

---

## Context

QA Center is an actively used brownfield application. Phase 1 introduces a new capability — Change Intelligence — as a strictly additive extension. Existing users are unaffected throughout Phase 1.

Phase 1 does not modify: test runs, test suites, release reports, Development Gates, AI Studio, the MCP Server, existing API contracts, or the database schemas of any existing table.

---

## Objectives

1. Enable QA to compare a PR diff against its requirements and receive a structured analysis without leaving QA Center.
2. Generate targeted manual test-case proposals and Playwright automation proposals from the analysis.
3. Route approved test cases into the existing test-case library via the existing import workflow.
4. Validate the feature's usefulness and accuracy with a small internal pilot group before broader rollout.
5. Establish the implementation patterns (feature flag, API namespace, component namespace, migration conventions) that subsequent phases will follow.

---

## In Scope

| Milestone | Description | Status |
|-----------|-------------|--------|
| M0 | Existing system discovery — inspect the application repository before implementing anything | COMPLETE — 2026-07-26 |
| M1 | Isolated feature shell — route, navigation, feature flag, empty state | NEXT — blocked on blueprint reconciliation approval |
| M2 | Manual input analysis — pasted diff and requirement text, AI analysis, saved record | [OPEN] |
| M3 | Requirement comparison — coverage status per requirement, evidence, human review | [OPEN] |
| M4 | Risk and regression analysis — risk findings, impacted areas, regression recommendations | [OPEN] |
| M5 | Manual test generation — proposed test cases with approve/reject/import workflow | [OPEN] |
| M6 | Playwright proposals — reviewable automation code proposals | [OPEN] |
| M7 | Existing context integration — optional project, release, test suite, and PRD association | [OPEN] |
| M8 | Persistence and auditability — complete audit trail per analysis | [OPEN] |
| M9 | Controlled team pilot — enabled for a small group, metrics collected | [OPEN] |

---

## Out of Scope for Phase 1

The following are explicitly deferred to Phase 2 or later (pending Milestone 10 expansion decisions):

- GitHub API integration (auto-fetch PR diff)
- Jira auto-linking (auto-fetch story from PR branch name)
- MCP tools for Change Intelligence
- Development Gate integration
- Release Quality Report aggregation
- Repository-aware automation generation (reading test file structure)
- Multi-agent AI framework
- Organization-level GitHub OAuth credentials

---

## Exit Criteria

Phase 1 is complete when:

- [ ] Milestones 0 through 9 are complete
- [ ] All backward compatibility criteria (BC-01 through BC-15) pass on the final milestone
- [ ] A pilot has been run with a small user group
- [ ] A written pilot findings summary exists
- [ ] Milestone 10 expansion decisions have been made based on pilot data
- [ ] No existing QA Center workflow has been broken at any point during Phase 1

---

## Technology Stack

Phase 1 reuses the existing technology stack without additions:

| Concern | Technology | Confirmed Version |
|---------|-----------|-----------------|
| Framework | Next.js App Router | 16.2.11 |
| UI library | React | 19.2.4 |
| Language | TypeScript | 5.x |
| Styling | Tailwind CSS | v4 |
| Database | PostgreSQL | — |
| Database client | postgres.js | 3.4.9 |
| Authentication | Auth.js with Auth0 OIDC and dev credentials | v5 beta.32 |
| AI | Anthropic Claude API (existing `generateStructuredAIResponse<T>()` wrapper) | SDK 0.39.0 |
| Validation | Zod (frontend only) | v4.4.3 |
| Deployment | Docker multi-stage standalone build; Kubernetes/ArgoCD | — |

No new infrastructure is introduced in Phase 1.

---

## Dependencies

- [x] Milestone 0 must complete before Milestone 1 begins. — COMPLETE
- [x] The existing Anthropic API key and configuration are accessible via `integration_settings` table and env vars. — CONFIRMED
- [x] The existing test-case import mechanism (`test_cases` + `test_steps` tables and accept flow) is inspectable and reusable. — CONFIRMED
- [x] The AI Studio schema is known. — CONFIRMED (see `database-and-ai-studio.md` in the discovery documents)
- [x] Feature flag mechanism decided. — RESOLVED: `CHANGE_INTELLIGENCE_ENABLED` env var (D-012)
- [ ] Blueprint reconciliation must be reviewed and approved before Milestone 1 implementation begins.
- [ ] Large-diff chunking strategy must be defined before Milestone 2 is considered production-ready (OD-010).

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| PR diffs exceed Claude context window | Confirmed risk | Medium | Define `MAX_DIFF_TOKENS` and truncation behavior before M2. Test with large diffs in early M2 work. |
| AI analysis quality is insufficient to generate trust in pilot | Medium | High | Explicit evidence and confidence display on every output; `unable_to_verify` fallback; collect pilot feedback for M10 |
| Regression in existing features introduced by Milestone 2+ migration | Low | High | Migration (`031_change_intelligence.sql`) tested against production schema copy before merge. New tables only — no ALTER TABLE. |
| Auth error in a Change Intelligence API route | Low | Low | All routes call `requireAuth()` explicitly; confirmed pattern from M0 (D-014) |
| AI Studio component was not reusable, requiring parallel implementation | Confirmed — resolved | Low | Parallel Change Intelligence review UI confirmed as the correct approach (D-009). No blocker. |
| Feature flag approach doesn't support per-user pilot | Low | Low | M9 pilot will extend the flag with a `change_intelligence_pilot_users` table or equivalent (OD-012). Decision deferred to M9 design. |
