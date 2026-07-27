# Acceptance Criteria — Change Intelligence

Acceptance criteria are organized in two sections: backward compatibility criteria (which must pass before any milestone is considered complete) and feature criteria (specific to each capability milestone).

---

## Section 1 — Backward Compatibility Criteria

These criteria apply to every milestone. No milestone is accepted if any of these fail.

---

### BC-01 — Existing QA Center functionality continues to work

```
Given the Change Intelligence feature is in any state (enabled or disabled)
When an existing user accesses any existing QA Center feature
Then that feature operates exactly as it did before Change Intelligence was introduced
```

---

### BC-02 — Feature flag off produces no user-visible change

```
Given the Change Intelligence feature flag is disabled
When any user logs in and uses QA Center
Then no Change Intelligence navigation item is visible
And no Change Intelligence routes are accessible
And the application behaves identically to the pre-feature baseline
```

---

### BC-03 — Existing database records remain valid

```
Given Change Intelligence migrations have been applied
When an administrator queries any existing table
Then all pre-migration records are present and unmodified
And no existing column has been removed, renamed, or changed in type
And no existing constraint has been removed
```

---

### BC-04 — AI Studio workflows continue to work

```
Given Change Intelligence has been deployed
When a user submits a PRD to AI Studio
Then test cases are generated and displayed in the review interface
And the approve, reject, and import workflow completes successfully
And no data from AI Studio is affected by Change Intelligence tables or code
```

---

### BC-05 — Existing test-case import workflows continue to work

```
Given Change Intelligence has been deployed
When a user imports an AI Studio-generated test case into a suite
Then the import succeeds
And the test case appears in the test-case library
And the test case is visible in the assigned suite
```

---

### BC-06 — Existing MCP tools continue to work

```
Given Change Intelligence has been deployed
When an MCP client calls any existing MCP tool
Then the tool responds correctly
And no Change Intelligence code path is invoked
And existing test-case, test-run, and PRD workflow MCP tools behave as before
```

---

### BC-07 — Existing Release Quality Reports continue to generate

```
Given Change Intelligence has been deployed
When a user generates a Release Quality Report
Then the report completes successfully
And all existing metrics (DRE, escape rate, fix-fail rate, coverage, etc.) are present
And the public report link continues to work
```

---

### BC-08 — Development Gates remain unaffected

```
Given Change Intelligence has been deployed
When a user views Development Gates for any project
Then all six gate phases display correctly
And gate status updates continue to work
And no Change Intelligence data is injected into the gate view
```

---

### BC-09 — Existing authentication continues to work

```
Given Change Intelligence has been deployed
When a user logs in via Auth0 OIDC
Then authentication succeeds
And the session is established correctly
And the user is directed to the correct landing page
```

---

### BC-10 — Change Intelligence failure does not affect other areas

```
Given the Change Intelligence analysis API returns a 500 error
When a user is currently using a test run, the AI Studio, or any other existing feature
Then that feature continues to function normally
And no error from Change Intelligence propagates to unrelated pages or components
```

---

### BC-11 — No AI-generated test case enters a suite without human approval

```
Given Change Intelligence has generated a batch of test-case proposals
When the proposals are displayed to the user
Then no case has been automatically added to any test suite
And each proposal requires explicit approval before any import action is available
And the import action is only enabled after the case status is "approved"
```

---

### BC-12 — All AI conclusions display evidence and confidence

```
Given an analysis has completed
When any requirement coverage status, risk finding, or recommendation is displayed
Then each item shows its supporting evidence (diff excerpt or reasoning)
And each item shows a confidence label (high, medium, or low)
And no conclusion is presented without evidence
```

---

### BC-13 — Unable-to-verify is used when evidence is insufficient

```
Given an analysis was run on inputs that do not contain sufficient information
  to determine a requirement's status
When the requirement coverage is displayed
Then the status is "unable to verify" rather than a fabricated determination
And the evidence field explains what information was missing
```

---

### BC-14 — New migrations run safely on an existing production database

```
Given the Change Intelligence migration has not been applied to the production database
When the migration is applied
Then it completes without errors
And all existing tables, columns, and data remain unchanged
And no existing query used by the application returns different results
```

---

### BC-15 — Rolling back application code does not corrupt existing data

