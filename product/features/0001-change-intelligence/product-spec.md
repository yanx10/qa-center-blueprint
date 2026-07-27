# Product Spec — Change Intelligence

## Problem Statement

QA teams are the last line of defense before a release, but they lack a systematic way to verify that what Product intended is what Engineering actually implemented. Test plans are written from requirements, not from diffs. When code changes diverge from spec, QA often discovers the gap during manual testing or — worse — after the release. The cost is rework, escaped defects, and eroded trust between product, engineering, and QA.

Change Intelligence closes this gap. Given a PR diff and one or more requirement sources, QA Center compares the intended behavior against the implementation, identifies coverage gaps and risks, and produces focused testing recommendations.

---

## Goals

1. Enable QA to understand a PR's intended behavior and implementation in a single, structured view.
2. Surface requirement-to-implementation gaps before manual testing begins.
3. Generate targeted manual test proposals and Playwright automation proposals based on the actual change.
4. Integrate with the existing QA Center test-case library and review workflow rather than creating a parallel system.
5. Support controlled rollout so existing users are unaffected until the team is ready to adopt the feature.

## Non-Goals

- Replacing the existing AI Studio PRD-to-test-case workflow.
- Replacing or modifying existing test runs, test suites, release reports, or Development Gates in Phase 1.
- Fully automated GitHub PR integration in Phase 1 (manual diff paste is acceptable for the initial release).
- Executing, committing, or pushing any generated code.
- Creating a second, independent AI configuration system.

---

## Existing Platform Context

Change Intelligence is being built on top of an actively used QA Center application. The platform is a brownfield system with real production data and real daily users. The following capabilities already exist and must remain fully functional:

**Technology stack (confirmed by Milestone 0):** Next.js 16.2.11 (App Router), React 19.2.4, TypeScript 5, Tailwind CSS v4, PostgreSQL, postgres.js 3.4.9, Auth.js v5 beta.32 with Auth0 OIDC and dev credentials provider, Anthropic SDK 0.39.0, Zod v4.4.3 (frontend only), Radix UI, Docker multi-stage standalone build, Kubernetes/ArgoCD deployment.

**Existing features:**
- Test Cases — create, edit, version, bulk-manage, tag, prioritize
- Test Suites — organize cases, clone suites, reuse cases
- Test Runs — execute suites, record step-level results, notes, attachments
- My Work — personal work queue
- Releases — track release and staging dates, code freeze, target dates
- Release Quality Reports — AI-assisted quality reports from Jira data
- Development Gates — six-phase development lifecycle tracking
- AI Studio — PRD-to-test-case generation with review, approval, and import
- MCP Server — expose QA Center capabilities to Claude Desktop, Cursor, and other MCP clients
- Settings — projects, users, platforms, Jira, Anthropic, thresholds, run templates, MCP API keys

**Existing routes:**
```
app/api/
app/releases/
app/reports/
app/my-work/
app/ai-studio/
app/settings/
```

**Existing library structure:**
```
lib/db/
lib/ai/
lib/jira.ts
lib/auth.ts
lib/encryption.ts
```

None of these routes, tables, behaviors, or API contracts may be removed, renamed, or changed in a breaking way as part of this feature.

---

## Brownfield Product Strategy

Change Intelligence is additive. The strategy has four pillars:

**1. Additive-only changes.** New routes, new tables, new components, new API endpoints. Nothing removed or renamed. All new database columns are nullable or have safe defaults. Existing records require no migration.

**2. Controlled rollout.** The feature is gated behind a server-side environment variable (`CHANGE_INTELLIGENCE_ENABLED`). When the flag is off, no navigation item is shown, no new routes are accessible, and every existing workflow runs exactly as before. See D-012.

**3. Reuse existing infrastructure.** The existing Anthropic API wrapper, Jira integration settings, authentication patterns, project model, user model, release model, and test-case library are reused. No second configuration system is created.

**4. No forced migration.** Existing users are not required to adopt Change Intelligence. The feature can be disabled at any time. Failure in Change Intelligence must not prevent access to any existing QA Center functionality.

---

## Feature Overview

PR Intelligence is the first user-facing capability of Change Intelligence.

The user provides:
- A PR diff (pasted or manually supplied in Phase 1)
- One or more requirement sources (PRD text, Jira story, acceptance criteria, Markdown spec, or pasted requirement text)

QA Center processes these inputs and produces:
- A change summary
- Extracted and structured requirements
- A requirement-to-implementation comparison with coverage status for each requirement
- Impacted product areas and regression recommendations
- Manual test-case proposals
- Playwright automation proposals

All AI conclusions display evidence, confidence level, and uncertainty. Unable-to-verify is used when runtime or repository context is insufficient to reach a conclusion.

Approved test cases enter the existing QA Center test-case review and import workflow. No AI-generated test case enters an active suite without human approval.

---

## Current vs Extended Workflow

**Current workflow:**

```
PRD
  ↓
AI Studio
  ↓
Generated Test Cases
  ↓
QA Review
  ↓
Test Suite
  ↓
Test Run
  ↓
Release Quality Report
```

**Extended workflow with Change Intelligence:**

```
Requirement Sources
  ↓
Pull Request
  ↓
Change Intelligence Analysis
  ↓
Requirement-to-Implementation Comparison
  ↓
Risk and Regression Analysis
  ↓
Generated Test Cases
  ↓
Change Intelligence Review Workflow  ← separate review UI (not AI Studio)
  ↓
Existing Test Case Library and Suites  ← reused
  ↓
Existing Test Runs  ← unchanged
  ↓
Existing Release Quality Reports  ← unchanged
```

Change Intelligence connects existing parts of the product. It does not replace them.

---

## Relationship to Existing AI Studio

