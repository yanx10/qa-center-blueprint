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

### D-018 — Analysis lifecycle starts in draft and locks inputs at ready

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** A `change_analyses` record is created with `status = 'draft'`. Inputs can be added to and removed from a draft analysis. When the user promotes the analysis to `status = 'ready'`, inputs are locked — no further additions or deletions are permitted. The AI pipeline (M3) runs only on `ready` analyses.

**Rationale:** The draft/ready split gives users the ability to build up inputs across multiple sessions without accidentally triggering an AI call. Locking inputs at ready ensures the AI sees a stable, consistent set of data and provides a clear moment of user intent before any AI processing begins.

**Consequences:** All six M2 input management API routes enforce the draft/ready boundary. PATCH to `ready` requires at least one `pr_diff` input. Attempts to add or remove inputs from a non-draft analysis return 409. M3 must transition status from `ready` to `processing` atomically before beginning any AI call.

---

### D-019 — Inputs are immutable once the parent analysis leaves draft

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** `change_analysis_inputs` records can only be created or deleted when their parent `change_analyses` record is in `draft` status. Any attempt to add or remove an input from an analysis in `ready`, `processing`, `completed`, `failed`, or `cancelled` status is rejected with 409.

**Rationale:** Immutability after the ready transition ensures that the AI analysis runs against exactly the inputs the user reviewed and approved. Allowing post-ready modifications would create a mismatch between what the user confirmed and what the AI processed, undermining auditability.

**Consequences:** If a user needs to change inputs after promoting to ready, they must cancel the analysis and create a new one. A retry creates a new record (no in-place modification). See D-021.

---

### D-020 — Input content is excluded from all GET API responses

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** The `content` and `prd_snapshot` columns of `change_analysis_inputs` are never returned by any GET API endpoint. Responses include only `id`, `input_type`, `content_hash`, `source_label`, `source_reference`, and `created_at`. Content is stored in the database but is only accessed by the M3 AI pipeline.

**Rationale:** Input content (PR diffs, requirement text) can be large (up to 500,000 characters) and may contain sensitive code or unreleased product information. Excluding content from GET responses reduces bandwidth consumption, prevents accidental exposure in browser developer tools, and clarifies the API contract: content flows in (write), content hash flows out (read).

**Consequences:** API clients cannot retrieve the stored content via GET. If a client needs to verify the stored content, they compare `content_hash` against their locally computed hash. The M3 pipeline reads `content` directly from the database — it does not go through the GET API.

---

### D-021 — A failed or cancelled analysis cannot be retried in place; a new record must be created

**Status:** [SUPERSEDED by D-032 for M3 failed analyses; remains in force for cancelled analyses]
**Date:** 2026-07-27

**Decision:** There is no in-place retry mechanism for `failed` or `cancelled` analyses. The user must create a new `change_analyses` record and attach inputs again. The old record is preserved in its terminal state for audit purposes.

**Rationale:** Allowing in-place retry would require the status machine to permit transitions from `failed` or `cancelled` back to `ready`, which creates complex state and auditability problems. Creating a new record is simple, auditable, and consistent with the immutability principle of D-019.

**Consequences:** The UI should make it easy to copy inputs from a failed or cancelled analysis into a new analysis (M3/UX scope). The old record remains queryable. The `created_by` on the new record will be the authenticated user at the time of retry.

**Note for M3+:** D-032 supersedes this decision for `failed` analyses in M3 — in-place retry is introduced for analyses that fail during AI processing. Cancelled analyses still require a new record.

---

### D-022 — Input content is limited to 500,000 characters

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** The `content` field of `change_analysis_inputs` is limited to 500,000 characters. The API rejects any input with `content` exceeding this limit with a 400 response before any database write occurs.

**Rationale:** 500,000 characters is approximately 125,000 tokens — well within Anthropic's context limits while preventing pathological inputs that could cause excessive latency or cost. The limit provides a clear, enforceable contract for both the API and the AI pipeline. See also OD-010 (large-diff chunking algorithm, deferred).

**Consequences:** Users with diffs or documents exceeding 500,000 characters must manually trim or summarize the content before submission. The limit is enforced at the API layer only — the database `TEXT` type has no inherent limit. A clear error message must be returned indicating the limit.

---

### D-023 — Analysis title is required in persisted records; auto-generated if not provided

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** `change_analyses.title` is a NOT NULL column with no database default. If the caller does not supply a title in `POST /api/change-intelligence/analyses`, the application generates one before inserting: `"Change Analysis — [Month Day, Year H:MM AM/PM]"`. The timestamp is the server-side time at the moment of creation. `PATCH` cannot set title to null or empty string.