```
Given Change Intelligence has been deployed and used to create analysis records
When the application code is rolled back to the pre-Change-Intelligence version
Then all existing QA Center data (test cases, suites, runs, releases, etc.) remains intact
And the new Change Intelligence tables remain in the database but are not accessed
And no data loss or corruption occurs in existing tables
```

---

## Section 2 — Feature Acceptance Criteria

---

### Milestone 1 — Isolated Feature Shell

#### M1-AC-01 — Feature is disabled when `CHANGE_INTELLIGENCE_ENABLED` is unset

```
Given CHANGE_INTELLIGENCE_ENABLED is not set in the server environment
When any user accesses QA Center
Then no Change Intelligence navigation item is visible in SideNav
And the /change-intelligence route returns 404
And all existing pages load without error
```

#### M1-AC-02 — Feature is enabled when `CHANGE_INTELLIGENCE_ENABLED` is set

```
Given CHANGE_INTELLIGENCE_ENABLED is set to a non-empty string value in the environment
When an authenticated user accesses QA Center
Then the Change Intelligence navigation item is visible in SideNav
And navigating to /change-intelligence displays the empty-state shell page
```

#### M1-AC-03 — No NEXT_PUBLIC variable is required

```
Given CHANGE_INTELLIGENCE_ENABLED is a server-side environment variable only
When the application builds and runs
Then no NEXT_PUBLIC_CHANGE_INTELLIGENCE_ENABLED variable is defined or required
And the navigation item visibility is resolved from a server-rendered prop on SideNav
```

#### M1-AC-04 — SideNav receives resolved feature state from a server parent

```
Given app/layout.tsx is a server component
When the layout renders
Then it calls isChangeIntelligenceEnabled() on the server
And passes the resolved boolean as a prop to SideNav
And SideNav does not independently read environment variables or call the flag helper
```

#### M1-AC-05 — Navigation item is absent when feature is disabled

```
Given CHANGE_INTELLIGENCE_ENABLED is unset
When an authenticated user logs in
Then SideNav renders exactly the existing navigation items
And no Change Intelligence item is present
And the SideNav item count is unchanged from the pre-feature baseline
```

#### M1-AC-06 — Navigation item is visible when feature is enabled

```
Given CHANGE_INTELLIGENCE_ENABLED is set
When an authenticated user logs in
Then SideNav includes a Change Intelligence navigation item
And the item links to /change-intelligence
And all existing navigation items remain present and unchanged
```

#### M1-AC-07 — Direct access to /change-intelligence is blocked when disabled

```
Given CHANGE_INTELLIGENCE_ENABLED is unset
When any user navigates directly to /change-intelligence
Then the server returns a 404 response
And no Change Intelligence content is revealed
And the user is not redirected to an error page
```

#### M1-AC-08 — Direct access to /change-intelligence requires authentication

```
Given CHANGE_INTELLIGENCE_ENABLED is set
And the user is not authenticated
When the user navigates to /change-intelligence
Then the middleware redirects the user to /login
And the callbackUrl is set to /change-intelligence
```

#### M1-AC-09 — Any Change Intelligence API route calls requireAuth() explicitly

```
Given any Change Intelligence API route handler
When the handler processes a request
Then it calls requireAuth() at the top of the handler
And it returns the errorResponse if authentication fails
And it does not proceed to any business logic without a valid session
```

#### M1-AC-10 — Optional status endpoint returns 403 when feature is disabled

```
Given CHANGE_INTELLIGENCE_ENABLED is unset
And the user is authenticated
When GET /api/change-intelligence/status is called
Then the response is 403
And the body is { "success": false, "error": "Change Intelligence is not enabled." }
```

#### M1-AC-11 — Existing pages render correctly after layout prop addition

```
Given app/layout.tsx has been modified to read the feature flag and pass a prop to SideNav
When any existing QA Center page renders
Then the page renders without error
And no existing component receives an unexpected prop
And the layout renders identically to the pre-change baseline
```

#### M1-AC-12 — Existing SideNav behavior is unchanged except for the conditional item

```
Given CHANGE_INTELLIGENCE_ENABLED is unset
When the application is running
Then SideNav renders identically to the pre-change baseline in all respects
And no existing navigation item is affected by the new prop
And SideNav still hides on /login, /auth/*, and /reports/* paths
```