AI Studio already supports PRD-to-test-case generation. It accepts a PRD document, generates test cases using Claude, and provides a review-approve-reject-import workflow.

Change Intelligence adds capabilities that AI Studio does not have:

| Capability | AI Studio | Change Intelligence |
|-----------|-----------|-------------------|
| Input | PRD document | PR diff + requirement sources |
| Scope | Test generation from spec | Comparison of spec vs implementation |
| Requirement extraction | No | Yes |
| Implementation evidence | No | Yes |
| Gap and risk analysis | No | Yes |
| Regression recommendations | No | Yes |
| Playwright proposals | No | Yes |
| Test-case generation | Yes | Yes (Phase 1 Phase 5) |

AI Studio remains intact and continues to support its existing workflow independently. Change Intelligence does not reuse AI Studio's tables or review component:

- **Tables:** AI Studio uses PRD-centric tables (`prd_documents`, `ai_generation_sessions`, `ai_generated_test_cases`, `prd_gaps`) that are not compatible with Change Intelligence's diff-centric data model. Change Intelligence uses its own tables. See D-008 in `decisions.md`.
- **Review interface:** The AI Studio review UI is embedded in a 1,353-line page component and is not safely reusable. Change Intelligence implements its own review UI under `components/change-intelligence/`. See D-009 in `decisions.md`.

The import path to the permanent test-case library is shared: approved Change Intelligence test cases are written to the same `test_cases` and `test_steps` tables that AI Studio uses, preserving a unified test library.

---

## Relationship to Release Quality Reports

Release Quality Reports operate at the release level. They aggregate data across all test runs associated with a release and produce metrics such as DRE, escape rate, fix-fail rate, and test coverage.

Change Intelligence operates at the individual-change level. Each analysis is scoped to a single PR and its associated requirements.

In a future version, change-level findings may be aggregated into release-level reporting — for example, surfacing the number of analyzed PRs, the overall requirement coverage rate, or the number of risk findings that were verified by QA.

Phase 1 does not modify Release Quality Reports in any way.

---

## User Value

A QA engineer reviewing a PR can answer the following questions in minutes rather than hours:
- What was this change supposed to do?
- What did it actually do?
- Which requirements are fully covered, partially covered, or missing?
- What areas of the product are at risk of regression?
- What should I test manually?
- What automation should I add?

---

## Phase 1 Inputs

**Required minimum inputs:**
1. PR diff — pasted or manually supplied git diff text
2. Requirement source — at least one of: PRD text, Jira story, acceptance criteria, Markdown spec, or pasted requirement text

**Supported requirement source types:**
- PRD text (free-form or from existing AI Studio PRD documents)
- Jira story (manual paste in Phase 1; Jira API integration may be added later)
- Acceptance criteria
- Markdown specification
- Pasted requirement text

**Optional context (Milestone 7):**
- Existing QA Center test cases for the affected area
- Existing test suites
- Release association
- Project association

---

## Phase 1 Outputs

1. Change summary
2. Requirement summary
3. Structured requirements
4. Requirement-to-implementation comparison:
   - Implemented requirements
   - Partially implemented requirements
   - Potentially missing requirements
   - Ambiguous requirements
   - Unable-to-verify requirements
   - Unexpected implementation behavior
5. Impacted product areas
6. Risk assessment
7. Regression recommendations
8. Manual test-case proposals
9. Playwright automation proposals
10. Evidence and reasoning for all AI conclusions
11. QA recommendation
12. Human review status
13. Saved analysis record

---

## Success Metrics

| Metric | Baseline | Target | How Measured |
|--------|---------|--------|--------------|
| Time to understand a PR's test surface | [OPEN] | Reduced (pilot measurement) | Team survey |
| Generated test acceptance rate | [OPEN] | [OPEN] | Pilot data |
| Automation proposal acceptance rate | [OPEN] | [OPEN] | Pilot data |
| False-positive findings per analysis | [OPEN] | [OPEN] | Pilot data |
| Missed requirements per analysis | [OPEN] | [OPEN] | Pilot data |
| User trust score | — | [OPEN] | Pilot survey |

Baselines and targets will be established from Milestone 9 pilot data. Do not fabricate values.

---

## Confirmed Facts (from Milestone 0)

The following were assumptions before Milestone 0. All are now confirmed.

- [DECISION] The existing Anthropic API wrapper `generateStructuredAIResponse<T>()` in `lib/ai/anthropic.ts` is reused for Change Intelligence AI calls without modification.
- [DECISION] The existing Auth.js v5 session provides `session.user.id` as a UUID that directly references `users.id`. Feature-flag checking requires no schema changes — only a new `lib/features.ts` helper and a server env var.
- [DECISION] The existing postgres.js connection pool in `lib/db/index.ts` is reused. No second connection layer.
- [ASSUMPTION] PR diffs in Phase 1 may exceed Claude's context window. A `MAX_DIFF_TOKENS` truncation strategy must be defined before Milestone 2 is production-ready. This is not an assumption — it is a known risk that must be mitigated.

## Resolved Questions

All open questions below were resolved by Milestone 0 discovery:

| Question | Answer |
|----------|--------|
| AI Studio review component: shared or page-specific? | Page-specific — 1,353-line monolithic component. Not reusable (D-009). |
| AI Studio schema | `prd_documents`, `ai_generation_sessions`, `ai_generated_test_cases`, `prd_gaps`. Not reused by Change Intelligence (D-008). |
| Database migration naming | Numbered SQL files. Next safe number: `030` (D-010). |
| Feature-flag architecture | No mechanism exists. Must introduce env var `CHANGE_INTELLIGENCE_ENABLED` (D-012). |
| Existing feature flag patterns | None. Change Intelligence introduces the first feature flag in the application. |
| API response envelope | `{ success: boolean, data?, error? }` throughout (D-015). |
