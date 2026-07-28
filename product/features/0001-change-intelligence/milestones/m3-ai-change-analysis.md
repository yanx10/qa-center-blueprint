# Milestone 3 — AI Change Analysis

## Milestone Summary

| Field | Value |
|-------|-------|
| **Milestone** | M3 — AI Change Analysis |
| **Feature** | Change Intelligence (Feature 0001) |
| **Phase** | Phase 1 |
| **Depends on** | M2 — Analysis Persistence Foundation (complete) |
| **Unlocks** | M4 — Requirement Comparison |
| **Status** | Blueprint Frozen — Ready for Implementation |
| **Migration file** | `033_change_intelligence.sql` |
| **New tables** | None — adds columns to `change_analyses` |
| **New API routes** | 3 new endpoints: POST /generate, POST /retry, POST /cancel |
| **Enhanced routes** | GET /analyses/:id (returns analysis_json when completed) |

---

## Blueprint Freeze Record

| Field | Value |
|-------|-------|
| **Freeze date** | 2026-07-27 |
| **Schema version** | 1.0.0 |
| **Prompt version** | m3-v1 |
| **Acceptance criteria** | 87 (M3-AC-01 through M3-AC-87) |
| **Decisions** | D-028 through D-041 |
| **Planned migration** | 033_change_intelligence.sql |
| **Implementation status** | Not started |

The M3 blueprint is frozen for implementation. The documents listed below form the implementation contract. No change to blueprint content is permitted after this date without:

1. A documented decision amendment (D-NNN) recorded in `decisions.md`
2. Corresponding updates to affected acceptance criteria
3. A CHANGELOG entry recording the amendment
4. Explicit review before implementation continues

**Implementation contract documents:**

| Document | Purpose |
|----------|---------|
| `milestones/m3-ai-change-analysis.md` | Primary milestone specification |
| `prompt-design.md` | System prompt, user prompt template, versioning |
| `analysis-schema.md` | Canonical JSON output schema (v1.0.0) |
| `ui-design.md` | UI state specifications |
| `acceptance-criteria.md` | M3-AC-01 through M3-AC-87 |
| `decisions.md` | D-028 through D-041 |
| `data-model.md` | Schema additions for migration 033 |
| `api-design.md` | API contracts |

---

## Executive Summary

Milestone 3 introduces the AI execution engine for Change Intelligence. When a user marks an analysis ready and triggers generation, the system constructs a prompt from the persisted inputs, calls Claude, validates the structured output, and persists a complete QA Intelligence Report.

M3 does not produce test cases, automation code, or GitHub comments. It produces a single structured report that answers: what changed, what is at risk, what is missing, and where should testing focus. A QA lead can go from inputs to actionable intelligence in under two minutes.

---

## Problem Statement

Milestone 2 delivered the persistence layer: users can create analyses, attach inputs, and mark them ready. When a user views a ready analysis, there is nowhere to go — the page shows the inputs and a "Generate" button that does nothing yet.

Before M3, the application has no way to:
1. Construct a prompt from the persisted inputs.
2. Call the Anthropic API with that prompt.
3. Validate the structured output.
4. Persist the result.
5. Handle failures gracefully.
6. Allow retry when the AI call fails.

M3 closes all five gaps. After M3, the full loop from input to intelligence is complete.

---

## User Value

After M3, the primary QA Center persona — Morgan, a QA Engineer — can:

1. Create an analysis, attach a PR diff and requirement documents, and mark it ready.
2. Click "Generate QA Intelligence" and wait 15–60 seconds.
3. Read a structured report that identifies coverage gaps, risk areas, regression targets, edge cases, and missing requirements.
4. Use the report to design test scenarios without spending 30–90 minutes researching manually.

The two-minute QA Intelligence turnaround is the core value proposition of Change Intelligence. M3 delivers it.

---

## Goals