#### M1-AC-13 — No migration is introduced in Milestone 1

```
Given Milestone 1 has been implemented
When the database is inspected
Then no new Change Intelligence tables exist
And the migration file 030_change_intelligence.sql does not exist
And the schema.sql has not been modified
```

#### M1-AC-14 — No Anthropic call is introduced in Milestone 1

```
Given Milestone 1 has been implemented
When any Milestone 1 code path executes (page load, nav render, optional status endpoint)
Then no call is made to generateStructuredAIResponse or any Anthropic API
And lib/ai/anthropic.ts is not imported by any Milestone 1 file
```

#### M1-AC-15 — No existing AI Studio code is modified

```
Given Milestone 1 has been implemented
When app/ai-studio/page.tsx and all AI Studio API routes are inspected
Then none of these files have been modified
And AI Studio functionality is fully intact
```

#### M1-AC-16 — No existing MCP tools are modified

```
Given Milestone 1 has been implemented
When app/api/mcp/route.ts is inspected
Then the file has not been modified
And all 17 existing MCP tools respond correctly
```

#### M1-AC-17 — Existing tests, lint, type checks, and builds continue to pass

```
Given Milestone 1 has been implemented
When the test suite, lint checks, TypeScript compilation, and build are run
Then all existing tests pass
And no new lint errors are introduced
And TypeScript type checking succeeds
And the Next.js production build completes without error
```

#### M1-AC-18 — Change Intelligence failure cannot break unrelated routes

```
Given CHANGE_INTELLIGENCE_ENABLED is set
And the /change-intelligence page or status endpoint encounters an error
When a user is simultaneously using an unrelated QA Center page
Then the unrelated page continues to function normally
And the error is contained to the Change Intelligence route
```

---

### M1-AC-Legacy-01 — Route is accessible when feature is enabled (original criterion)

```
Given the feature flag is enabled
And the user is authenticated
When the user navigates to /change-intelligence
Then the page loads with an empty state
And the navigation item is visible
```

---

### M1-AC-Legacy-02 — Route is inaccessible when feature is disabled (original criterion)

```
Given the feature flag is disabled
When any user navigates directly to /change-intelligence
Then the server returns a 404 — not an error page
And no Change Intelligence content is revealed
```

---

### Milestone 2 — Analysis Persistence Foundation

#### M2-AC-01 — All M2 API routes return 403 when feature is disabled

```
Given CHANGE_INTELLIGENCE_ENABLED is not set
When any unauthenticated or authenticated user calls any M2 endpoint
Then the response is 403
And the body is { "success": false, "error": "Change Intelligence is not enabled." }
And no database query is executed
```

#### M2-AC-02 — All M2 API routes call requireAuth() before any business logic

```
Given any M2 route handler
When the handler processes a request with the feature enabled
Then it calls requireAuth() at the top of the handler, after the feature gate check
And it returns a 401 errorResponse if authentication fails
And it does not proceed to any database access without a valid session
```

#### M2-AC-03 — Unauthenticated requests to M2 routes return 401

```
Given CHANGE_INTELLIGENCE_ENABLED is set
And the request has no valid session
When any M2 endpoint is called
Then the response is 401
And the body is { "success": false, "error": "Authentication required." }
```

#### M2-AC-04 — POST /analyses creates a draft analysis with default values

```
Given an authenticated user calls POST /api/change-intelligence/analyses with an empty body
When the request is processed
Then a new change_analyses record is created with status 'draft' and trigger_type 'manual'
And created_by is set to the authenticated user's UUID
And the response title is a non-null auto-generated string in the format "Change Analysis — [Month Day, Year H:MM AM/PM]"
And the response status is 201
And the response body includes { "success": true, "data": { "id": "...", "status": "draft", "title": "..." } }
```

#### M2-AC-05 — POST /analyses with title stores the title

```
Given an authenticated user calls POST /api/change-intelligence/analyses with { "title": "Sprint 42 auth refactor" }
When the request is processed
Then the new record has title = 'Sprint 42 auth refactor'
And the response data includes the title field
```

#### M2-AC-06 — POST /analyses with project_id associates the analysis with a project

```
Given an authenticated user calls POST /api/change-intelligence/analyses with a valid project_id
When the request is processed
Then the new record has project_id set to the provided UUID
And the response data includes the project_id field
```

