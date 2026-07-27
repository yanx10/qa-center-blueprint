# Implementation Plan — Change Intelligence

## Overview

Change Intelligence is built as an additive extension to an actively used QA Center application. The implementation is structured in fifteen milestones (0 through 14). No implementation work begins before Milestone 0 discovery is complete.

Every milestone that writes production code must include regression verification for all existing QA Center workflows. Milestones 1 through 10 are blocked until Milestone 0 is done.

---

## Non-Negotiable Constraints

Before writing a single line of implementation code:

1. The existing application repository must be inspected (Milestone 0).
2. All new code must be additive. No existing routes, tables, columns, or API contracts may be removed or renamed.
3. New database changes use additive migrations only. New columns are nullable or have safe defaults.
4. The feature must be gated behind a feature flag. When disabled, no user-visible change occurs.
5. Failure in Change Intelligence must not affect access to any existing QA Center feature.
6. Every implementation milestone includes regression smoke testing of existing workflows.

---

## Milestone 0 — Existing System Discovery

**Goal:** Understand the actual application before designing anything.

**Status:** COMPLETE — 2026-07-26

**Discovery documents:** `../qa-center/docs/discovery/change-intelligence/` (in the application repository)

### Summary of Confirmed Findings

All eleven inspection areas were completed. Key findings that affect the blueprint:

**Technology stack (confirmed):**
- Next.js 16.2.11 (App Router), React 19.2.4, TypeScript 5, Tailwind CSS v4
- Auth.js v5 beta.32 with Auth0 OIDC and dev credentials provider
- postgres.js 3.4.9 (direct tagged template literals; Drizzle ORM installed but not used for queries)
- Anthropic SDK 0.39.0, Zod v4.4.3 (frontend only)
- Docker standalone build; Kubernetes/ArgoCD deployment

**Authentication:** Middleware protects pages; `/api/*` routes pass through with no auth check. Every Change Intelligence API route must call `requireAuth()` explicitly.

**AI Studio:** The review interface is a 1,353-line monolithic `'use client'` component — not reusable. Change Intelligence must implement its own review UI (D-009).

**Feature flags:** No feature flag mechanism exists in the application. Change Intelligence must introduce the first one (D-012).

**`created_by` convention:** Most existing tables use `TEXT`. Newer tables (`import_sessions`, `qa_reports`) use `UUID REFERENCES users(id)`. Change Intelligence uses UUID FKs (D-011).

**Migration number:** Files exist for 010, 011, 012, 019. Schema.sql contains inline content through 029. Next safe file: `030_change_intelligence.sql` (D-010).

**AI integration:** Synchronous only. No streaming. No retry. Single entry point: `generateStructuredAIResponse<T>()` in `lib/ai/anthropic.ts` (D-016).

**Response envelope:** `{ success: boolean, data?, error? }` — confirmed throughout (D-015).

**Open decisions resolved:** OD-001 through OD-007 all resolved. See `decisions.md`.

---

## Milestone 1 — Isolated Feature Shell

**Goal:** Add the minimum scaffolding for Change Intelligence with zero impact on existing features when disabled.

**Status:** NEXT — begins after blueprint reconciliation is reviewed and approved.

### Work items

- [ ] Add `CHANGE_INTELLIGENCE_ENABLED: z.string().optional()` to `env.ts` server schema
- [ ] Create `lib/features.ts` with `isChangeIntelligenceEnabled(): boolean`
- [ ] Modify `app/layout.tsx` to call `isChangeIntelligenceEnabled()` and pass a boolean prop to `SideNav`
- [ ] Modify `components/SideNav.tsx` to accept `changeIntelligenceEnabled?: boolean` and conditionally include the nav item
- [ ] Create `app/change-intelligence/page.tsx` — server component with `notFound()` guard and empty-state UI
- [ ] [OPTIONAL] Create `app/api/change-intelligence/status/route.ts` — feature-check-only endpoint (no DB access)
- [ ] Set up test infrastructure baseline for future API route tests (even if empty in M1)

### Expected file impact (proposals — subject to implementation verification)