1. Implement the `POST /generate` endpoint that executes AI analysis synchronously.
2. Implement the `POST /retry` endpoint for failed analyses (max 3 retries).
3. Implement the `POST /cancel` endpoint for ready analyses.
4. Add 10 new columns to `change_analyses` via migration `033_change_intelligence.sql` (9 nullable + `retry_count` NOT NULL DEFAULT 0).
5. Implement the prompt builder that constructs system + user prompts from persisted inputs.
6. Implement the output validator that enforces the analysis schema.
7. Implement atomic persistence of the complete analysis result.
8. Render all analysis lifecycle states in the UI.
9. Render the full QA Intelligence Report for completed analyses.

---

## Non-Goals

The following are explicitly out of scope for M3. Any PR that includes this work will be rejected:

- Test case generation — M6 scope
- Playwright automation code generation — M7 scope
- GitHub API integration — M9 scope
- Jira automatic linking — M10+ scope
- Requirement comparison with per-finding human review actions — M4 scope
- Streaming AI responses — future milestone
- Multi-agent orchestration — future milestone
- Per-user pilot gating — M13 scope
- Creating `change_requirements`, `change_risk_findings`, `change_generated_test_cases`, or `change_playwright_proposals` tables
- Editable prompts or prompt editor UI
- Multi-provider support (the `provider` column is reserved, not configurable in M3)

---

## Personas

### Morgan — QA Engineer

Morgan runs analyses during active development cycles. Morgan cares about speed: the analysis must complete fast enough to be useful before a PR is merged. The report must be clear enough to act on without re-reading the diff.

### Alex — QA Lead

Alex reviews completed analyses across the team. Alex uses the risk level summary and missing requirements list to prioritize review effort and escalate gaps to engineering. Alex cares about the quality of findings more than processing speed.

Persona details: see `user-experience.md`.

---

## User Journeys

### Journey 1 — Successful Analysis

1. Morgan opens a ready analysis with a PR diff and acceptance criteria attached.
2. Morgan clicks "Generate QA Intelligence."
3. The page transitions to the processing state with an animated indicator.
4. After 20–40 seconds, the page updates to show the completed QA Intelligence Report.
5. Morgan reads the Executive Summary and sees a "Medium" risk badge.
6. Morgan reviews the Recommended Test Focus section to plan the testing sprint.
7. Morgan notes a missing requirement and creates a Jira comment to flag it.

### Journey 2 — Failed Analysis with Retry

1. Morgan triggers generation. The page shows the processing state.
2. After 120 seconds, the page shows a failed state with error code "PROVIDER_TIMEOUT."
3. Morgan reads: "The AI provider took too long to respond. Try again."
4. The page shows: "Attempt 0 of 3" and a "Retry Analysis" button.
5. Morgan clicks "Retry Analysis." The page transitions to processing state again.
6. The retry succeeds. The report is displayed.

### Journey 3 — Cancellation Before Generation

1. Morgan marks an analysis ready but realizes the PR diff is stale.
2. Morgan clicks "Cancel."
3. The analysis transitions to cancelled. Morgan can delete it or use the inputs as reference for a new analysis.

---

## Architecture Overview

### Component Map

```
User clicks "Generate"
       │
       ▼
POST /api/change-intelligence/analyses/:id/generate
       │
       ├── Precondition check: status must be 'ready' → 409 if not
       │
       ├── Status transition: ready → processing (atomic DB UPDATE)
       │
       ├── Prompt builder
       │       ├── Read all inputs from change_analysis_inputs
       │       ├── Check total character count → INPUT_TOO_LARGE if exceeded
       │       └── Assemble system prompt + user prompt with labeled input blocks
       │
       ├── AI call: generateStructuredAIResponse<T>() from lib/ai/anthropic.ts
       │       └── Temperature: 0.2, Model: resolved model chain
       │
       ├── Output parser
       │       ├── JSON parse → INVALID_OUTPUT if fails
       │       └── Schema validate → SCHEMA_VALIDATION_FAILED if fails
       │
       ├── Result persistence (atomic DB UPDATE)
       │       ├── On success: status → completed, analysis_json, all metadata columns
       │       └── On failure: status → failed, error_code, error_message
       │
       └── Return 200 with full analysis object
```

