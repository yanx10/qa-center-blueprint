# Decisions — Change Intelligence

Product and design decisions specific to this feature. Engineering-level architecture decisions should also be recorded as ADRs in `architecture/adr/` and cross-referenced here.

---

## Approved Decisions

These decisions are adopted. They are not open for reconsideration without a documented rationale and explicit approval.

---

### D-001 — Change Intelligence is additive and must not replace existing QA Center modules

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** Change Intelligence is implemented as an extension of the existing QA Center platform. No existing feature, route, table, column, or API contract is removed, renamed, or changed in a breaking way as part of this feature.

**Rationale:** QA Center is a live brownfield system with active users. Additive-only changes protect existing workflows and minimize risk during rollout.

**Consequences:** Every implementation decision must first ask "does this preserve existing behavior?" before asking "does this enable the new behavior?"

---

### D-002 — AI Studio remains fully supported

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** AI Studio continues to operate as an independent, unmodified feature. Change Intelligence does not modify, replace, or deprecate AI Studio functionality.

**Rationale:** AI Studio is in active use. Teams rely on the PRD-to-test-case workflow daily. Disrupting it would be unacceptable.

**Consequences:** Any reuse of AI Studio components must be non-destructive. Because reuse of the AI Studio review page is not feasible without modification (see D-009), a compatible parallel implementation is used instead.

---

### D-003 — Existing test-case and test-suite models must be reused

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** Test cases and test suites generated or approved through Change Intelligence are stored in the existing `test_cases` and `test_suites` tables. A parallel test-management system is not created.

**Rationale:** Creating a parallel model would fragment the test library and break workflows that depend on the unified test-case library (runs, suites, MCP tools, reports).

**Consequences:** `change_generated_test_cases` is a staging table only. Approved cases are imported into `test_cases` via an explicit import action. The import path mirrors the AI Studio accept flow (confirmed in Milestone 0).

---

### D-004 — Phase 1 begins with manual PR diff and requirement text input

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** Phase 1 does not require GitHub API integration. Users paste or manually supply a PR diff and requirement text.

**Rationale:** GitHub API integration adds credential management, OAuth flow, and rate-limit handling complexity. The core value can be demonstrated and validated without it.

**Consequences:** The input UX for Phase 1 uses text areas. The data model stores raw diff text without assuming a structured GitHub PR object.

---

### D-005 — Playwright is the first and only automation proposal format in Phase 1

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** Automation proposals in Phase 1 target Playwright (TypeScript). Other frameworks are not proposed in Phase 1.

**Rationale:** Playwright is the dominant modern automation framework for web applications. Narrowing to one format reduces AI prompt complexity and focuses the evaluation surface.

**Consequences:** Proposals are clearly labeled "Playwright (TypeScript)". Users using other frameworks must adapt the proposals manually.

---

### D-006 — Human approval is required before any generated case enters an existing suite

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** No AI-generated test case from Change Intelligence may be added to an active test suite without explicit human approval on a per-case basis.

**Rationale:** Automated insertion of unreviewed cases into production suites would corrupt active test runs and damage trust in the test library.

**Consequences:** A review interface (approve / reject per case) is required before any import mechanism is implemented. Import sets `review_status = 'approved'` in `change_generated_test_cases` and creates a corresponding row in `test_cases`.

---

### D-007 — The feature must support controlled rollout

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** Change Intelligence is gated behind a feature flag. When the flag is off, no user-visible change occurs. See D-012 for the specific flag mechanism.

**Rationale:** A brownfield feature with active users cannot be released without a kill switch.

**Consequences:** The flag must be checked independently at every page and API route. Navigation state alone is not a sufficient security control.

---

### D-008 — Change Intelligence uses separate persistence tables; AI Studio tables are not reused

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** Change Intelligence introduces its own tables (`change_analyses`, `change_analysis_inputs`, `change_requirements`, `change_risk_findings`, `change_generated_test_cases`, `change_playwright_proposals`). The AI Studio tables `prd_documents`, `ai_generation_sessions`, `ai_generated_test_cases`, and `prd_gaps` are not extended or reused.

