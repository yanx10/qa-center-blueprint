# API Design — Change Intelligence

## Overview

Change Intelligence introduces a new API namespace under the existing Next.js App Router structure. All new endpoints are additive. No existing API routes are modified.

**API namespace:** `/api/change-intelligence/`

All routes in this namespace:
1. Check the `CHANGE_INTELLIGENCE_ENABLED` feature flag independently. When the flag is off, all endpoints return a 403 — regardless of session state.
2. Call `requireAuth()` explicitly. The Next.js middleware does not protect `/api/*` routes. Every handler must call `requireAuth()` at the top.

---

## Confirmed Conventions (from Milestone 0)

### Response Envelope

All routes use the existing application response shape:

**Success:**
```json
{ "success": true, "data": {} }
```

**Failure:**
```json
{ "success": false, "error": "Human-readable error message" }
```

### Route Handler Pattern

Every Change Intelligence handler follows this template:

```typescript
import { isChangeIntelligenceEnabled } from '@/lib/features';
import { requireAuth } from '@/lib/auth';
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  if (!isChangeIntelligenceEnabled()) {
    return NextResponse.json(
      { success: false, error: 'Change Intelligence is not enabled.' },
      { status: 403 }
    );
  }

  const { ctx, errorResponse } = await requireAuth();
  if (errorResponse) return errorResponse;

  try {
    // handler logic
    return NextResponse.json({ success: true, data: { ... } });
  } catch (error) {
    console.error('[change-intelligence]', error);
    return NextResponse.json(
      { success: false, error: 'An unexpected error occurred.' },
      { status: 500 }
    );
  }
}
```

The feature-flag check comes before auth. A disabled feature does not reveal authentication state to unauthenticated callers.

### Authentication

```typescript
const { ctx, errorResponse } = await requireAuth();
if (errorResponse) return errorResponse;
// ctx.userId — UUID string matching users.id
// ctx.email  — user email
// ctx.role   — 'admin' | 'member' | 'viewer'
```

### Database Access

```typescript
import sql from '@/lib/db';
const rows = await sql`SELECT * FROM change_analyses WHERE id = ${id}`;
```

---

## Compatibility Rules

1. No existing API route is modified.
2. All new routes are additive and isolated under `/api/change-intelligence/`.
3. Auth is handled using `requireAuth()` from `lib/auth.ts` — no new auth mechanism.
4. Error responses match the existing envelope: `{ success: false, error: "..." }`.
5. The existing Anthropic and Jira integration settings are read-only from this namespace — not duplicated.
6. Rate limiting is not implemented in Phase 1. No rate limiting exists anywhere in the current application.

---

## Endpoints by Milestone

### Milestone 1 — Feature Shell

#### `GET /api/change-intelligence/status` [OPTIONAL]

**Purpose:** Confirm that the feature flag is enabled and the caller is authenticated.
**Auth:** `requireAuth()` — returns 401 if not authenticated.
**Feature flag:** Returns 403 if `CHANGE_INTELLIGENCE_ENABLED` is unset.
**Milestone:** 1 (optional — include if needed for frontend feature detection)
**Database:** No Change Intelligence tables are queried or mutated.

**Response — 200**
```json
{
  "success": true,
  "data": {
    "enabled": true
  }
}
```

**Error Responses**

| Status | Condition |
|--------|-----------|
| 401 | No valid session |
| 403 | Feature flag disabled — `{ "success": false, "error": "Change Intelligence is not enabled." }` |

---

### Milestone 2 — Analysis Persistence Foundation

> **Note:** The section below reflects the original API design draft, which pre-dates the M2 milestone restructuring. The M2 milestone was redesigned in the current blueprint — see [milestones/m2-analysis-persistence-foundation.md](milestones/m2-analysis-persistence-foundation.md) for the authoritative M2 API contracts. M3 AI processing endpoints are documented in the Milestone 3 section below.

**Authoritative M2 endpoints** (full contracts in [milestones/m2-analysis-persistence-foundation.md](milestones/m2-analysis-persistence-foundation.md)):