### Key Files (Implementation Guidance)

| File | Purpose |
|------|---------|
| `033_change_intelligence.sql` | New migration — 9 nullable columns + retry_count (NOT NULL DEFAULT 0) + indexes |
| `lib/ai/prompts/change-intelligence.ts` | m3-v1 system prompt, user prompt template, version constant |
| `lib/ai/change-intelligence-provider.ts` | Thin provider adapter wrapping `generateStructuredAIResponse<T>()` — returns `{ rawOutput, inputTokens, outputTokens, processingMs }`. Isolates route handlers from the Anthropic client; enables future provider swap without route changes. |
| `lib/db/change-intelligence-analysis.ts` | Service: read inputs, build prompt, write results |
| `app/api/change-intelligence/analyses/[id]/generate/route.ts` | POST /generate handler |
| `app/api/change-intelligence/analyses/[id]/retry/route.ts` | POST /retry handler |
| `app/api/change-intelligence/analyses/[id]/cancel/route.ts` | POST /cancel handler |
| `app/api/change-intelligence/analyses/[id]/route.ts` | GET /:id — updated to include analysis_json when completed |
| `app/change-intelligence/analyses/[id]/page.tsx` | Detail page — all lifecycle states + report renderer |

---

## Database Changes

### Migration: `033_change_intelligence.sql`

Adds 9 nullable columns to the existing `change_analyses` table. No new tables. No existing columns modified.

```sql
-- All additive. No existing column is modified.
ALTER TABLE change_analyses ADD COLUMN IF NOT EXISTS provider text;
ALTER TABLE change_analyses ADD COLUMN IF NOT EXISTS temperature real;
ALTER TABLE change_analyses ADD COLUMN IF NOT EXISTS analysis_json jsonb;
ALTER TABLE change_analyses ADD COLUMN IF NOT EXISTS analysis_schema_version text;
ALTER TABLE change_analyses ADD COLUMN IF NOT EXISTS risk_level text;
ALTER TABLE change_analyses ADD COLUMN IF NOT EXISTS change_type_summary text;
ALTER TABLE change_analyses ADD COLUMN IF NOT EXISTS input_tokens integer;
ALTER TABLE change_analyses ADD COLUMN IF NOT EXISTS output_tokens integer;
ALTER TABLE change_analyses ADD COLUMN IF NOT EXISTS processing_ms integer;
ALTER TABLE change_analyses ADD COLUMN IF NOT EXISTS retry_count smallint NOT NULL DEFAULT 0;

-- Partial index for fast lookup of failed analyses eligible for retry
CREATE INDEX IF NOT EXISTS idx_change_analyses_retry
  ON change_analyses (id)
  WHERE status = 'failed' AND retry_count < 3;

-- Index for risk-level filtering on the list view (populated on completion)
CREATE INDEX IF NOT EXISTS idx_change_analyses_risk_level
  ON change_analyses (risk_level)
  WHERE risk_level IS NOT NULL;

-- Add supplemental_context to the input_type CHECK constraint.
-- The constraint must be dropped and recreated because PostgreSQL does not support
-- ALTER CONSTRAINT to extend a CHECK. This is idempotent: IF NOT EXISTS guards prevent
-- duplicate constraint errors on re-run.
ALTER TABLE change_analysis_inputs
  DROP CONSTRAINT IF EXISTS chk_change_analysis_inputs_type;

ALTER TABLE change_analysis_inputs
  ADD CONSTRAINT chk_change_analysis_inputs_type
    CHECK (input_type IN (
      'pr_diff', 'pr_description', 'requirement_text', 'acceptance_criteria',
      'prd_text', 'jira_story', 'markdown_spec', 'supplemental_context'
    ));
```

Full column reference: `data-model.md`, section "M3 Additions."

---

## API Specification

### POST /api/change-intelligence/analyses/:id/generate

**Auth:** `requireAuth()` — feature flag checked before auth.
**Precondition:** `status = 'ready'` — returns 409 for any other status.
**Execution:** Synchronous. Awaits AI completion. Server timeout: 120 seconds.