**Rationale:** Requiring a non-null title simplifies the data model (no null-handling in display code) and gives every analysis a human-readable label without burdening the caller. Auto-generation is invisible to users who don't care about titles and useful to those who do.

**Consequences:** The POST handler must generate a title before INSERT if the caller omits one. The schema has a NOT NULL constraint on title with no default, so any INSERT without a title fails at the database layer. A future improvement may derive a context-aware title from input content [FUTURE]; this is out of scope for M2. Resolves OQ-M2-01.

---

### D-024 — Analysis visibility is based on project access, not creator identity

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** `GET /api/change-intelligence/analyses` returns all analyses the authenticated user is authorized to view through project access. In M2, with no project-level access control in the existing application, this means all authenticated users can view all analyses. `created_by` is an optional convenience filter parameter, not an authorization boundary.

**Rationale:** Creator-restricted visibility would prevent QA leads from reviewing team analyses and would not match the existing application's org-wide access model (OD-005). Project-based authorization is the correct long-term model and is naturally org-wide in M2 since project-level access control does not yet exist.

**Consequences:** Any authenticated user can call `GET /analyses?created_by=<any-uuid>`. The UI may present "My analyses" and "All analyses" views as convenience filters using the same endpoint. If project-level access control is added in a future milestone, it gates which project-associated analyses a user sees. Resolves OQ-M2-02.

---

### D-025 — Offset pagination with 25 items per page and deterministic secondary sort

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** The list endpoint uses offset-based pagination. Default page size: 25. Maximum page size: 100. Default sort: `updated_at DESC, id DESC`. The secondary `id DESC` sort is required to ensure deterministic ordering when two analyses have the same `updated_at` timestamp. Cursor-based pagination is deferred to M12 [FUTURE].

**Rationale:** Offset pagination is sufficient at pilot scale (expected hundreds of analyses, not millions). The secondary sort prevents non-deterministic page boundaries, which cause duplicate or missing items when a user pages through the list. 25 items matches the expected analysis density per sprint; 100 is a generous maximum for bulk export use cases.

**Consequences:** Large analysis histories (thousands of records) will experience the O(offset) performance degradation inherent to offset pagination. This is acceptable for M2 pilot scale and is addressed in M12. Resolves OQ-M2-03.

---

### D-026 — At most one pr_diff input per analysis; replace-on-update behavior for drafts

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** Each analysis may have at most one input with `input_type = 'pr_diff'`. This is enforced at two layers: (1) a partial unique index in the database: `UNIQUE (analysis_id) WHERE input_type = 'pr_diff'`; (2) the service layer, which replaces an existing pr_diff on a draft analysis when `POST /inputs` is called with `input_type = 'pr_diff'`. Replacement is a DELETE + INSERT within a single transaction. No uniqueness constraint applies to any other input type.

**Rationale:** A Change Intelligence analysis conceptually represents the review of one PR diff. Allowing multiple pr_diff inputs would create ambiguity about which diff was analyzed and complicate the AI prompt construction in M3. Replace-on-update (rather than reject-on-duplicate) makes the API ergonomic — the caller can freely correct the diff without first deleting the old one.

**Consequences:** `POST /inputs` with `input_type = 'pr_diff'` returns 201 when creating the first pr_diff and 200 when replacing an existing one. If the partial unique index fires due to a concurrent race (two requests both attempted insert after checking for an existing row), the service layer must catch PostgreSQL error code `23505` on constraint `uq_change_analysis_inputs_pr_diff` and return 409 Conflict — not 500. The constraint name, SQL, and driver details must not appear in the error response. Any other database error continues to return 500. Resolves OQ-M2-04.

---

### D-027 — Content hash: SHA-256, lowercase hex, with CRLF normalization

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** The `content_hash` field stores the SHA-256 hash of input content, encoded as 64 lowercase hexadecimal characters, with no algorithm prefix. Canonicalization before hashing: (1) remove optional UTF-8 BOM (EF BB BF); (2) normalize CRLF (`\r\n`) to LF (`\n`); (3) preserve all other whitespace; (4) encode as UTF-8; (5) compute SHA-256; (6) store as 64 lowercase hex characters. The hash is computed server-side before INSERT. Callers must not supply `content_hash`; any caller-supplied value is ignored.