#### M2-AC-07 — POST /analyses with release_id associates the analysis with a release

```
Given an authenticated user calls POST /api/change-intelligence/analyses with a valid release_id
When the request is processed
Then the new record has release_id set to the provided UUID
And the response data includes the release_id field
```

#### M2-AC-08 — POST /analyses with a nonexistent project_id returns 400

```
Given an authenticated user calls POST /api/change-intelligence/analyses
With a project_id UUID that does not reference an existing project
When the request is processed
Then the response is 400
And the body is { "success": false, "error": "..." }
And no analysis record is created
```

#### M2-AC-09 — POST /analyses uses the standard response envelope

```
Given any successful POST /api/change-intelligence/analyses request
When the response is received
Then the body matches { "success": true, "data": { ... } }
And on failure the body matches { "success": false, "error": "..." }
```

#### M2-AC-10 — POST /analyses does not make any Anthropic API call

```
Given any POST /api/change-intelligence/analyses request
When the request is processed
Then no call is made to generateStructuredAIResponse or any Anthropic API
And lib/ai/anthropic.ts is not imported by any M2 route file
```

#### M2-AC-11 — POST /analyses response does not include input content

```
Given a POST /api/change-intelligence/analyses call succeeds
Then the response data does not include a content field
And the response data does not include a prd_snapshot field
```

#### M2-AC-12 — GET /analyses returns a paginated list

```
Given at least one analysis exists
When an authenticated user calls GET /api/change-intelligence/analyses
Then the response is 200
And the response data includes an analyses array and a pagination object
And the analyses array contains at most 25 records per page by default
```

#### M2-AC-13 — GET /analyses list excludes input content

```
Given analyses with attached inputs exist
When GET /api/change-intelligence/analyses is called
Then each analysis object in the array does not include a content field on any input
And each analysis object includes input_count (the number of attached inputs)
```

#### M2-AC-14 — GET /analyses page and limit parameters are respected

```
Given more than 25 analyses exist
When GET /api/change-intelligence/analyses?page=2&limit=5 is called
Then the response contains at most 5 analyses
And the pagination object shows page=2 and limit=5
```

#### M2-AC-14a — GET /analyses rejects invalid page and limit values

```
Given an authenticated user with the feature enabled
When GET /api/change-intelligence/analyses is called with any of the following:
  page=0, page=-1, page=abc, page=1.5,
  limit=0, limit=-1, limit=abc, limit=1.5, limit=101
Then the response is 400
And the body is { "success": false, "error": "<descriptive message>" }
And absent parameters apply the default (page=1, limit=25) without error
```

Contract clarification: defaults apply only when the parameter is absent (not supplied). A supplied parameter must be a valid positive integer within its allowed range. The server must not silently normalize invalid supplied values.

#### M2-AC-15 — GET /analyses returns an empty array when no analyses exist

```
Given no analyses have been created
When GET /api/change-intelligence/analyses is called
Then the response is 200
And data.analyses is an empty array
And data.pagination.total is 0
```

#### M2-AC-16 — GET /analyses/:id returns the full analysis record

```
Given a draft analysis with an ID exists
When GET /api/change-intelligence/analyses/:id is called with that ID
Then the response is 200
And the data object includes all analysis metadata fields
And the data object includes an inputs array
```

#### M2-AC-17 — GET /analyses/:id excludes input content

```
Given an analysis with inputs exists
When GET /api/change-intelligence/analyses/:id is called
Then each object in the inputs array does not include a content field
And each object does not include a prd_snapshot field
And each object does include content_hash, input_type, source_label, and created_at
```

#### M2-AC-18 — GET /analyses/:id returns 404 for a nonexistent ID

```
Given a UUID that does not reference any analysis
When GET /api/change-intelligence/analyses/:id is called
Then the response is 404
And the body is { "success": false, "error": "Analysis not found." }
```

#### M2-AC-19 — GET /analyses/:id returns M3 result fields as null for M2-lifecycle records

```
Given a draft or ready analysis (M3 pipeline has not run)
When GET /api/change-intelligence/analyses/:id is called
Then ai_model, change_summary, requirement_summary, error_code, error_message,
  started_at, and completed_at are all null in the response
```

#### M2-AC-20 — PATCH /analyses/:id can update title when status is draft