| File | Change |
|------|--------|
| `env.ts` | Add `CHANGE_INTELLIGENCE_ENABLED: z.string().optional()` |
| `lib/features.ts` | New file — `isChangeIntelligenceEnabled()` helper |
| `components/SideNav.tsx` | Add `changeIntelligenceEnabled?: boolean` prop; conditional nav item |
| `app/layout.tsx` | Read flag; pass boolean prop to `SideNav` |
| `app/change-intelligence/page.tsx` | New file — server component, `notFound()` guard, empty state |
| `app/api/change-intelligence/status/route.ts` | Optional — auth + flag check only |

### What Milestone 1 must NOT include

- No database migration (`030_change_intelligence.sql` is Milestone 2)
- No analysis persistence or Change Intelligence tables
- No Anthropic calls
- No requirement extraction, diff processing, test generation, or Playwright generation
- No modification to AI Studio code (`app/ai-studio/page.tsx`, AI Studio API routes)
- No modification to MCP server (`app/api/mcp/route.ts`)
- No modification to Release Quality Reports or Development Gates

### Database changes
None. No migration is introduced in Milestone 1.

### Acceptance criteria

See `acceptance-criteria.md` M1-AC-01 through M1-AC-18.

### Regression requirement
Run full regression smoke test of all existing workflows before merging. See regression checklist at the bottom of this document.

---

## Milestone 2 — Analysis Persistence Foundation

**Goal:** Introduce the database tables and API endpoints that store and manage Change Intelligence analysis records. No AI processing in this milestone.

**Status:** Specification Complete — Ready for Implementation

**Specification:** See `milestones/m2-analysis-persistence-foundation.md` for the full milestone specification.

### Work items

- [ ] Database migration `030_change_intelligence.sql` — introduces `change_analyses` and `change_analysis_inputs` tables
- [ ] `GET /api/change-intelligence/analyses` — paginated list endpoint (excludes content)
- [ ] `POST /api/change-intelligence/analyses` — create draft analysis record
- [ ] `GET /api/change-intelligence/analyses/:id` — retrieve analysis with input metadata (excludes content)
- [ ] `PATCH /api/change-intelligence/analyses/:id` — update title or transition status
- [ ] `POST /api/change-intelligence/analyses/:id/inputs` — add input to draft analysis
- [ ] `DELETE /api/change-intelligence/analyses/:id/inputs/:inputId` — remove input from draft analysis
- [ ] Feature gate check (403) before auth check on all M2 routes
- [ ] `requireAuth()` call at top of every M2 route handler
- [ ] Status machine enforcement: draft → ready → (M3) processing → completed/failed; draft/ready → cancelled
- [ ] Input immutability: inputs can only be added/removed when analysis is in `draft`
- [ ] Content size limit: reject `content` exceeding 500,000 characters
- [ ] Content hash: compute SHA-256 of `content` server-side; never return `content` in GET responses
- [ ] Unit tests for status machine, enum validation, content limit, and content_hash
- [ ] API tests for all 6 endpoints × happy path and all error cases

### Expected file impact (proposals — subject to implementation verification)

| File | Change |
|------|--------|
| `030_change_intelligence.sql` | New migration — `change_analyses`, `change_analysis_inputs`, indexes |
| `app/api/change-intelligence/analyses/route.ts` | New file — GET list, POST create |
| `app/api/change-intelligence/analyses/[id]/route.ts` | New file — GET detail, PATCH update |
| `app/api/change-intelligence/analyses/[id]/inputs/route.ts` | New file — POST add input |
| `app/api/change-intelligence/analyses/[id]/inputs/[inputId]/route.ts` | New file — DELETE remove input |
| `lib/change-intelligence/analyses.ts` | New file — DB query functions |

### What Milestone 2 must NOT include

- No Anthropic API calls of any kind
- No requirement extraction, diff analysis, AI processing, or AI output display
- No GitHub API integration, OAuth flows, or webhook endpoints
- No `ChangeRepository`, `ChangePullRequest`, `ChangeSyncRun`, or any provider entity
- No `change_requirements`, `change_risk_findings`, `change_generated_test_cases`, or `change_playwright_proposals` tables
- No analysis submission form or result display UI (M3 scope)
- No modification to existing QA Center routes, tables, or API contracts
- No `trigger_type = 'webhook'` value set by any M2 code path