**Rationale:** Lowercase hex is the most portable representation and matches common SHA-256 output from cryptographic libraries. CRLF normalization ensures the hash is stable across operating systems — a diff pasted from a Windows machine should hash identically to the same diff on Unix. No algorithm prefix is stored because SHA-256 is the only algorithm used in this system. The purpose is deterministic comparison and idempotency checking, not a cryptographic security guarantee.

**Consequences:** The hash comparison is valid for idempotency checking only when both sides apply the same canonicalization pipeline. Any change in the canonicalization rules requires a rehash of all existing inputs — a breaking change requiring a data migration. Resolves OQ-M2-05.

---

## Milestone 3 Decisions

### D-028 — AI output stored as JSONB; downstream analysis tables are M4+ scope

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** M3 stores the complete AI output as a single JSONB object in `change_analyses.analysis_json`. The tables `change_requirements`, `change_risk_findings`, `change_generated_test_cases`, and `change_playwright_proposals` are deferred to M4+ milestones and are NOT created in M3.

**Rationale:** The AI output schema will evolve as prompts are refined. Creating normalized rows now would require migrations every time the prompt changes what is extracted. Keeping findings as conceptual entities in JSONB until the schema stabilizes reduces churn. M4 will extract individual requirements into rows when per-requirement reviewer interaction is needed.

**Consequences:** M3 never writes to `change_requirements` or any of the other downstream tables. The analysis_json JSONB field is the single source of truth for all AI output in M3.

---

### D-029 — M3 AI processing is synchronous (one HTTP round-trip)

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** `POST /generate` waits for the Anthropic API response and returns the result in the same HTTP response. No background jobs, message queues, or polling in M3. The server timeout is configured for 120 seconds.

**Rationale:** The existing `generateStructuredAIResponse<T>()` function is synchronous (D-016). Building async infrastructure in M3 would require significant new tooling. Expected response times of 15–60 seconds are acceptable for the internal pilot.

**Consequences:** Async processing is a [FUTURE] candidate if M3 latency proves unacceptable in broader use. The client must display a processing state while awaiting the synchronous response.

---

### D-030 — Findings are conceptual entities in M3, not database rows

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** Individual AI findings (risk items, coverage gaps, edge cases, recommendations) exist only within `analysis_json` in M3. They are not materialized as separate database rows.

**Rationale:** Aligns with D-028. The finding schema will change as the QA Intelligence Report prompt matures. Materializing rows now creates migration and compatibility obligations before the schema is stable. Findings become independently actionable (and deserve DB rows) when M4 introduces per-finding reviewer interaction.

**Consequences:** The QA Intelligence Report UI in M3 reads directly from `analysis_json`. No cross-finding queries or individual finding endpoints are possible in M3.

---

### D-031 — Five denormalized projection columns are populated from analysis_json on completion

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** After a successful AI analysis, four columns on `change_analyses` are populated as denormalized projections from `analysis_json`:

| Column | Source in analysis_json |
|--------|------------------------|
| `change_summary` | `executive_summary.headline` |
| `requirement_summary` | `executive_summary.primary_concern` |
| `risk_level` | `executive_summary.risk_level` |
| `analysis_schema_version` | `schema_version` |
| `change_type_summary` | `change_summary.change_type` (array joined as comma-separated string) |

All five are written atomically with `analysis_json` in the `persistSuccess()` transaction. None are recomputed on subsequent reads.

**Rationale:** These fields are needed in the analysis list view. Including them as top-level columns avoids loading `analysis_json` (which can be up to ~5,000 tokens) in list queries. `risk_level` enables visual risk-level badges on the list. `analysis_schema_version` enables efficient filtering when the AI output schema evolves across milestones. `change_type_summary` enables list-level change-type display.

**Consequences:** If the prompt evolves such that `executive_summary.headline`, `executive_summary.risk_level`, `schema_version`, or `change_summary.change_type` change their structure, the denormalization logic must be updated. A prompt version bump is required in that case (see D-035). The `persistSuccess()` implementation must include all five fields in a single `UPDATE` statement.

---

### D-032 — Retry updates the existing analysis record in-place

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** `POST /retry` does not create a new analysis record. It reuses the existing `analysis_id`, clears `error_code`, `error_message`, and `analysis_json`, increments `retry_count`, and re-runs AI processing. Maximum 3 retries total.

**Rationale:** The user's intent is to re-run the same analysis, not to create a duplicate. Keeping the same ID preserves URL stability and simplifies UI state management. The `retry_count` cap prevents runaway retries in case of a persistent provider error.