```
Given a draft analysis
When PATCH /api/change-intelligence/analyses/:id is called with { "title": "New title" }
Then the response is 200
And the analysis title is updated to "New title"
And updated_at is updated to the current time
```

#### M2-AC-21 — PATCH /analyses/:id can transition status to ready when pr_diff input exists

```
Given a draft analysis that has at least one input with input_type 'pr_diff'
When PATCH /api/change-intelligence/analyses/:id is called with { "status": "ready" }
Then the response is 200
And the analysis status transitions to 'ready'
```

#### M2-AC-22 — PATCH /analyses/:id transition to ready fails without a pr_diff input

```
Given a draft analysis with no inputs, or with only requirement-type inputs
When PATCH /api/change-intelligence/analyses/:id is called with { "status": "ready" }
Then the response is 400
And the body contains an error explaining that a pr_diff input is required
And the analysis remains in draft status
```

#### M2-AC-23 — PATCH /analyses/:id on a ready analysis returns 409 for status changes back to draft

```
Given an analysis in ready status
When PATCH /api/change-intelligence/analyses/:id is called with { "status": "draft" }
Then the response is 409
And the analysis status remains ready
```

#### M2-AC-24 — PATCH /analyses/:id updates updated_at on every successful write

```
Given any successful PATCH /api/change-intelligence/analyses/:id request
When the response is received
Then the data.updated_at value is newer than the previous updated_at value
```

#### M2-AC-25 — POST /analyses/:id/inputs adds an input to a draft analysis

```
Given a draft analysis
When POST /api/change-intelligence/analyses/:id/inputs is called
  with { "input_type": "requirement_text", "content": "As a user..." }
Then the response is 201
And a new change_analysis_inputs record is created for the analysis
And the response data includes id, input_type, content_hash, and created_at
And the response data does not include the content field
```

#### M2-AC-26 — POST /analyses/:id/inputs fails with 409 when analysis is not draft

```
Given an analysis in ready, processing, completed, failed, or cancelled status
When POST /api/change-intelligence/analyses/:id/inputs is called
Then the response is 409
And the body contains an error explaining that inputs cannot be modified in the current status
And no input record is created
```

#### M2-AC-27 — POST /analyses/:id/inputs computes content_hash server-side with canonicalization

```
Given POST /api/change-intelligence/analyses/:id/inputs is called with content "diff --git a/..."
When the input is stored
Then content_hash is set to the SHA-256 of the canonicalized content:
  remove UTF-8 BOM, normalize CRLF→LF, preserve other whitespace, encode as UTF-8, SHA-256, 64 lowercase hex characters
And content_hash is not accepted from the request body — the client cannot override it
And the stored hash is exactly 64 lowercase hexadecimal characters with no algorithm prefix
```

#### M2-AC-28 — POST /analyses/:id/inputs rejects unknown input_type values

```
Given POST /api/change-intelligence/analyses/:id/inputs is called with input_type "github_link"
When the request is processed
Then the response is 400
And the body contains an error listing the allowed input_type values
And no input record is created
```

#### M2-AC-29 — POST /analyses/:id/inputs rejects content exceeding 500,000 characters

```
Given POST /api/change-intelligence/analyses/:id/inputs is called
  with content containing more than 500,000 characters
When the request is processed
Then the response is 400
And the body is { "success": false, "error": "Content exceeds the 500,000 character limit." }
And no input record is created
```

#### M2-AC-30 — DELETE /analyses/:id/inputs/:inputId removes the input from a draft analysis

```
Given a draft analysis with an input identified by inputId
When DELETE /api/change-intelligence/analyses/:id/inputs/:inputId is called
Then the response is 200
And the input record is deleted from change_analysis_inputs
And the response body is { "success": true, "data": { "deleted": true, "id": "..." } }
```

#### M2-AC-31 — DELETE /analyses/:id/inputs/:inputId fails with 409 when analysis is not draft

```
Given an analysis in ready status with an input
When DELETE /api/change-intelligence/analyses/:id/inputs/:inputId is called
Then the response is 409
And the input record is not deleted
```

#### M2-AC-32 — DELETE /analyses/:id/inputs/:inputId returns 404 when input does not belong to analysis