- `GET /api/change-intelligence/analyses` — List analyses with offset pagination (page/limit, sort: updated_at DESC + id DESC)
- `POST /api/change-intelligence/analyses` — Create a new analysis in draft status
- `GET /api/change-intelligence/analyses/:id` — Get analysis detail with input metadata (enhanced in M3 to return `analysis_json`)
- `PATCH /api/change-intelligence/analyses/:id` — Update title or transition status (draft → ready, draft/ready → cancelled)
- `POST /api/change-intelligence/analyses/:id/inputs` — Add or replace an input on a draft analysis
- `DELETE /api/change-intelligence/analyses/:id/inputs/:inputId` — Remove an input from a draft analysis
- `DELETE /api/change-intelligence/analyses/:id` — Delete a draft or cancelled analysis

---

#### [LEGACY DRAFT] `POST /api/change-intelligence/analyses`

> The endpoint below documents the original design intent before M2 was restructured. It is superseded by the M2 milestone specification linked above.

**Purpose:** Create a new analysis record and trigger AI processing.
**Auth:** `requireAuth()`
**Feature flag:** 403 if disabled
**Milestone:** 2

**Request**
```json
{
  "projectId": "uuid or null",
  "releaseId": "uuid or null",
  "inputs": [
    {
      "type": "pr_diff",
      "content": "string"
    },
    {
      "type": "requirement_text",
      "content": "string"
    }
  ]
}
```

Valid `type` values: `pr_diff`, `pr_description`, `requirement_text`, `jira_story`, `acceptance_criteria`, `prd_text`, `markdown_spec`, `supplemental_context`

**Response — 201**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "submitted",
    "createdAt": "ISO 8601 timestamp"
  }
}
```

**Error Responses**

| Status | Condition |
|--------|-----------|
| 400 | Missing required inputs (no diff, no requirement source, or input exceeds size limit) |
| 401 | No valid session |
| 403 | Feature flag disabled |
| 500 | AI processing error — error detail included in response; does not affect other routes |

---

#### `GET /api/change-intelligence/analyses`

**Purpose:** List saved analyses.
**Auth:** `requireAuth()`
**Feature flag:** 403 if disabled
**Milestone:** 2

**Query parameters**

| Param | Type | Description |
|-------|------|-------------|
| `projectId` | `uuid` | Filter by project |
| `status` | `string` | Filter by status (`draft`, `submitted`, `complete`, `error`) |

**Response — 200**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "status": "complete",
        "projectId": "uuid or null",
        "releaseId": "uuid or null",
        "changeSummary": "string or null",
        "createdAt": "ISO 8601 timestamp",
        "updatedAt": "ISO 8601 timestamp"
      }
    ],
    "total": 42
  }
}
```

**Note:** No standardized pagination convention exists in the current application. Pagination will be standardized before Milestone 8. The initial list endpoint returns all matching records with a `total` count.

---

#### `GET /api/change-intelligence/analyses/:id`

**Purpose:** Retrieve the full result of a single analysis.
**Auth:** `requireAuth()`
**Feature flag:** 403 if disabled
**Milestone:** 2

**M3 enhancement:** When `status = 'completed'`, the response includes `analysis_json` containing the full QA Intelligence Report. When status is any other value, `analysis_json` is null. The field is never returned in the list endpoint (D-009).