### Database changes

New tables via `030_change_intelligence.sql`:
- `change_analyses` (17 columns including nullable M3 result columns)
- `change_analysis_inputs` (9 columns)

See `milestones/m2-analysis-persistence-foundation.md` for the full schema specification.

### Acceptance criteria

See `acceptance-criteria.md` M2-AC-01 through M2-AC-36.

### Regression requirement
Run full regression smoke test of all existing workflows before merging.

---

## Milestone 3 — Manual Input Analysis

**Goal:** Accept pasted requirement text and PR diff, run AI analysis, and save a completed analysis record.

### Work items

- [ ] Input form: requirement source type selector and text area
- [ ] Input form: PR diff text area
- [ ] Trigger AI analysis on a `ready` analysis record
- [ ] Server-side validation of inputs
- [ ] AI workflow: requirement extraction module
- [ ] AI workflow: diff analysis module
- [ ] Output: change summary
- [ ] Output: extracted requirement list
- [ ] Update analysis record with AI results (status → completed or failed)
- [ ] Display analysis result page
- [ ] All AI outputs show evidence, confidence, and uncertainty labels

### Database changes
Introduces `change_requirements` table via an additive migration. See `data-model.md` for schema.

### Does not include
- GitHub API integration
- Requirement-to-implementation comparison (Milestone 4)
- Risk analysis (Milestone 5)
- Test generation (Milestone 6)

### Acceptance criteria

```
Given an analysis is in ready status with at least one pr_diff and one requirement input
When the user triggers analysis
Then QA Center runs the AI pipeline and displays a change summary
And extracted requirements are listed
And the analysis record status transitions to completed
And all AI conclusions show evidence and confidence
```

```
Given the AI call fails
When the user triggers analysis
Then an error is displayed and the analysis status transitions to failed
And the failure does not affect any other QA Center page
```

### Regression requirement
Run full regression smoke test of all existing workflows before merging.

---

## Milestone 4 — Requirement Comparison

**Goal:** Map extracted requirements to implementation evidence and classify each requirement's coverage status.

### Work items

- [ ] AI workflow: requirement-to-implementation mapping module
- [ ] Coverage status classification:
  - Implemented
  - Partially implemented
  - Potentially missing
  - Ambiguous
  - Unable to verify
- [ ] Unexpected implementation behavior detection
- [ ] Evidence display per requirement
- [ ] Confidence level display per requirement
- [ ] Update analysis result page with comparison view
- [ ] Save comparison output to the analysis record

### Acceptance criteria

```
Given an analysis has been run
When the user views the comparison
Then each requirement has a coverage status
And each status is supported by a specific excerpt from the diff
And unable-to-verify is used when evidence is insufficient
```

### Regression requirement
Run full regression smoke test of all existing workflows before merging.

---

## Milestone 5 — Risk and Regression Analysis

**Goal:** Identify impacted product areas, classify risks, and produce regression recommendations.

### Work items

- [ ] AI workflow: risk analysis module
- [ ] Risk category classification (to be defined — examples: data integrity, auth, performance, UI regression)
- [ ] Impacted area identification
- [ ] Regression recommendation generation
- [ ] Evidence display per risk finding
- [ ] Human review status field (unreviewed / acknowledged / disputed)
- [ ] Save risk output to analysis record

### Acceptance criteria

```
Given an analysis has been run
When the user views risk findings
Then each finding names the impacted area and risk category
And each finding shows evidence from the diff
And each finding has a reviewable status
```

```
Given a risk finding is disputed
When the user marks it disputed and adds a note
Then the dispute is saved and the finding is flagged for follow-up
```

### Regression requirement
Run full regression smoke test of all existing workflows before merging.

---

## Milestone 6 — Manual Test Generation

**Goal:** Generate proposed manual test cases from the analysis and route them through the existing review workflow.

### Work items

