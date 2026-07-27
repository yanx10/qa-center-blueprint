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

### Milestone 2 — Manual Input Analysis

#### `POST /api/change-intelligence/analyses`

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

Valid `type` values: `pr_diff`, `requirement_text`, `jira_story`, `acceptance_criteria`, `prd_document`, `markdown_spec`

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

### Milestone 3 — Requirement Comparison

#### `PATCH /api/change-intelligence/analyses/:id/requirements/:requirementId`

**Purpose:** Record a human reviewer's override on a requirement's coverage status.
**Auth:** `requireAuth()`
**Feature flag:** 403 if disabled
**Milestone:** 3

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