**Response — 200**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "complete",
    "projectId": "uuid or null",
    "releaseId": "uuid or null",
    "aiModel": "string",
    "workflowVersion": "string",
    "changeSummary": "string",
    "requirementSummary": "string",
    "requirements": [
      {
        "id": "uuid",
        "ordinal": 1,
        "requirementText": "string",
        "coverageStatus": "implemented",
        "evidence": "string",
        "confidence": "high",
        "reviewerOverride": "string or null",
        "reviewerNote": "string or null"
      }
    ],
    "riskFindings": [
      {
        "id": "uuid",
        "riskCategory": "string",
        "impactedArea": "string",
        "description": "string",
        "evidence": "string",
        "confidence": "medium",
        "reviewStatus": "unreviewed"
      }
    ],
    "generatedTestCases": [
      {
        "id": "uuid",
        "title": "string",
        "reviewStatus": "pending",
        "importedTestCaseId": "uuid or null"
      }
    ],
    "playwrightProposals": [
      {
        "id": "uuid",
        "description": "string",
        "reviewStatus": "pending"
      }
    ],
    "createdAt": "ISO 8601 timestamp",
    "updatedAt": "ISO 8601 timestamp"
  }
}
```

**Error Responses**

| Status | Condition |
|--------|-----------|
| 401 | No valid session |
| 403 | Feature flag disabled |
| 404 | Analysis not found |

---

### Milestone 3 — AI Change Analysis

---

#### `POST /api/change-intelligence/analyses/:id/generate`

**Purpose:** Trigger AI analysis on a ready analysis and return the completed result.
**Auth:** `requireAuth()`
**Feature flag:** 403 if disabled
**Milestone:** 3
**Execution:** Synchronous — the endpoint waits for AI completion. Typical duration: 15–60 seconds. Configured server timeout: 120 seconds.

**Preconditions:**
- Analysis must be in `ready` status. Returns 409 for any other status.

**Status transitions:**
```
ready → processing → completed   (AI succeeded)
ready → processing → failed      (AI failed — timeout, parse error, validation failure)
```

**Response contract:**
- Returns HTTP 200 for both `completed` and `failed` outcomes (D-030).
- Returns HTTP 409 if the analysis is not in `ready` status.
- Returns HTTP 500 only for unexpected server failures (e.g., DB write failure).
- The client must read `data.status` to determine the outcome.

**Request:** No body required.

**Success response (200) — AI completed:**
```json
{
  "success": true,
  "data": {
    "id": "7f3c…",
    "status": "completed",
    "provider": "anthropic",
    "ai_model": "claude-sonnet-4-6",
    "analysis_version": "m3-v1",
    "temperature": 0.2,
    "input_tokens": 12450,
    "output_tokens": 3280,
    "processing_ms": 34200,
    "change_summary": "Adds optional email notification preference to user settings",
    "requirement_summary": "User notification preferences not fully addressed in implementation",
    "analysis_json": { "schema_version": "1.0.0", "executive_summary": { "..." : "..." }, "...": "..." },
    "started_at": "2026-07-27T14:00:00Z",
    "completed_at": "2026-07-27T14:00:34Z",
    "updated_at": "2026-07-27T14:00:34Z"
  }
}
```

**Response (200) — AI failed:**
```json
{
  "success": true,
  "data": {
    "id": "7f3c…",
    "status": "failed",
    "error_code": "PROVIDER_TIMEOUT",
    "error_message": "The AI provider did not respond within the configured timeout.",
    "analysis_json": null,
    "retry_count": 0,
    "started_at": "2026-07-27T14:00:00Z",
    "completed_at": "2026-07-27T14:02:01Z",
    "updated_at": "2026-07-27T14:02:01Z"
  }
}
```

**Error responses:**

| Status | Condition |
|--------|-----------|
| 409 | Analysis is not in `ready` status |
| 403 | Feature disabled |
| 401 | Not authenticated |
| 500 | Unexpected server error (e.g., DB write failure — analysis may be left in `processing` state) |

**Error codes stored in `error_code` on failure:**

| error_code | Condition |
|-----------|-----------|
| `PROVIDER_TIMEOUT` | Anthropic API call timed out |
| `PROVIDER_ERROR` | Anthropic returned an error HTTP response |
| `INVALID_OUTPUT` | AI response could not be parsed as JSON |
| `SCHEMA_VALIDATION_FAILED` | JSON parsed but failed schema validation against `analysis-schema.md` |
| `INPUT_TOO_LARGE` | Combined inputs exceed the character limit (enforced before AI call) |
| `DB_WRITE_FAILED` | Database write failed after successful AI call — results cannot be saved |
| `UNKNOWN_ERROR` | Any other unexpected error |

---

#### `POST /api/change-intelligence/analyses/:id/retry`

**Purpose:** Retry a failed analysis using the same persisted inputs.
**Auth:** `requireAuth()`
**Feature flag:** 403 if disabled
**Milestone:** 3
**Execution:** Synchronous — same as /generate.

**Preconditions:**
- Analysis must be in `failed` status. Returns 409 for any other status.
- `retry_count` must be less than 3. Returns 422 if `retry_count >= 3`.

**Behavior:**
1. Validate preconditions.
2. Increment `retry_count` atomically (conditional `UPDATE WHERE status = 'failed' AND retry_count < 3`).
3. Clear `error_code` and `error_message`.
4. Set status to `processing`, set `started_at`.
5. Execute AI analysis (same as /generate).
6. Persist result (same as /generate).
7. Return 200 with the full analysis object.

**Request:** No body required.

**Response (200):** Same shape as /generate response.

**Error responses:**

| Status | Condition |
|--------|-----------|
| 409 | Analysis is not in `failed` status |
| 422 | `retry_count` has reached the maximum of 3 — no further retries permitted |
| 403 | Feature disabled |
| 401 | Not authenticated |
| 500 | Unexpected server error |

---

#### `POST /api/change-intelligence/analyses/:id/cancel`

**Purpose:** Cancel a ready analysis before AI processing begins.
**Auth:** `requireAuth()`
**Feature flag:** 403 if disabled
**Milestone:** 3

**Preconditions:**
- Analysis must be in `ready` status. Returns 409 for any other status.
- **M3 limitation:** Cancellation of an in-progress (`processing`) analysis is not supported. Synchronous execution cannot be interrupted. (D-033)

**Status transition:** `ready` → `cancelled`

**Request:** No body required.

**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "7f3c…",
    "status": "cancelled",
    "updated_at": "2026-07-27T14:00:00Z"
  }
}
```

