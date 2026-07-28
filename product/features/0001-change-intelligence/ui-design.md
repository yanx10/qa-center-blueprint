# UI Design — Change Intelligence M3

## Overview

This document specifies the UI design for Milestone 3 — AI Change Analysis. It covers the analysis detail page states for each lifecycle status, the QA Intelligence Report layout and section ordering, the failed state error mapping, and accessibility requirements.

M3 does not introduce new pages. It extends the analysis detail page introduced in M2 to handle the new lifecycle states (processing, completed, failed) and to render the QA Intelligence Report when the analysis is complete.

Related documents: `user-experience.md` (personas, overall journey), `analysis-schema.md` (report structure), `milestones/m3-ai-change-analysis.md` (full specification).

---

## Analysis Detail Page — State Overview

The analysis detail page at `/change-intelligence/analyses/[id]` renders differently depending on `analysis.status`. The state determines which actions are available and what content is shown.

| Status | Primary Content | Primary Action |
|--------|-----------------|---------------|
| `draft` | Input list, Edit inputs | Mark Ready, Cancel |
| `ready` | Input list (read-only) | Generate QA Intelligence |
| `processing` | Loading indicator, elapsed timer | None (no cancel in M3) |
| `completed` | QA Intelligence Report + metadata card | None (report is terminal) |
| `failed` | Error message, retry count | Retry Analysis (if retries remain) |
| `cancelled` | Input summary (read-only) | None |

---

## State: `ready`

The analysis has been marked ready by the user. Inputs are locked. No AI processing has started.

**Content:**
- Header with analysis title, status badge ("Ready"), and metadata (created by, date)
- Read-only input list — each input card shows type, source label, hash, and date; no content text
- Readiness summary — e.g., "1 PR diff, 2 requirement documents"

**Actions:**
- **"Generate QA Intelligence"** — primary button (filled). Clicking triggers `POST /generate` and transitions the page to processing state.
- **"Cancel"** — secondary link. Clicking triggers `POST /cancel` and transitions to cancelled state.

**Error handling:** If `POST /generate` returns 409 (race condition — analysis already generated), reload the page to show the current state.

---

## State: `processing`

The `POST /generate` request is in flight. The server is executing the AI call.

**Content:**
- Animated loading indicator (spinner or progress animation)
- Message: "Generating QA Intelligence…"
- Elapsed timer showing seconds elapsed since generation started (client-side timer, resets on mount)

**Actions:**
- None. The cancel button is not shown. The generate button is not shown. (M3 limitation — D-033)

**UX note:** The user may navigate away. The server continues the AI call regardless of client connection state. When the user returns to the page, the page loads with the then-current status (completed or failed). The elapsed timer is not persisted.

---

## State: `completed`

The AI analysis succeeded. `analysis_json` is populated.

**Content:**
- Header with title, status badge ("Completed" in green), and metadata
- **QA Intelligence Report** (see Report Layout section below)
- **Metadata card** (see Metadata Card section below)

**Actions:**
- None. A completed analysis is a terminal read-only state. The user reads the report and acts on it externally.

---

## State: `failed`

The AI analysis failed. `error_code` and `error_message` are set. `analysis_json` is null.

**Content:**
- Header with title, status badge ("Failed" in red), and metadata
- **Error panel** showing a human-readable explanation of the failure (see Error Message Mapping below)
- Retry count indicator: "Attempt {retry_count} of 3" (e.g., "Attempt 1 of 3")

**Actions:**
- **"Retry Analysis"** — shown when `retry_count < 3`. Clicking triggers `POST /retry`.
- When `retry_count = 3`: the retry button is not shown. A message reads: "Maximum retry attempts reached. Create a new analysis to try again."

**UX note:** After clicking Retry, the page transitions to processing state while the retry request is in flight.

---

## State: `cancelled`

The user cancelled the analysis before generation.

**Content:**
- Header with title, status badge ("Cancelled" in gray), and metadata
- Read-only input summary
- Informational message: "This analysis was cancelled before processing."