- [ ] AI workflow: manual test-case generation module
- [ ] Display proposed test cases before any approval
- [ ] Implement or reuse human review interface (approve / reject per proposed case)
- [ ] On approval, import case into the existing test-case library using the existing import mechanism
- [ ] No unreviewed AI case may enter any test suite
- [ ] Implement Change Intelligence review UI in `app/change-intelligence/_components/` — separate from AI Studio (D-009)

### Acceptance criteria

```
Given test cases have been generated for an analysis
When the user reviews the proposals
Then each proposed case is shown individually with approve / reject options
And no case is added to the test-case library until it is explicitly approved
```

```
Given a case is approved
When the user imports it
Then it appears in the existing test-case library using the existing format
And the existing test-case CRUD workflow continues to work for the imported case
```

### Regression requirement
Run full regression smoke test of all existing workflows, with special focus on the AI Studio review and import flow. Milestone 6 must not break AI Studio.

---

## Milestone 7 — Playwright Proposals

**Goal:** Generate reviewable Playwright test proposals based on the analysis.

### Work items

- [ ] AI workflow: Playwright automation proposal generation module
- [ ] Display proposals as reviewable code blocks (no execution, no commit, no PR)
- [ ] Human review status per proposal (accepted / rejected / deferred)
- [ ] Copy-to-clipboard for accepted proposals
- [ ] Save review decisions to analysis record

### Acceptance criteria

```
Given Playwright proposals have been generated
When the user views them
Then proposals are displayed as readable code blocks
And no code is executed or committed automatically
And the user can mark each proposal accepted, rejected, or deferred
```

### Regression requirement
Run full regression smoke test of all existing workflows before merging.

---

## Milestone 8 — Existing Context Integration

**Goal:** Allow optional use of existing QA Center data to improve analysis quality.

### Work items

- [ ] Optional project association (link analysis to an existing project)
- [ ] Optional release association (link analysis to an existing release)
- [ ] Optional test-case context (include relevant existing test cases in the analysis prompt)
- [ ] Optional test-suite context (include suite structure in the analysis)
- [ ] Optional AI Studio PRD reference (link to or include an existing PRD document)
- [ ] All associations are optional — analysis must work without any of them

### Database changes
New nullable foreign keys on `change_analyses` referencing existing tables (projects, releases, etc.). All nullable. No changes to referenced tables.

### Acceptance criteria

```
Given a user submits an analysis without any optional context
Then the analysis runs and produces results normally
```

```
Given a user links an existing test suite to an analysis
When the analysis runs
Then the AI considers the existing test coverage in its recommendations
```

### Regression requirement
Run full regression smoke test of all existing workflows before merging.

---

## Milestone 9 — Repository and Provider Foundation

[FUTURE] — Not planned until M8 is complete and pilot demand is confirmed.

**Goal:** Introduce the database tables and API endpoints that model source control repositories and pull request objects, enabling future automated PR ingestion.

**Status:** [FUTURE] — Blocked on M13 pilot findings.

This milestone introduces `change_repositories`, `change_pull_requests`, and provider credential infrastructure. It does not implement GitHub API calls. It is the schema foundation for M10.

No M9 work begins without a written decision record in `decisions.md`.

---

## Milestone 10 — GitHub Connection and Manual Synchronization

[FUTURE] — Not planned until M9 is complete.

**Goal:** Allow users to connect a GitHub repository and manually trigger synchronization of a specific pull request, populating the PR object and pre-filling the analysis inputs.

**Status:** [FUTURE] — Blocked on M9.

This milestone introduces GitHub OAuth, repository connection settings, and the manual "fetch PR" action. It does not introduce automated webhooks.

No M10 work begins without a written decision record in `decisions.md`.

---

## Milestone 11 — Automated Synchronization and Webhooks

[FUTURE] — Not planned until M10 is complete.

**Goal:** Automatically ingest PRs from connected repositories via GitHub webhooks, removing the need for users to manually trigger synchronization.

**Status:** [FUTURE] — Blocked on M10.

No M11 work begins without a written decision record in `decisions.md`.

---

## Milestone 12 — Persistence and Auditability

**Goal:** Ensure every analysis is fully auditable.