**Status transitions:**
- `ready` → `processing` → `completed` (AI succeeded)
- `ready` → `processing` → `failed` (AI failed)

**Response:** HTTP 200 for both `completed` and `failed` outcomes. Client reads `data.status`. See D-036.

**Error codes on failure:**

| Code | Condition |
|------|-----------|
| `PROVIDER_TIMEOUT` | Anthropic API call timed out |
| `PROVIDER_ERROR` | Anthropic returned an error response |
| `INVALID_OUTPUT` | Response is not valid JSON |
| `SCHEMA_VALIDATION_FAILED` | JSON is valid but fails schema validation |
| `INPUT_TOO_LARGE` | Combined inputs exceed `MAX_TOTAL_INPUT_CHARACTERS` |
| `DB_WRITE_FAILED` | DB write failed after successful AI call |
| `UNKNOWN_ERROR` | Any other unexpected error |

Full contract: `api-design.md`, Milestone 3 section.

---

### POST /api/change-intelligence/analyses/:id/retry

**Auth:** `requireAuth()`.
**Precondition:** `status = 'failed'` — 409 for any other status. `retry_count < 3` — 422 if at limit.
**Execution:** Same as /generate. Uses the same persisted inputs.

**Behavior:**
- Increment `retry_count` atomically (conditional UPDATE: `WHERE status = 'failed' AND retry_count < 3`).
- Clear `error_code` and `error_message`.
- Execute AI analysis.
- Persist result.

See D-032 for the in-place retry rationale.

---

### POST /api/change-intelligence/analyses/:id/cancel

**Auth:** `requireAuth()`.
**Precondition:** `status = 'ready'` — 409 for any other status. In-progress analyses cannot be cancelled in M3 (D-033).

**Transition:** `ready` → `cancelled`.

---

### GET /api/change-intelligence/analyses/:id (Enhanced)

Returns `analysis_json` in the response body when `status = 'completed'`. For all other statuses, `analysis_json` is null in the response. The list endpoint (`GET /analyses`) never returns `analysis_json`. See D-009.

---

## Prompt Design

Full prompt specification: `prompt-design.md`.

### Prompt Version

`m3-v1` — constant in `lib/ai/prompts/change-intelligence.ts`.

### Input Ordering

1. PR diff inputs
2. Requirement inputs (requirement_text, jira_story, acceptance_criteria, prd_text, markdown_spec)
3. Supplemental context

### Input Limits

- `MAX_DIFF_CHARACTERS`: 100,000
- `MAX_REQUIREMENT_CHARACTERS`: 50,000
- `MAX_TOTAL_INPUT_CHARACTERS`: 200,000

### Temperature

Fixed at 0.2 per D-034.

---

## Output Schema

Full schema specification: `analysis-schema.md`.

Schema version: `1.0.0`. The top-level fields are:
`schema_version`, `executive_summary`, `change_summary`, `risk_assessment`, `impacted_components`, `regression_recommendations`, `high_risk_scenarios`, `missing_requirements`, `ambiguities`, `edge_cases`, `recommended_test_focus`, `confidence`, `metadata`.

Validation enforces: required fields present, enum values from allowed sets, `recommended_test_focus` non-empty, `risk_level` consistency, non-negative `input_count`.

---

## Processing Lifecycle

### State Machine (M3-specific transitions)

```
draft ──────────────────────────────────────────► cancelled
  │
  │  (mark ready)
  ▼
ready ──────────────────────────────────────────► cancelled
  │
  │  POST /generate
  ▼
processing
  │
  ├──► completed   (AI call succeeded, validation passed, DB write succeeded)
  │
  └──► failed      (any error at any stage)
          │
          │  POST /retry (retry_count < 3)
          ▼
       processing
          │
          ├──► completed
          └──► failed (retry_count incremented)
```

### Atomicity Requirements