**Error responses:**

| Status | Code | Message |
|--------|------|---------|
| 404 | `NOT_FOUND` | `"Analysis not found."` |
| 409 | `INVALID_STATUS` | `"Processing analyses cannot be cancelled. The analysis is running synchronously and cannot be interrupted."` (when `processing`) |
| 409 | `INVALID_STATUS` | `"Completed analyses cannot be cancelled."` (when `completed`) |
| 409 | `INVALID_STATUS` | `"Failed analyses cannot be cancelled. Use retry instead."` (when `failed`) |
| 409 | `INVALID_STATUS` | `"This analysis is already cancelled."` (when `cancelled`) |
| 409 | `INVALID_STATUS` | `"Draft analyses cannot be cancelled. Mark the analysis as ready first, or delete it."` (when `draft`) |
| 403 | — | Feature disabled |
| 401 | — | Not authenticated |

---

#### `GET /api/change-intelligence/analyses/:id` (M3 Enhancement)

The M2 detail endpoint is enhanced in M3 to return `analysis_json` when `status = completed` and to include all new M3 fields.

**New fields added in M3:** `provider`, `temperature`, `analysis_json` (when completed), `input_tokens`, `output_tokens`, `processing_ms`, `retry_count`, `error_code`, `error_message`, `started_at`, `completed_at`.

**`analysis_json` rules:**
- Returned only when `status = 'completed'`; `null` for all other statuses
- Never returned by the list endpoint (`GET /analyses`)

---

### Milestone 4 — Requirement Comparison

#### `PATCH /api/change-intelligence/analyses/:id/requirements/:requirementId`

**Purpose:** Record a human reviewer's override on a requirement's coverage status.
**Auth:** `requireAuth()`
**Feature flag:** 403 if disabled
**Milestone:** 4

**Request**
```json
{
  "reviewerOverride": "implemented",
  "reviewerNote": "string"
}
```

Valid `reviewerOverride` values: `implemented`, `partial`, `missing`, `ambiguous`, `unable_to_verify`

**Response — 200**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "reviewerOverride": "implemented",
    "reviewerNote": "string",
    "reviewedBy": "uuid",
    "reviewedAt": "ISO 8601 timestamp"
  }
}
```

---

### Milestone 4 — Risk and Regression Analysis

#### `PATCH /api/change-intelligence/analyses/:id/risk-findings/:findingId`

**Purpose:** Update a reviewer's status on a risk finding.
**Auth:** `requireAuth()`
**Feature flag:** 403 if disabled
**Milestone:** 4

**Request**
```json
{
  "reviewStatus": "acknowledged",
  "reviewerNote": "string"
}
```

Valid `reviewStatus` values: `acknowledged`, `disputed`

**Response — 200**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "reviewStatus": "acknowledged",
    "reviewedBy": "uuid",
    "reviewedAt": "ISO 8601 timestamp"
  }
}
```

---

### Milestone 5 — Manual Test Generation

#### `PATCH /api/change-intelligence/analyses/:id/test-cases/:testCaseId/review`

**Purpose:** Approve or reject a generated test-case proposal.
**Auth:** `requireAuth()`
**Feature flag:** 403 if disabled
**Milestone:** 5