### Work items

Each saved analysis record must include:

- [ ] Analysis input (requirement sources, diff content — storage and redaction policy to be decided before this milestone)
- [ ] Analysis output (all structured findings)
- [ ] AI model identifier used
- [ ] Prompt or workflow version used
- [ ] Review status and reviewer identity
- [ ] Timestamps (created, submitted, reviewed)
- [ ] Evidence per finding
- [ ] User edits and override notes

### Acceptance criteria

```
Given an analysis was run six months ago
When an admin views the analysis record
Then the original inputs, outputs, model, and reviewer are visible
And the record has not been modified by any subsequent migration
```

### Regression requirement
Run full regression smoke test of all existing workflows before merging.

---

## Milestone 13 — Controlled Team Pilot

**Goal:** Enable Change Intelligence for a small internal group and measure outcomes.

### Work items

- [ ] Enable feature flag for a designated pilot group
- [ ] Define and instrument pilot metrics:
  - Time saved per PR review (survey)
  - Generated test acceptance rate
  - Automation proposal acceptance rate
  - False-positive finding rate
  - Missed requirement rate
  - User trust score
- [ ] Weekly review of pilot findings
- [ ] Bug fixing based on pilot feedback
- [ ] Document findings that inform Milestone 14 decisions

### Acceptance criteria

```
Given the pilot is running
When a non-pilot user accesses QA Center
Then no Change Intelligence functionality is visible or accessible
```

```
Given the pilot runs for an agreed period
When the period ends
Then a written summary of findings is produced and reviewed
```

### Regression requirement
Maintain regression coverage throughout the pilot period.

---

## Milestone 14 — Expansion Decisions

**Goal:** Based on pilot data, decide which capabilities to build next.

Candidates (all deferred until pilot data exists — none are commitments):

| Candidate | Depends On |
|-----------|-----------|
| GitHub API integration — auto-fetch PR diffs (M9–M11 path) | Pilot demand |
| Jira auto-linking — auto-fetch story from PR branch name | Pilot demand |
| MCP tools (analyze_change, get_change_analysis, etc.) | Pilot demand |
| Development Gate integration (surface findings in Gate 3/4) | Pilot demand |
| Release Report aggregation (change-level findings in release reports) | Pilot demand |
| Repository-aware automation generation (read test file structure) | Pilot demand |

No Milestone 14 work begins without a written decision record in `decisions.md`.

---

## Regression Checklist (All Milestones)

Every implementation milestone must verify the following before merging:

**Authentication**
- [ ] Auth0 login continues to work
- [ ] Development auth (if any) continues to work
- [ ] Session expiry and renewal behave normally

**Navigation and routing**
- [ ] All existing routes return the correct pages
- [ ] No 404s introduced on existing routes

**Test Cases**
- [ ] Create, read, update, delete test cases
- [ ] Bulk operations
- [ ] Tagging and prioritization

**Test Suites**
- [ ] Create, clone, manage suites
- [ ] Add/remove cases from suites

**Test Runs**
- [ ] Start a test run
- [ ] Record step-level results
- [ ] Add notes and attachments

**My Work**
- [ ] Personal work queue loads and updates correctly

**Releases**
- [ ] Release list and detail pages load
- [ ] Date fields editable

**Release Quality Reports**
- [ ] Report generation completes successfully
- [ ] Public report links remain accessible

**Development Gates**
- [ ] All six gate phases display correctly
- [ ] Gate status updates work

**AI Studio**
- [ ] PRD submission works
- [ ] Test case generation completes
- [ ] Review, approve, reject, and import workflow works

**MCP Server**
- [ ] Existing MCP test-case tools respond correctly
- [ ] Existing MCP test-run tools respond correctly
- [ ] Existing MCP PRD workflow tools respond correctly

**Settings**
- [ ] Project settings
- [ ] User management
- [ ] Jira integration settings
- [ ] Anthropic integration settings
- [ ] MCP API key management

**Feature flag — disabled state**
- [ ] No Change Intelligence navigation item visible
- [ ] No Change Intelligence routes accessible
- [ ] All above items pass in the disabled state