**Actions:**
- **"Delete"** — if user wants to clean up. Links to the delete confirmation flow from M2.

---

## QA Intelligence Report Layout

The report is rendered in a fixed section order, always top-to-bottom:

| Order | Section | Always Shown? | Source Field |
|-------|---------|--------------|-------------|
| 1 | Executive Summary | Yes | `executive_summary` |
| 2 | Change Summary | Yes | `change_summary` |
| 3 | Risk Assessment | Yes | `risk_assessment` |
| 4 | Impacted Components | Only if non-empty | `impacted_components` |
| 5 | Regression Recommendations | Only if non-empty | `regression_recommendations` |
| 6 | High Risk Scenarios | Only if non-empty | `high_risk_scenarios` |
| 7 | Missing Requirements | Only if non-empty | `missing_requirements` |
| 8 | Ambiguities | Only if non-empty | `ambiguities` |
| 9 | Edge Cases | Only if non-empty | `edge_cases` |
| 10 | Recommended Test Focus | Yes | `recommended_test_focus` |
| 11 | Confidence and Limitations | Yes | `confidence` |

Sections 4–9 are hidden when the corresponding array is empty. No "None found" placeholder is shown for hidden sections.

---

## Section Rendering Details

### Executive Summary

Rendered as a prominent summary card at the top of the report.

- **Risk level badge** — colored chip: low=green, medium=yellow, high=orange, critical=red
  - Must include an `aria-label` attribute: e.g., `aria-label="Risk level: High"` (not color-only)
- **Headline** — large bold text (the one-sentence summary)
- **Primary concern** — body text below the headline
- **Recommended action** — highlighted call-to-action text (e.g., in a bordered callout)

### Change Summary

- **Description** — prose paragraph
- **Change type pills** — one chip per `change_type` value (e.g., "Feature", "Refactor")
- **Scope badge** — single chip with scope value (e.g., "Moderate")
- **Key areas modified** — comma-separated list or tags

### Risk Assessment

- **Overall risk level** — same colored badge as Executive Summary
- **Rationale** — prose paragraph
- **Risk factors** — rendered as a table or card list:
  | Factor | Impact | Explanation |
  |--------|--------|-------------|
  | [factor] | [impact badge] | [explanation] |

### Impacted Components

Rendered as a two-section list: "Directly Impacted" and "Indirectly Impacted".

Each item shows the component name and a one-line description. If all components are the same type, the section label is omitted.

### Regression Recommendations, High Risk Scenarios, Missing Requirements, Ambiguities, Edge Cases

Each section renders as a card list. Cards are sorted by their priority/risk/severity field (highest first).

Card layout:
- **Title** — the primary description field (`area`, `scenario`, `description`)
- **Badge** — priority, risk, or severity (colored by level)
- **Detail** — reason, mitigation, recommendation, or testing guidance

### Recommended Test Focus

Rendered as an ordered list of focus area cards, sorted by priority (critical first, then high, medium, low).

Each card:
- **Area** — section title
- **Priority badge** — critical/high/medium/low
- **Rationale** — one-paragraph explanation
- **Suggested scenarios** — bulleted list of specific test scenario descriptions

### Confidence and Limitations

Rendered as a four-field summary with a limitations list.

```
Overall: [badge]   Diff: [badge]   Requirements: [badge]   Context: [badge]

Limitations:
• [limitation 1]
• [limitation 2]
```

The four badges use the same low/medium/high coloring.

---

## Metadata Card

Displayed alongside or below the Executive Summary for completed analyses.

| Field | Source | Display |
|-------|--------|---------|
| Provider | `change_analyses.provider` | "Anthropic" |
| Model | `change_analyses.ai_model` | e.g., "claude-sonnet-4-6" |
| Prompt version | `change_analyses.analysis_version` | e.g., "m3-v1" |
| Input tokens | `change_analyses.input_tokens` | Formatted number, e.g., "12,450" |
| Output tokens | `change_analyses.output_tokens` | Formatted number, e.g., "3,280" |
| Processing time | `change_analyses.processing_ms` | Formatted duration, e.g., "34.2 s" |
| Confidence | `analysis_json.confidence.overall` | Badge: low/medium/high |