**Rationale:** AI Studio is PRD-centric. Its schema stores PRD documents, generation sessions, and generated cases in a structure that is tightly coupled to the PRD-to-test-case flow. Change Intelligence is diff-centric: it stores requirement snapshots, diff inputs, risk findings, and structured coverage mappings. These shapes are incompatible. Reuse would create coupling that makes both features harder to evolve independently. Confirmed by Milestone 0 schema inspection.

**Consequences:** Change Intelligence has its own complete data model. The only shared tables are the permanent domain tables: `users`, `projects`, `releases`, `test_cases`, and `test_suites`.

---

### D-009 — Change Intelligence uses a separate review interface

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** Change Intelligence builds its own review UI under `app/change-intelligence/_components/` and `components/change-intelligence/`. It does not reuse or modify the AI Studio review page (`app/ai-studio/page.tsx`).

**Rationale:** The AI Studio review interface is a 1,353-line monolithic `'use client'` component. The review UI is fully embedded with PRD-specific state and side effects and cannot be extracted without modifying the AI Studio page. Modification is prohibited by D-002. Confirmed by Milestone 0 source inspection.

**Consequences:** Change Intelligence implements its own case review flow. Future refactoring that extracts a shared review pattern from both flows is a separate, deferred decision.

---

### D-010 — Migration file: `030_change_intelligence.sql`

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** The first Change Intelligence migration file is named `030_change_intelligence.sql`. The file number `020` must not be used.

**Rationale:** Migration files exist for 010, 011, 012, and 019. The schema.sql contains inline DO blocks equivalent to migrations through 029. Using `020` would create a numeric conflict with existing inline schema content. Confirmed by Milestone 0 inspection.

**Consequences:** Any blueprint reference to `020_change_intelligence.sql` is incorrect and must be updated to `030_change_intelligence.sql`. This migration is not introduced until Milestone 2.

---

### D-011 — `created_by` uses UUID foreign keys

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** All Change Intelligence tables use `created_by UUID REFERENCES users(id) ON DELETE SET NULL` for user attribution. Where review attribution is required, `reviewed_by UUID REFERENCES users(id) ON DELETE SET NULL` follows the same pattern. Foreign keys are nullable to support migration and system-generated records.

**Rationale:** The Auth.js v5 session loads `users.id` (a UUID) into the JWT. Using a UUID foreign key provides relational integrity, enables future authorization queries, and supports durable auditability. Older tables in the application use `created_by TEXT` — this was an early implementation shortcut. Change Intelligence follows the more correct approach used by `import_sessions` and `qa_reports`. Confirmed by Milestone 0.

**Consequences:** All new Change Intelligence tables diverge from the AI Studio `created_by TEXT` pattern. This is intentional. No existing table is altered.

---

### D-012 — Feature flag: server-side environment variable `CHANGE_INTELLIGENCE_ENABLED`

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** The Milestone 1 feature flag is a server-side environment variable named `CHANGE_INTELLIGENCE_ENABLED`. A public build-time variable (`NEXT_PUBLIC_CHANGE_INTELLIGENCE_ENABLED`) is not used. The root server layout reads the flag and passes an explicit boolean prop to `SideNav`.

**Rationale:** No feature flag mechanism existed in the application before this feature. An environment variable is the simplest implementation that requires no migration, no DB query on every request, and no client-side exposure of operational state. The navigation flag does not need to be a public variable because the server-side layout can resolve it at request time and pass it as a prop.

**Consequences:**
- `lib/features.ts` exports `isChangeIntelligenceEnabled(): boolean`.
- `env.ts` adds `CHANGE_INTELLIGENCE_ENABLED: z.string().optional()` to the server schema.
- `app/layout.tsx` calls `isChangeIntelligenceEnabled()` and passes the result to `SideNav`.
- Every page and API route independently checks the flag — the navigation state is not a security control.
- For the Milestone 9 pilot, a database-backed per-user control may be added as a secondary layer.

---