**Request**
```json
{
  "decision": "approved"
}
```

Valid `decision` values: `approved`, `rejected`

**Response — 200**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "reviewStatus": "approved",
    "reviewedBy": "uuid",
    "reviewedAt": "ISO 8601 timestamp"
  }
}
```

---

#### `POST /api/change-intelligence/analyses/:id/test-cases/:testCaseId/import`

**Purpose:** Import an approved generated test case into the existing test-case library.
**Auth:** `requireAuth()` — required; do not repeat the AI Studio omission (D-014)
**Feature flag:** 403 if disabled
**Milestone:** 5
**Precondition:** `review_status` must be `approved`. Returns 409 if not approved.

**Request**
```json
{
  "suiteId": "uuid or null"
}
```

**Response — 201**
```json
{
  "success": true,
  "data": {
    "importedTestCaseId": "uuid",
    "importedToSuiteId": "uuid or null"
  }
}
```

**Error Responses**

| Status | Condition |
|--------|-----------|
| 401 | No valid session |
| 403 | Feature flag disabled |
| 404 | Test case not found |
| 409 | Test case has not been approved |
| 500 | Database write failure |

---

### Milestone 6 — Playwright Proposals

#### `PATCH /api/change-intelligence/analyses/:id/playwright-proposals/:proposalId`

**Purpose:** Record acceptance, rejection, or deferral of a Playwright proposal.
**Auth:** `requireAuth()`
**Feature flag:** 403 if disabled
**Milestone:** 6

**Request**
```json
{
  "reviewStatus": "accepted"
}
```

Valid `reviewStatus` values: `accepted`, `rejected`, `deferred`

**Response — 200**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "reviewStatus": "accepted",
    "reviewedBy": "uuid",
    "reviewedAt": "ISO 8601 timestamp"
  }
}
```

---

#### `GET /api/change-intelligence/analyses/:id/playwright-proposals/:proposalId/code`

**Purpose:** Retrieve the full Playwright code for a proposal (for display and copy-to-clipboard).
**Auth:** `requireAuth()`
**Feature flag:** 403 if disabled
**Milestone:** 6

**Response — 200**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "description": "string",
    "code": "string"
  }
}
```

---

## Feature Flag Behavior (All Endpoints)

When `CHANGE_INTELLIGENCE_ENABLED` is unset or falsy:

```
HTTP/1.1 403 Forbidden
Content-Type: application/json

{
  "success": false,
  "error": "Change Intelligence is not enabled."
}
```

The flag check runs before auth. A disabled feature does not reveal whether the caller has a valid session.

**Page behavior when disabled:** `notFound()` — the server renders a 404. This is separate from the API 403.

---

## Auth and Authorization

- All endpoints call `requireAuth()` explicitly. There is no middleware protection on `/api/*` routes.
- Access control in Phase 1 is org-wide: any authenticated user can access any analysis when the feature flag is on. The `created_by` field provides audit trail only.
- Analyses associated with a project do not add project-level access control in Phase 1. The existing application has no project-level access control.
- The multi-user collaboration model is defined by OD-005 (resolved): all authenticated org users can read and review any analysis.

---

## Rate Limiting

No rate limiting exists anywhere in the current application. Rate limiting is not implemented in Phase 1. AI-powered endpoints may take 15–40 seconds. Cost monitoring in Phase 1 is manual.

---

## Future Endpoints (Milestone 10+, Not in Phase 1)

These are placeholders, not commitments:

```
GET    /api/change-intelligence/pilot/users
POST   /api/change-intelligence/pilot/users
DELETE /api/change-intelligence/pilot/users/:userId
```

MCP tool endpoints are out of scope for Phase 1. See OD-015.

---

## Resolved Questions

All open questions from the previous API design version have been resolved by Milestone 0:

| Question | Answer |
|----------|--------|
| Existing API response envelope format | `{ success: boolean, data?, error? }` — confirmed |
| Authentication at the route level | `requireAuth()` called inside each handler — not middleware |
| Long-running AI requests | Synchronous — one blocking HTTP call per analysis (D-016) |
| Pagination convention | No standard exists. Use `{ items: [...], total: N }` for now |
| Existing API tests | None exist. Change Intelligence will be the first feature with API test coverage. |