**Consequences:** Supersedes D-021 for M3 and later milestones. D-021 applied to M2 where no retry mechanism existed. In M3, in-place retry is the correct behavior because the analysis already has its inputs locked in `ready` state and there is no need to re-enter inputs.

---

### D-033 — Cancellation is supported from ready status only in M3

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** `POST /cancel` transitions `ready` → `cancelled`. Cancelling a `processing` analysis is not supported in M3 and returns 409.

**Rationale:** M3 uses synchronous HTTP processing. Once the AI call is in flight, there is no mechanism to interrupt it. The 120-second server timeout is the only circuit breaker. A future async model would enable mid-flight cancellation.

**Consequences:** The UI must not display a cancel button while the analysis is in the `processing` state in M3. This limitation must be communicated to users.

---

### D-034 — AI temperature fixed at 0.2 for all M3 analyses

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** The temperature parameter is hardcoded to 0.2 and is not user-configurable in M3.

**Rationale:** QA analysis benefits from low temperature (high determinism). Temperature selection requires empirical calibration and UI surface area that is out of scope for M3.

**Consequences:** If temperature configurability is needed, it should be addressed as a separate feature in M12 or later. The value 0.2 is stored in `change_analyses.temperature` on every completed analysis for auditability.

---

### D-035 — Prompt versioning format is "m{milestone}-v{sequence}"; M3 initial version is "m3-v1"

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** The prompt version identifier uses the format `"m{milestone}-v{sequence}"`. M3's initial prompt is `"m3-v1"`. The version is stored in `change_analyses.analysis_version` for every analysis. Any change to the prompt that could alter output structure or content requires incrementing the sequence (m3-v2, m3-v3, etc.).

**Rationale:** Prompt changes affect output quality and schema. Recording the version on every analysis enables quality correlation, debugging, and safe prompt evolution without invalidating existing completed analyses.

**Consequences:** A prompt change in M3 that is backwards-compatible (wording improvement, no structural change) may use a sub-version such as `"m3-v1.1"` — [OPEN] this sub-version convention is not yet decided and must be resolved before M4.

---

### D-036 — POST /generate and POST /retry return HTTP 200 for both completed and failed AI outcomes

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** Both `/generate` and `/retry` always return HTTP 200 when the request completes (whether AI succeeded or failed). The caller reads `data.status` to determine whether the AI succeeded (`status: completed`) or failed (`status: failed`). HTTP 4xx is reserved for precondition failures (wrong status, max retries). HTTP 5xx is reserved for unexpected server errors.

**Rationale:** An AI failure is an expected outcome, not a server error. Using HTTP 200 for expected outcomes (even negative ones) keeps the HTTP status semantics clean and simplifies client error handling. A 200 with `status: failed` is explicitly different from a 500 with an unexpected server error.

**Consequences:** Clients must not assume a 200 response means the analysis completed successfully — they must read `data.status`. This pattern is documented in the API contracts and the UI handles both outcomes from a single response.

---

### D-038 — M3 migration is registered as `033_change_intelligence`; `032` prefix is reserved

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** The M3 AI Change Analysis migration is named `033_change_intelligence` (both the registry entry and the `db/migrations/033_change_intelligence.sql` file). The `032` number prefix is already occupied by `032_change_intelligence_pr_description` (M2 UX polish, added the `pr_description` input type). Reusing the `032_` prefix for a second file would create migration ordering ambiguity.

**Rationale:** The application's migration runner (registry-based, not file-scan-based) uses insertion-order from the `MIGRATIONS` array as the execution order. Having two entries with the same numeric prefix (`032_change_intelligence` and `032_change_intelligence_pr_description`) would make the ordering non-obvious and could cause confusion when reading the registry. Sequential numeric prefixes maintain clarity.

**Consequences:** Any blueprint reference to `032_change_intelligence.sql` as the M3 migration file is incorrect. The M3 migration file is `033_change_intelligence.sql` and must be registered as `'033_change_intelligence'` in the `MIGRATIONS` array in `app/api/admin/migrate/route.ts` after the `032_change_intelligence_pr_description` entry.

---

### D-039 — `supplemental_context` is a valid `input_type`; `prd_text` is the canonical PRD input type

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** The canonical set of allowed `input_type` values for `change_analysis_inputs` is:
`pr_diff`, `pr_description`, `requirement_text`, `jira_story`, `acceptance_criteria`, `prd_text`, `markdown_spec`, `supplemental_context`.