1. The `ready → processing` transition must be atomic. Use `UPDATE ... WHERE status = 'ready' AND id = :id` and check rows affected. If 0 rows updated, return 409.
2. The `processing → completed` persistence must be a single transaction. If the transaction fails, attempt 3 DB-write retries, then set `error_code = 'DB_WRITE_FAILED'`.
3. The `retry_count` increment must be atomic. Use `UPDATE ... WHERE id = :id AND status = 'failed' AND retry_count < 3`. If 0 rows affected, return 422.

---

## Error Strategy

### Error Code Classification

| Category | Codes | Retry Likely to Help? |
|----------|-------|----------------------|
| Transient | `PROVIDER_TIMEOUT`, `PROVIDER_ERROR` | Yes |
| Structural | `INVALID_OUTPUT`, `SCHEMA_VALIDATION_FAILED` | No (prompt issue) |
| Input | `INPUT_TOO_LARGE` | No (user must change inputs) |
| Infrastructure | `DB_WRITE_FAILED` | Yes |
| Unknown | `UNKNOWN_ERROR` | Maybe |

### Error Logging

All errors are logged server-side with:
- `analysis_id`
- `error_code`
- `retry_count` at time of failure
- Sanitized error detail (no user content, no secrets)

Error responses to the client never contain:
- Stack traces
- SQL error messages or driver details
- Anthropic API keys or response bodies
- Content from the analysis inputs

---

## UI States

Full UI specification: `ui-design.md`.

### Quick Reference

| Status | Primary Content | Primary Action |
|--------|-----------------|---------------|
| `ready` | Input list (read-only) | "Generate QA Intelligence" |
| `processing` | Spinner + "Generating QA Intelligence…" + elapsed timer | None |
| `completed` | QA Intelligence Report + metadata card | None |
| `failed` | Error panel + retry count | "Retry Analysis" (if retries remain) |
| `cancelled` | Read-only summary | Delete |

### Report Section Order (Completed State)

1. Executive Summary (always shown)
2. Change Summary (always shown)
3. Risk Assessment (always shown)
4. Impacted Components (hidden if empty)
5. Regression Recommendations (hidden if empty)
6. High Risk Scenarios (hidden if empty)
7. Missing Requirements (hidden if empty)
8. Ambiguities (hidden if empty)
9. Edge Cases (hidden if empty)
10. Recommended Test Focus (always shown)
11. Confidence and Limitations (always shown)

---

## Implementation Plan

### Phase 1 — Database

- Write `033_change_intelligence.sql` with 9 new columns + retry_count + indexes.
- Test migration on a copy of the production schema with `031_change_intelligence` and `032_change_intelligence_pr_description` already applied.

### Phase 2 — Backend Service Layer

- Implement `lib/db/change-intelligence-analysis.ts`:
  - `getAnalysisWithInputs(id)` — fetch analysis + inputs
  - `transitionToProcessing(id)` — atomic `ready → processing`
  - `persistSuccess(id, result, metadata)` — atomic `processing → completed`
  - `persistFailure(id, errorCode, errorMessage, metadata)` — atomic `processing → failed`
  - `incrementRetryCount(id)` — atomic `retry_count` increment with 422 guard

### Phase 3 — Prompt Builder

- Implement `buildPrompt(inputs)` in `lib/ai/prompts/change-intelligence.ts`:
  - Read inputs from DB by `input_type`.
  - Check total character count; throw `INPUT_TOO_LARGE` if exceeded.
  - Assemble labeled input blocks.
  - Return `{ systemPrompt, userPrompt }`.
  - Export `CHANGE_INTELLIGENCE_PROMPT_VERSION = 'm3-v1'`.

### Phase 4 — AI Client Integration

- Call `generateStructuredAIResponse<T>()` with temperature 0.2.
- Capture `usage.input_tokens`, `usage.output_tokens`.
- Record `processing_ms` (wall clock around the AI call).

### Phase 5 — Output Validation

- Implement `validateAnalysisOutput(raw)`:
  - JSON parse → `INVALID_OUTPUT` on failure.
  - Schema validate → `SCHEMA_VALIDATION_FAILED` on failure.
  - Return validated object on success.

