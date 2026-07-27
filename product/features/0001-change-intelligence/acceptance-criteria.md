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

### Milestone 2 — Manual Input Analysis

#### M2-AC-01 — Analysis submits and produces output

```
Given a user provides at least one requirement source and one PR diff
When they submit the analysis form
Then a change summary is displayed
And a list of extracted requirements is displayed
And each requirement shows evidence and confidence
And the analysis is saved as a record accessible via GET /api/change-intelligence/analyses/:id
```

#### M2-AC-02 — Missing required inputs are rejected

```
Given a user submits the form without a PR diff
Or without any requirement source
When the request reaches the API
Then a 400 response is returned with a descriptive error
And no analysis record is created
```

#### M2-AC-03 — AI failure is isolated

```
Given the Anthropic API returns an error during analysis
When the user is on the analysis submission page
Then an error message is shown on the Change Intelligence page
And all other QA Center pages are unaffected
```

---

### Milestone 3 — Requirement Comparison

#### M3-AC-01 — Each requirement has a coverage status

```
Given an analysis is complete
When the user views the comparison section
Then every extracted requirement displays one of:
  implemented, partial, missing, ambiguous, unable_to_verify
And every status is supported by evidence
```

#### M3-AC-02 — Reviewer can override a status

```
Given a requirement has an AI-assigned coverage status
When the reviewer selects a different status and saves a note
Then the override status and note are saved
And the AI-assigned status is still visible for comparison
```

---

### Milestone 4 — Risk and Regression Analysis

#### M4-AC-01 — Risk findings are displayed with evidence

```
Given an analysis is complete
When the user views risk findings
Then each finding names the impacted area and risk category
And each finding shows supporting evidence from the diff
And each finding has a review status
```

#### M4-AC-02 — Reviewer can dispute a finding

```
Given a risk finding has review status "unreviewed"
When the reviewer marks it "disputed" and adds a note
Then the dispute is saved
And the finding is visually distinguished from acknowledged findings
```

---

### Milestone 5 — Manual Test Generation

#### M5-AC-01 — Generated cases require approval before import

Covered by BC-11.

#### M5-AC-02 — Approved case import matches existing library format

```
Given a generated test case has been approved
When the user imports it
Then it appears in the test-case library in the same format as manually created test cases
And it can be added to suites and included in test runs
And the import route calls requireAuth() explicitly
```

---

### Milestone 6 — Playwright Proposals

#### M6-AC-01 — Proposals are displayed as readable code

```
Given Playwright proposals have been generated
When the user views them
Then each proposal is displayed as a formatted code block
And the user can copy the code
And the framework is labeled "Playwright (TypeScript)"
```

#### M6-AC-02 — No code is executed or committed

```
Given any number of Playwright proposals have been generated and accepted
Then no code has been executed, saved to disk, committed to any repository,
  or submitted as a PR
```

---

### Milestone 9 — Pilot

#### M9-AC-01 — Non-pilot users are unaffected

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