The value `prd_text` is correct (not `prd_document`). The value `supplemental_context` is a valid input type referenced in the M3 prompt builder (OQ-M3-P prompt ordering) and M3-AC-25 — it must be added to the `chk_change_analysis_inputs_type` check constraint. A M2.x or M3 migration must drop and recreate this constraint to include both `supplemental_context` (if not already present from a future M2 polish migration) and any other new values. The M3 `033_change_intelligence.sql` migration must include this constraint update if `supplemental_context` is not already present.

**Rationale:** The M2 migration `031_change_intelligence` created the constraint without `supplemental_context`. The M3 prompt builder references it as a recognized input ordering category. Including it in the constraint ensures DB-level validation matches the prompt builder's expectations.

**Consequences:** The `033_change_intelligence.sql` migration must add `supplemental_context` to the `chk_change_analysis_inputs_type` CHECK constraint using a DROP + ADD CONSTRAINT pattern (idempotent). The `data-model.md` input_type enum list is authoritative.

---

### D-037 — Failed analysis records do not persist partial AI output; analysis_json remains null on failure

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** When AI processing fails for any reason (timeout, parse error, schema validation failure, or unexpected error), the analysis transitions to `failed` status with `error_code` and `error_message` populated. `analysis_json` remains `null`. Partial or structurally invalid AI output is not stored, even if the response contained some valid fields.

**Rationale:** Storing partial output would create an ambiguous data state where some fields are populated and others are null, with no clear indication of what can be trusted. The validation layer enforces all-or-nothing semantics — if the complete output cannot be validated against the schema, none of it is stored. A `null` `analysis_json` on a failed record is an unambiguous signal that no valid output exists for that record.

**Consequences:** If an analysis fails repeatedly due to `SCHEMA_VALIDATION_FAILED`, the issue is likely a prompt-schema incompatibility and should be investigated before further retries are attempted. The `error_code` distinguishes transient failures (`PROVIDER_TIMEOUT`, `PROVIDER_ERROR` — retry likely to succeed) from structural failures (`SCHEMA_VALIDATION_FAILED`, `INVALID_OUTPUT` — may indicate a prompt or schema issue requiring investigation). Retry (D-032) re-runs the full analysis from scratch on the original inputs.

---

### D-040 — The `change-intelligence-provider.ts` adapter is a thin integration seam and does not imply multi-provider capability in M3

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** `lib/ai/change-intelligence-provider.ts` is a thin adapter that wraps `generateStructuredAIResponse<T>()`. It provides a stable interface for the service layer to call without depending directly on the Anthropic client. It does not introduce provider selection, switching, or multi-provider routing in M3. The `provider` column on `change_analyses` records which provider was used (always `'anthropic'` in M3) for auditability. Provider switching is explicitly out of scope for M3.

**Rationale:** Introducing a provider abstraction now (even as a thin seam) reduces coupling between the route handler and the Anthropic client. It also makes future provider swaps non-breaking for the service layer. However, advertising it as "multi-provider support" would create false expectations. The adapter pattern is a clean seam, not a feature.

**Consequences:** `change-intelligence-provider.ts` exports exactly one function: a typed wrapper that calls `generateStructuredAIResponse<T>()` and returns `{ rawOutput, inputTokens, outputTokens, processingMs }`. No provider registry, no switching logic, no fallback — those belong to a future milestone (M12 or later). Multi-provider support is [FUTURE].

---

### D-041 — `schema_version: "1.0.0"` is the canonical schema version identifier for M3 AI output; server validator rejects any other value

**Status:** [DECISION]
**Date:** 2026-07-27

**Decision:** Every AI response for M3 must include `schema_version: "1.0.0"` as the first field in the output JSON. The server-side validator treats any response where this field is absent or has a value other than `"1.0.0"` as a `SCHEMA_VALIDATION_FAILED` error. The schema version is not stored in a separate column; it is part of `analysis_json` and is preserved verbatim in every completed analysis record. It is also denormalized into `analysis_schema_version` (D-031).

**Rationale:** Embedding the schema version in the output JSON creates a self-describing record. When the output schema changes in M4+, an analyst can query `analysis_json->>'schema_version'` to determine which analyses used which schema, enabling safe schema migration and version-gated UI rendering. Validating the version field first provides early failure detection before any other field is inspected.

**Consequences:** Any prompt update that changes the output schema requires a version bump (`"1.0.0"` → `"1.1.0"` for additive changes, `"2.0.0"` for breaking changes) and a corresponding update to the server-side validator. Old analyses retain their original `schema_version` within `analysis_json`. The UI may use `schema_version` in M4+ to conditionally render sections that do not exist in older outputs.

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