### Phase 6 — Persistence

- On validation success: call `persistSuccess()`.
- On any error: call `persistFailure()` with the appropriate error code.
- If `persistSuccess()` fails: retry up to 3 times; if all fail, call `persistFailure(id, 'DB_WRITE_FAILED', ...)`.

### Phase 7 — API Routes

- `POST /generate`: precondition → transition → build prompt → AI call → validate → persist → return 200.
- `POST /retry`: precondition → atomic increment → same flow as generate.
- `POST /cancel`: precondition → transition → return 200.
- `GET /:id`: include `analysis_json` in response when `status = 'completed'`.

### Phase 8 — UI — Processing States

- Update the analysis detail page to handle: processing state (spinner + timer), failed state (error panel + retry button), cancelled state.
- Wire "Generate QA Intelligence" button to `POST /generate`.
- Wire "Retry Analysis" button to `POST /retry`.
- Wire "Cancel" button to `POST /cancel`.

### Phase 9 — UI — Report Rendering

- Implement `QaIntelligenceReport` component.
- Render all 11 sections in the documented order.
- Hide sections 4–9 when their arrays are empty.
- Implement risk level badge with color + accessible text.
- Implement metadata card.

### Phase 10 — Testing

- Unit tests: prompt builder, input size validator, output schema validator, error translator.
- API tests: all 3 new endpoints × happy path and all documented error cases.
- API test: concurrent retry race condition (retry_count enforcement).
- Migration test: 032 applies cleanly after 031.

### Phase 11 — Polish

- Error message mapping in UI (see `ui-design.md`).
- Elapsed timer in processing state.
- Risk level badge on list page (pending OQ-M3-UI-01 resolution).
- Regression smoke test of all existing workflows.

---

## Test Strategy

### Unit Tests

| Module | What to Test |
|--------|-------------|
| Prompt builder | Input ordering, character limit enforcement, labeled block format |
| Input size validator | Exact limit boundary (200,000 chars), multi-input accumulation |
| Output validator — JSON parse | Valid JSON, invalid JSON, empty string, null |
| Output validator — schema | All required fields, each invalid enum, missing recommended_test_focus item, risk_level mismatch |
| Error translator | Each `error_code` constant maps to correct HTTP behavior |

### API Tests

| Endpoint | Cases |
|----------|-------|
| POST /generate | Happy path → completed; PROVIDER_TIMEOUT → failed; INVALID_OUTPUT → failed; SCHEMA_VALIDATION_FAILED → failed; INPUT_TOO_LARGE → failed; 409 for draft/processing/completed/cancelled status; 403 feature disabled; 401 unauthenticated |
| POST /retry | Happy path → completed; 409 for non-failed status; 422 for retry_count = 3; concurrent retry race → exactly one 422 |
| POST /cancel | Happy path → cancelled; 409 for processing/completed/failed status; 403 feature disabled |
| GET /:id | analysis_json present when completed; analysis_json null when not completed |

### Migration Tests

- Apply 031, 032, then 033 — verify all 10 columns added (9 nullable + retry_count), retry_count defaults to 0 for existing rows.
- Apply 033 twice (idempotency check).

---

## Architectural Decisions

All M3 architectural decisions are documented in `decisions.md` with full rationale:

