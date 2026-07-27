# User Experience — Change Intelligence

## Personas

### Primary

**Name:** Morgan  
**Role:** QA Engineer  
**Goal:** Quickly understand what a PR was supposed to do, verify what it actually changed, identify gaps in test coverage, and produce a focused set of tests before the PR merges.  
**Pain today:** Morgan reads the PR description, the linked Jira story, and the diff manually — across three separate tools — then writes test cases from scratch. The process takes 30–90 minutes per PR. Coverage gaps are discovered during testing or, worse, after release.

### Secondary

**Name:** Alex  
**Role:** QA Lead / Engineering Manager  
**Goal:** Ensure that PR reviews systematically capture requirement coverage and regression risk, without blocking development velocity.  
**Pain today:** No visibility into whether QA has checked a PR against its requirements. Post-release escapes are traced back to requirement-implementation mismatches that were never surfaced.

---

## User Journeys

### Journey 1 — PR Analysis (Milestone 2–6)

1. Morgan receives a PR for review. She opens QA Center and navigates to Change Intelligence.
2. She pastes the PR diff into the diff field and the Jira story text into the requirements field.
3. She submits the analysis. QA Center processes it (15–40 seconds) and displays:
   - A change summary explaining what the PR does
   - A list of extracted requirements with coverage status, evidence, and confidence
   - Risk findings with impacted areas and regression recommendations
   - Proposed test cases, each with approve / reject options
   - Playwright automation proposals, each with accept / reject / defer options
4. Morgan reviews each requirement status, overriding any she disagrees with and adding notes.
5. She approves the test cases she wants and triggers import into her chosen test suite.
6. She accepts one Playwright proposal and copies the code to add to the automation suite.

### Journey 2 — Milestone 1 Shell (Current Milestone)

1. An administrator enables `CHANGE_INTELLIGENCE_ENABLED` in the deployment environment.
2. Team members log in and see "Change Intelligence" in the navigation.
3. Clicking the nav item opens the Change Intelligence page, which shows:
   - The feature name and a concise description
   - An early-access indicator
   - A summary of the coming workflow
   - No analysis form, no fake results, no non-functional buttons
4. The team is aware the feature is coming. No existing workflows are affected.

---

## UX Principles

1. **Evidence before conclusion.** Every AI-generated status, finding, or recommendation shows its supporting evidence. Conclusions without evidence are not displayed.
2. **Human decision, AI suggestion.** Every AI output is a proposal. The interface makes it visually clear that each item requires a human decision before any action is taken.
3. **Explicit over implicit.** Import, approval, and acceptance are explicit user actions. Nothing moves to a permanent table automatically.
4. **Transparent uncertainty.** When Claude cannot determine a coverage status, the status is "unable to verify" with an explanation — not a guess. Confidence levels (`high`, `medium`, `low`) are always visible.
5. **Isolated failure.** If Change Intelligence fails (AI error, timeout, disabled flag), the rest of QA Center is completely unaffected. The error is contained to the Change Intelligence page.

---

## Milestone 1 Page Content

The Milestone 1 page at `/change-intelligence` is an isolated shell. It contains no analysis functionality, no database queries, and no AI calls.

**Required page elements:**

1. **Heading:** Change Intelligence

2. **Description:**
   > Compare product intent with implementation, identify risk, and generate focused test coverage.

3. **Early-access indicator:** A clear label or callout showing that this is an early feature area — not yet fully operational. Wording such as "Early Access" or "Coming Soon" is appropriate.

4. **Future workflow summary:** A brief prose description or simple list of what the feature will do when enabled:
   - Paste a PR diff and your requirements
   - Receive a structured requirement-to-implementation comparison
   - Identify risk areas and regression candidates
   - Review and import proposed test cases
   - Accept or defer Playwright automation proposals

**Prohibited in Milestone 1:**

- No analysis form or text areas
- No "Analyze" button (functional or non-functional) unless it is clearly labeled as disabled and non-interactive
- No fake or placeholder analysis results
- No suggestion that GitHub integration already exists
- No reference to specific AI analysis outputs that cannot be produced yet

---

## Key Screens

### Screen 1 — Feature Shell (Milestone 1)

- URL: `/change-intelligence`
- Component: Server component (`app/change-intelligence/page.tsx`)
- Gate: `notFound()` when `CHANGE_INTELLIGENCE_ENABLED` is falsy
- Auth gate: Middleware redirects unauthenticated users to login before the page renders

Content layout (prose description, not a wireframe):

```
[Page Header]
Change Intelligence

[Subheading / description]
Compare product intent with implementation, identify risk, and generate focused test coverage.

[Early Access Badge or Callout]
Early Access — Full analysis will be available in the next release.

[Coming Soon Panel — dashed border]
What's coming:
• Paste a PR diff and your requirements to start an analysis
• See which requirements are covered, partial, missing, or ambiguous
• Review risk findings and regression recommendations
• Approve and import proposed test cases
• Accept Playwright automation proposals

[No buttons, no form, no fake data]
```

### Screen 2 — Analysis Input (Milestone 2, not in M1)

To be designed when Milestone 2 begins.

### Screen 3 — Analysis Results (Milestone 2–6, not in M1)

To be designed when Milestone 2–3 begins.

---

## Edge Cases

| Scenario | Expected Behavior |
|----------|------------------|
| Feature flag disabled; user navigates directly to `/change-intelligence` | `notFound()` — server returns 404. No Change Intelligence content is revealed. |
| Feature flag disabled; API is called directly | API returns `{ "success": false, "error": "Change Intelligence is not enabled." }` with status 403 |
| Feature flag enabled; unauthenticated user navigates to `/change-intelligence` | Middleware redirects to `/login?callbackUrl=/change-intelligence` |
| Feature flag enabled; API is called without a valid session | `requireAuth()` returns 401 |
| AI call times out during analysis | Error message displayed on Change Intelligence page; no other page is affected |
| Analysis is submitted with no requirement source | Server returns 400 with descriptive error; no analysis record is created |
| Analysis is submitted with an oversized diff | Server returns 400 with explanation of size limit; no AI call is made |
| Approved test case is imported; import fails | Error shown on Change Intelligence page; existing test_cases table is unchanged |

---

## Accessibility

- All interactive controls must be keyboard-navigable.
- Status labels (`implemented`, `partial`, `missing`, `ambiguous`, `unable_to_verify`) must not rely on color alone. Use text labels and icons.
- Evidence text areas must be readable at 200% zoom without horizontal scrolling.
- The early-access indicator must not rely on color alone — it must include visible text.
- All form fields must have associated labels (not placeholder-only).