### D-013 — Route `/change-intelligence`; API namespace `/api/change-intelligence/`; UI label "PR Intelligence"

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** The application route is `/change-intelligence`. The API namespace is `/api/change-intelligence/`. UI copy may use "PR Intelligence" where helpful to users, while "Change Intelligence" remains the domain and architecture name.

**Rationale:** `/change-intelligence` is simpler and aligns directly with the domain name. The distinction between UI label and domain name allows the product label to evolve without requiring route or schema changes.

**Consequences:** All routes, API paths, and component namespaces use `change-intelligence`. Any blueprint reference to `/pr-intelligence` as a route is incorrect.

---

### D-014 — Every Change Intelligence API route must call `requireAuth()` explicitly

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** Every Change Intelligence API route handler calls `requireAuth()` at the top of the handler, before any feature-flag check, before any database access, and before any AI call.

**Rationale:** The Next.js middleware (`proxy.ts`) passes all `/api/*` requests through without any authentication check. API routes are not automatically protected. This is confirmed behavior in the existing application. Change Intelligence must not repeat the security gap found in the AI Studio accept route, which does not call `requireAuth()`.

**Consequences:** The established pattern is:
```
const { ctx, errorResponse } = await requireAuth();
if (errorResponse) return errorResponse;
```
The feature-flag check comes after auth, before business logic.

---

### D-015 — Standard API response envelope

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** All Change Intelligence API routes use the existing application response envelope:

Success: `{ "success": true, "data": {} }`
Failure: `{ "success": false, "error": "Human-readable error" }`

**Rationale:** This is the confirmed response shape used throughout the existing application. Consistency is required. The previous blueprint proposal of bare `{ data, error }` was incorrect and is superseded by this decision.

**Consequences:** All API response examples in `api-design.md` must use this envelope. The feature-flag disabled response is `{ "success": false, "error": "Change Intelligence is not enabled." }` with status 403 — not the previously proposed `{ "error": "CHANGE_INTELLIGENCE_DISABLED", "message": "..." }` shape.

---

### D-016 — Phase 1 uses the existing synchronous Anthropic integration pattern

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** Change Intelligence uses `generateStructuredAIResponse<T>()` from `lib/ai/anthropic.ts` without modification. The existing synchronous pattern — one blocking HTTP request per AI call, no streaming, no automatic retry — is retained for all Milestone 1 through Milestone 8 work.

**Rationale:** No streaming or retry framework currently exists in the application. Introducing one is out of scope for Phase 1. The existing synchronous pattern has proven workable for AI Studio's PRD generation (15–40 second calls) and is sufficient for initial Change Intelligence calls.

**Consequences:** Before Milestone 2 is considered complete, the following must be defined in the blueprint:
- Maximum accepted diff size and maximum accepted requirement size
- Token estimation approach
- Safe truncation or chunking behavior for oversized inputs
- Request timeout limits
- Partial failure behavior
- Structured-response validation

Streaming, queues, and background jobs are deferred to Phase 2.

---

### D-017 — Full PR diff storage is temporarily permitted

**Status:** [DECISION]
**Date:** 2026-07-26

**Decision:** For the early internal implementation, full PR diff content is stored in `change_analysis_inputs.content TEXT`. This is not the final data retention policy.

**Rationale:** Simplest implementation for an internal-only, pre-pilot feature. Redaction and retention complexity is deferred until the data policy is aligned.

**Consequences:** Before Milestone 8 or any broader pilot, the team must approve:
- Data retention duration
- Redaction rules
- Repository sensitivity handling
- Deletion behavior
- Access restrictions
- Audit requirements

---

## Resolved Open Decisions

The following open decisions were recorded before Milestone 0. All have been resolved by Milestone 0 discovery or by explicit approval. They are recorded here for traceability.

---

### OD-001 — Route name — RESOLVED

**Resolution:** `/change-intelligence`. See D-013.

---

### OD-002 — Whether to reuse AI Studio generation-session tables — RESOLVED

**Resolution:** Do not reuse. Separate Change Intelligence tables are required. See D-008.

---

### OD-003 — Whether generated cases appear in the existing AI Studio review interface — RESOLVED

**Resolution:** No. A separate Change Intelligence review interface is built. See D-009.