| Decision | Summary |
|----------|---------|
| D-028 | AI output stored as JSONB — findings are conceptual, not DB rows; M4+ tables deferred |
| D-029 | Synchronous execution — no background jobs in M3; 120-second server timeout |
| D-030 | Findings are conceptual entities in M3 — not materialized as DB rows |
| D-031 | Five denormalized projection columns populated atomically on completion: `change_summary`, `requirement_summary`, `risk_level`, `analysis_schema_version`, `change_type_summary` |
| D-032 | Retry updates existing record in-place — max 3 retries; no new record created |
| D-033 | Cancel from ready only — in-progress analyses cannot be interrupted in M3 |
| D-034 | Temperature fixed at 0.2 — not user-configurable in M3 |
| D-035 | Prompt versioning required — `m3-v1` is the M3 initial version |
| D-036 | HTTP 200 for both completed and failed outcomes — client reads `data.status` |
| D-037 | `analysis_json` is null on failure — all-or-nothing persistence; partial output never stored |
| D-038 | M3 migration is `033_change_intelligence` — `032` prefix already used by `032_change_intelligence_pr_description` (M2 UX polish) |
| D-039 | `supplemental_context` is a valid `input_type`; `prd_text` is the canonical PRD input type (not `prd_document`) |
| D-040 | `change-intelligence-provider.ts` is a thin seam only — does not imply multi-provider support in M3 |
| D-041 | `schema_version: "1.0.0"` is the canonical M3 output schema version; server validator rejects any other value |

---

## Resolved Design Questions

All open questions for M3 have been resolved as of 2026-07-27.

### OQ-M3-01 — Should failed analyses persist partial analysis_json? [RESOLVED]

**Resolution:** No. Complete valid output only. Partial output creates an ambiguous data state. The full failed response is logged server-side for debugging; nothing is stored in `analysis_json`.
**Rationale:** A null `analysis_json` on a failed record is an unambiguous signal. Partial data creates uncertainty about which fields can be trusted.
**Related decision:** D-037
**Resolved:** 2026-07-27

### OQ-M3-02 — Should the generate endpoint timeout exceed 120 seconds for large inputs? [RESOLVED]

**Resolution:** 120 seconds for M3. Revisit in M12 if pilot data shows large inputs frequently exceeding this limit.
**Rationale:** No empirical data justifies a longer timeout before the pilot. 120 seconds is already generous for typical inputs.
**Related decision:** D-029
**Resolved:** 2026-07-27

### OQ-M3-03 — When DB write fails after AI success, retry DB write or mark failed? [RESOLVED]

**Resolution:** Retry the DB write up to 3 times with exponential backoff. If all retries fail, set `error_code = 'DB_WRITE_FAILED'` and leave the analysis in a recoverable state for manual admin recovery.
**Rationale:** Retrying the DB write is cheaper than retrying the full AI call. Three retries provide resilience against transient DB issues without infinite loops.
**Related decision:** D-037
**Resolved:** 2026-07-27

### OQ-M3-04 — Populate denormalized columns on every GET or only on completion? [RESOLVED]

**Resolution:** Written once on completion. Not recomputed on subsequent reads. The fields are display-only; `analysis_json` is the authoritative data.
**Rationale:** Recomputing on every read adds per-request overhead and complexity with no benefit in the expected access pattern.
**Related decision:** D-031
**Resolved:** 2026-07-27

### OQ-M3-05 — Is 3-retry maximum sufficient or should it be configurable? [RESOLVED]

**Resolution:** Hardcoded at 3 for M3. Configurable retry limits are [FUTURE] (M12 candidate) if pilot data shows demand.
**Rationale:** Simplicity outweighs flexibility at pilot scale. Adding configuration requires UI surface area and validation logic that is out of scope for M3.
**Related decision:** D-032, D-035
**Resolved:** 2026-07-27

### OQ-M3-UI-01 — Risk level badge in the list view [RESOLVED]

**Resolution:** `risk_level text NULLABLE` and `change_type_summary text NULLABLE` are included in the `033_change_intelligence.sql` migration. Both are populated atomically with `change_summary` and `requirement_summary` on successful completion. No `analysis_json` extraction is needed in list queries.
**Related decision:** D-031
**Resolved:** 2026-07-27

---

## Open Questions

All M3 open questions are resolved. See Resolved Design Questions above.

---

## Acceptance Criteria

Full acceptance criteria: `acceptance-criteria.md`, Section M3-AC-01 through M3-AC-87.

### Summary by Category