The metadata card is secondary information — it should not compete visually with the report content. Render as a collapsed or muted card that can be expanded.

---

## Error Message Mapping

When an analysis is in `failed` status, show a human-readable explanation based on `error_code`. Never show `error_message` directly to the user.

| `error_code` | UI Message |
|-------------|-----------|
| `PROVIDER_TIMEOUT` | "The AI provider took too long to respond. This usually resolves on retry." |
| `PROVIDER_ERROR` | "The AI provider returned an error. Try again — if the problem persists, contact support." |
| `INVALID_OUTPUT` | "The AI response was not in the expected format. Retry to attempt again." |
| `SCHEMA_VALIDATION_FAILED` | "The AI response did not match the expected structure. This may indicate a prompt issue — contact support if it persists after retrying." |
| `INPUT_TOO_LARGE` | "The combined inputs exceed the maximum size limit. Reduce the size of your PR diff or requirement documents and try again." |
| `DB_WRITE_FAILED` | "The analysis completed but the result could not be saved. Contact support." |
| `UNKNOWN_ERROR` | "An unexpected error occurred. Try again — if the problem persists, contact support." |

---

## Accessibility Requirements

- **Risk level badges** must use both color and text. The badge label is the text (e.g., "High"); the `aria-label` confirms the meaning (e.g., `aria-label="Risk level: High"`). Do not rely on color alone.
- **Priority badges** must use both color and text labels (e.g., "Critical", "High", "Medium", "Low").
- **Loading state** must have an `aria-live="polite"` region announcing "Generating QA Intelligence…" to screen readers.
- **Report sections** must use semantic heading hierarchy: H2 for the report title, H3 for each section, H4 for sub-items.
- **Retry count** must be announced to screen readers when updated.
- **No horizontal scroll** at 1280px viewport on any device. All tables must be wrapped in `overflow-x: auto` containers.

---

## Analysis List Page — M3 Enhancements

The analysis list page at `/change-intelligence/analyses` shows one row per analysis. M3 adds two enhancements to completed analysis rows:

1. **Risk level badge** — rendered inline in the row using the same low/medium/high/critical color coding as the report. Source: `change_analyses.change_summary` combined with `risk_level` from the analysis object. The list endpoint does not return `analysis_json`, so the risk level badge requires a denormalized field.

   [DECISION] OQ-M3-UI-01 resolved: `risk_level` and `change_type_summary` are included in the `033_change_intelligence.sql` migration as nullable denormalized columns on `change_analyses`. Both are populated atomically with `change_summary` on successful completion (D-031). No `analysis_json` extraction is needed in list queries.

2. **"Completed" status pill** — replaces the generic status pill with a pill that includes the risk level badge inline for completed analyses.

---

## Open Questions

### OQ-M3-UI-01 — Risk level badge in the list view [RESOLVED]

**Resolution:** `risk_level text NULLABLE` and `change_type_summary text NULLABLE` are included in the `033_change_intelligence.sql` migration. Both are populated atomically with `change_summary` and `requirement_summary` on successful completion (D-031). The list endpoint returns these columns directly; no `analysis_json` extraction is needed.

**Resolved:** 2026-07-27

### OQ-M3-UI-02 — Should the elapsed timer be shown in the processing state?

**Why it matters:** A timer gives the user feedback that the system is working and isn't frozen. However, it creates a perception problem if the timer reaches 60+ seconds — users may assume the system has failed.

**Recommendation:** Show elapsed timer. Cap displayed time at "60+" seconds if elapsed exceeds 60 seconds. Add a note at 45+ seconds: "This is taking longer than usual. Please wait."

**Owner:** Implementation engineer. Non-blocking for M3 spec.