```
Given inputId is a valid UUID that exists in change_analysis_inputs
But it belongs to a different analysis, not the one in the URL path
When DELETE /api/change-intelligence/analyses/:id/inputs/:inputId is called
Then the response is 404 (not 403)
And no record is deleted
```

#### M2-AC-33 — Migration creates both tables with all specified columns

```
Given 031_change_intelligence.sql has been applied
When the database schema is inspected
Then change_analyses exists with all 17 columns and check constraints on status and trigger_type
And change_analysis_inputs exists with all 9 columns and check constraint on input_type
And all 7 indexes are present
And no existing table has been modified
```

#### M2-AC-34 — Migration is idempotent

```
Given 031_change_intelligence.sql has already been applied
When the migration is applied a second time
Then no error is raised
And all tables, indexes, and constraints remain unchanged
```

#### M2-AC-35 — Deleting an analysis cascades to delete all its inputs

```
Given an analysis with three attached inputs
When the analysis record is deleted from change_analyses
Then all three input records are automatically deleted from change_analysis_inputs
And no orphaned input records remain
```

#### M2-AC-36 — Posting a second pr_diff to a draft analysis replaces the existing one

```
Given a draft analysis that already has one input with input_type 'pr_diff'
When POST /api/change-intelligence/analyses/:id/inputs is called
  with another input of input_type 'pr_diff' and new content
Then the existing pr_diff input is replaced (not duplicated)
And the analysis has exactly one input with input_type 'pr_diff'
And the content_hash of the remaining input reflects the new content (canonicalized)
And the response is 200 (not 201, since the input was replaced)
```

#### M2-AC-37 — Auto-generated title is non-null when no title is provided

```
Given a POST /api/change-intelligence/analyses request with no title field
When the analysis is created
Then the response title is a non-null string
And the format matches "Change Analysis — [Month Day, Year H:MM AM/PM]"
And the stored change_analyses record has a NOT NULL title value
```

#### M2-AC-38 — Title cannot be cleared via PATCH

```
Given a draft analysis with a title
When PATCH /api/change-intelligence/analyses/:id is called with { "title": "" }
Then the response is 400
And the error body matches { "success": false, "error": "..." }
And the title in the database is unchanged
```

#### M2-AC-39 — createdBy filter narrows results without gating authorization

```
Given analyses exist created by user A and user B
When user A calls GET /api/change-intelligence/analyses?created_by=<user-A-uuid>
Then only user A's analyses are returned
When user A calls GET /api/change-intelligence/analyses?created_by=<user-B-uuid>
Then user B's analyses are returned (created_by filter is not an authorization boundary in M2)
```

#### M2-AC-40 — Secondary sort by id ensures deterministic ordering

```
Given two analyses have the same updated_at timestamp
When GET /api/change-intelligence/analyses is called
Then the two analyses appear in a consistent order (id DESC as secondary sort key)
And repeating the identical request returns the same order
```

#### M2-AC-41 — Default page size is 25

```
Given 30 analyses exist
When GET /api/change-intelligence/analyses is called with no limit parameter
Then the response contains 25 analyses
And data.pagination.total is 30
And data.pagination.limit is 25
```

#### M2-AC-42 — PR diff replacement is atomic: exactly one pr_diff remains

```
Given a draft analysis already has a pr_diff input
When POST /api/change-intelligence/analyses/:id/inputs is called with a new pr_diff
Then GET /api/change-intelligence/analyses/:id shows exactly one input with input_type 'pr_diff'
And no orphaned pr_diff rows exist in change_analysis_inputs for that analysis
```

#### M2-AC-43 — Multiple non-primary inputs of the same type are permitted

```
Given a draft analysis
When two POST /api/change-intelligence/analyses/:id/inputs requests are made
  each with { "input_type": "requirement_text", "content": "..." }
Then two separate requirement_text input records exist on the analysis
And GET /api/change-intelligence/analyses/:id shows both inputs
```

#### M2-AC-44 — Content hash canonicalization: CRLF normalized to LF

```
Given an input is created with content "Hello\r\nWorld" (CRLF line endings)
When the input is stored
Then the stored content_hash equals the SHA-256 of "Hello\nWorld" (LF after normalization)
And the hash is 64 lowercase hexadecimal characters with no prefix
```

#### M2-AC-45 — Content hash canonicalization: UTF-8 BOM removed