| Category | Criteria |
|----------|---------|
| Feature gate and auth | M3-AC-01 – M3-AC-03 |
| Preconditions and state validation | M3-AC-04 – M3-AC-12 |
| Execution and AI processing | M3-AC-13 – M3-AC-22 |
| Prompt building | M3-AC-23 – M3-AC-28 |
| AI output and schema validation | M3-AC-29 – M3-AC-36 |
| Persistence | M3-AC-37 – M3-AC-43 |
| Retry | M3-AC-44 – M3-AC-48 |
| Cancellation | M3-AC-49 – M3-AC-51 |
| UI — processing state | M3-AC-52 – M3-AC-55 |
| UI — completed report | M3-AC-56 – M3-AC-62 |
| UI — failed state | M3-AC-63 – M3-AC-66 |
| Error handling | M3-AC-67 – M3-AC-73 |
| Security and privacy | M3-AC-74 – M3-AC-77 |
| Performance | M3-AC-78 – M3-AC-80 |
| Auditability | M3-AC-81 – M3-AC-83 |
| Migration | M3-AC-84 – M3-AC-85 |
| Backward compatibility | M3-AC-86 – M3-AC-87 |

---

## Security Considerations

- Input content (diff text, requirement text) is never logged at any level.
- `analysis_json` is not returned in list API responses.
- `error_message` stored in the DB contains sanitized text only — no stack traces, SQL, or driver details.
- Anthropic API key is never returned in any API response.
- All M3 endpoints enforce the feature flag before auth, and auth before business logic.
- `retry_count` enforcement uses a conditional DB UPDATE to prevent race conditions (not application-layer checking).

---

## Performance Targets

| Metric | Target |
|--------|--------|
| POST /generate — typical inputs | 15–60 seconds |
| POST /generate — maximum inputs | Under 120 seconds |
| GET /analyses/:id — completed analysis | Under 500ms |
| GET /analyses — list (analysis_json excluded) | Under 200ms |
| analysis_json in list query SELECT clause | Never |

---

## Rollout Plan

1. Apply `031_change_intelligence.sql` and `032_change_intelligence_pr_description.sql` (if not already applied) to the production database, then apply `033_change_intelligence.sql`, before deploying M3 application code.
2. Deploy M3 application code with feature flag still off.
3. Enable feature flag for the internal QA team (the first pilot group from M13 planning).
4. Monitor: PROVIDER_TIMEOUT rate, SCHEMA_VALIDATION_FAILED rate, retry usage, processing duration.
5. After 5 successful end-to-end analyses from the internal team, open to the broader pilot group (M13 scope).

---

## Handoff to M4 — Requirement Comparison

M4 (Requirement Comparison) will introduce individual finding review. When M4 is ready to begin:

1. The `analysis_json.missing_requirements` and `analysis_json.recommended_test_focus` arrays become the candidates for extraction into `change_requirements` DB rows.
2. M4 will introduce the first DB entity tables (`change_requirements`) extracted from `analysis_json`.
3. M4 will implement `PATCH /analyses/:id/requirements/:requirementId` for human override of AI coverage assessments.
4. No `analysis_json` structure changes are expected for M4. If the M4 schema differs, a new `schema_version` will be defined.

The `analysis_json` JSONB field is designed for M4 compatibility: the `missing_requirements` and `recommended_test_focus` arrays contain all the structured data M4 will need to populate the `change_requirements` table.

---

## Definition of Done

M3 is done when:

- [ ] Migration `033_change_intelligence.sql` verified against a production schema copy with migrations 031 and 032 already applied
- [ ] POST /generate tested end-to-end with real Anthropic API call
- [ ] POST /retry tested including the race condition (concurrent requests)
- [ ] POST /cancel tested
- [ ] All M3-AC-01 through M3-AC-87 acceptance criteria pass
- [ ] All BC-01 through BC-15 backward compatibility criteria pass
- [ ] All M2-AC-01 through M2-AC-46 criteria continue to pass
- [ ] Unit tests written for prompt builder, validator, and error translator
- [ ] API tests written for all new endpoints
- [ ] Regression smoke test of all existing workflows passes
- [ ] No critical or high bugs remain open
- [ ] `CHANGELOG.md` updated