---

### OD-004 — Feature flag implementation — RESOLVED

**Resolution:** Server-side environment variable `CHANGE_INTELLIGENCE_ENABLED` for Milestone 1. Database-backed per-user control may be added for Milestone 9 pilot. See D-012.

---

### OD-005 — Initial access control — RESOLVED

**Resolution:** All authenticated users in the organization can access Change Intelligence when the flag is enabled. Access matches existing application behavior (org-wide RBAC: admin/member/viewer). The `created_by` field provides audit trail but does not gate access. Revisit for Milestone 9 pilot if per-user access control is needed.

---

### OD-006 — Whether PRD documents should be referenced or copied — RESOLVED

**Resolution:** Snapshot PRD content at analysis time into `change_analysis_inputs.prd_snapshot TEXT`. The `prd_documents.md_content` can change after an analysis is run; a snapshot ensures the analysis record remains self-contained and auditable. A `source_reference` column stores the PRD document ID as a reference.

---

### OD-007 — Whether diff input should be permanently stored, partially stored, or redacted — RESOLVED

**Resolution:** Full diff stored in Phase 1. See D-017. Retention policy deferred.

---

## Deferred Decisions

These decisions are explicitly deferred. They must not be made by assumption. Each requires a documented decision record before any implementation proceeds.

---

### OD-008 — GitHub authentication and credential ownership

**Status:** [OPEN] — Deferred to Phase 2 / Milestone 10

**Context:** GitHub API integration requires OAuth tokens or personal access tokens. Whether these are org-level or user-level affects credential management, token rotation, and access control significantly.

**Blocked by:** Milestone 10 expansion decision. GitHub integration is not in Phase 1 scope.

---

### OD-009 — Final diff retention policy

**Status:** [OPEN] — Deferred to pre-Milestone 8

**Context:** D-017 permits full storage temporarily. Before any broader pilot, the team must agree on duration, redaction, deletion, and audit requirements.

---

### OD-010 — Large-diff chunking algorithm

**Status:** [OPEN] — Deferred to pre-Milestone 2 completion

**Context:** PR diffs can be arbitrarily large. A `MAX_DIFF_TOKENS` truncation or chunking strategy must be defined before Milestone 2 is considered production-ready.

---

### OD-011 — Future queue or streaming architecture

**Status:** [OPEN] — Deferred to Phase 2

**Context:** D-016 retains the synchronous pattern for Phase 1. If 15–40 second AI calls prove problematic in production, a background job or streaming approach may be needed. This requires explicit scoping and tooling decisions.

---

### OD-012 — Per-user pilot flag implementation (Milestone 9)

**Status:** [OPEN] — Deferred to Milestone 9

**Context:** D-012 defers per-user control to Milestone 9. The specific implementation (pilot users table, JSON in `integration_settings`, or role-based gate) must be decided before Milestone 9 begins.

---

### OD-013 — Release Quality Report integration

**Status:** [OPEN] — Deferred to Phase 2 / Milestone 10

**Context:** Change Intelligence findings may eventually surface in release-level reporting. Phase 1 does not modify Release Quality Reports in any way.

---

### OD-014 — Development Gate integration

**Status:** [OPEN] — Deferred to Phase 2 / Milestone 10

**Context:** Change Intelligence risk findings could inform Development Gate status. Phase 1 does not touch Development Gates.

---

### OD-015 — MCP tool exposure

**Status:** [OPEN] — Deferred to Phase 2 / Milestone 10

**Context:** The existing MCP server has 17 tools. Change Intelligence MCP tools (`analyze_change`, `get_change_analysis`, etc.) are deferred to Milestone 10 based on pilot demand.

---

## Decision Template

Use this template for new decisions:

```
### D-NNN — [Title]

**Status:** [DECISION | PROPOSAL | OPEN]
**Date:** YYYY-MM-DD

**Decision:** [One-sentence statement of what was decided.]

**Rationale:** [Why this option over the alternatives.]

**Consequences:** [What this decision requires, prevents, or changes.]

**Related ADR:** [Link to ADR in architecture/adr/ if applicable.]
```