```
Given an input is created with content that begins with the UTF-8 BOM (EF BB BF) followed by "Hello"
When the input is stored
Then the stored content_hash equals the SHA-256 of "Hello" (BOM stripped before hashing)
And the hash is 64 lowercase hexadecimal characters
```

#### M2-AC-46 — Concurrent pr_diff creation returns 409, not 500

```
Given the database has a partial unique index UNIQUE (analysis_id) WHERE input_type = 'pr_diff'
And two concurrent POST /inputs requests with input_type 'pr_diff' are sent to the same draft analysis
Then exactly one succeeds (200 or 201)
And the other receives HTTP 409 Conflict
And the error body is { "success": false, "error": "An analysis can have at most one PR diff input. The diff was already added by a concurrent request." }
And the constraint name, SQL, driver details, or stack trace do not appear in the error body
And the analysis has exactly one pr_diff input after both requests complete
```

---

### Milestone 3 — Manual Input Analysis

#### M3-AC-01 — Analysis submits and produces output

```
Given a ready analysis with at least one pr_diff and one requirement input
When the user triggers the analysis
Then a change summary is displayed
And a list of extracted requirements is displayed
And each requirement shows evidence and confidence
And the analysis status transitions to completed
```

#### M3-AC-02 — Missing required inputs are rejected

```
Given an analysis with no pr_diff input
When the user attempts to trigger analysis
Then an error is shown
And the analysis is not processed
```

#### M3-AC-03 — AI failure is isolated

```
Given the Anthropic API returns an error during analysis
When the user is on the analysis page
Then an error message is shown on the Change Intelligence page
And the analysis status transitions to failed
And all other QA Center pages are unaffected
```

---

### Milestone 4 — Requirement Comparison

#### M4-AC-01 — Each requirement has a coverage status

```
Given an analysis is complete
When the user views the comparison section
Then every extracted requirement displays one of:
  implemented, partial, missing, ambiguous, unable_to_verify
And every status is supported by evidence
```

#### M4-AC-02 — Reviewer can override a status

```
Given a requirement has an AI-assigned coverage status
When the reviewer selects a different status and saves a note
Then the override status and note are saved
And the AI-assigned status is still visible for comparison
```

---

### Milestone 5 — Risk and Regression Analysis

#### M5-AC-01 — Risk findings are displayed with evidence

```
Given an analysis is complete
When the user views risk findings
Then each finding names the impacted area and risk category
And each finding shows supporting evidence from the diff
And each finding has a review status
```

#### M5-AC-02 — Reviewer can dispute a finding

```
Given a risk finding has review status "unreviewed"
When the reviewer marks it "disputed" and adds a note
Then the dispute is saved
And the finding is visually distinguished from acknowledged findings
```

---

### Milestone 6 — Manual Test Generation

#### M6-AC-01 — Generated cases require approval before import

Covered by BC-11.

#### M6-AC-02 — Approved case import matches existing library format

```
Given a generated test case has been approved
When the user imports it
Then it appears in the test-case library in the same format as manually created test cases
And it can be added to suites and included in test runs
And the import route calls requireAuth() explicitly
```

---

### Milestone 7 — Playwright Proposals

#### M7-AC-01 — Proposals are displayed as readable code

```
Given Playwright proposals have been generated
When the user views them
Then each proposal is displayed as a formatted code block
And the user can copy the code
And the framework is labeled "Playwright (TypeScript)"
```

#### M7-AC-02 — No code is executed or committed

```
Given any number of Playwright proposals have been generated and accepted
Then no code has been executed, saved to disk, committed to any repository,
  or submitted as a PR
```

---

### Milestone 13 — Pilot

#### M13-AC-01 — Non-pilot users are unaffected

```
Given the pilot group is enabled
When a non-pilot user logs in
Then no Change Intelligence functionality is visible or accessible
And their workflows are completely unaffected
```

---

## Definition of Done

A milestone is done when:

- [ ] All backward compatibility criteria (BC-01 through BC-15) pass
- [ ] All feature acceptance criteria for the milestone pass
- [ ] Unit tests exist for all new business logic
- [ ] API tests exist for all new endpoints
- [ ] Migration has been verified against the production schema (Milestone 2+)
- [ ] Regression smoke test checklist has been run and all items pass
- [ ] No critical or high bugs remain open
- [ ] Documentation updated in this repository
