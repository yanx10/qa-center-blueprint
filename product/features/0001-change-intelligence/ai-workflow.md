# AI Workflow — Change Intelligence

## Overview

Change Intelligence uses Claude to compare a PR diff against one or more requirement sources and produce a structured analysis covering requirement coverage, risk, test-case proposals, and Playwright automation proposals. AI is required because no deterministic algorithm can reliably extract requirements from unstructured text, map those requirements to implementation evidence in a diff, classify coverage status, identify regression risk, or generate targeted test cases without semantic understanding.

All AI calls in Change Intelligence use the existing shared wrapper at `lib/ai/anthropic.ts`. No new AI infrastructure is introduced in Phase 1.

---

## Confirmed Constraints

These constraints were confirmed by Milestone 0 inspection and govern all AI work in Phase 1:

- **Synchronous execution.** Every AI call is a blocking HTTP request to the Anthropic API. The application waits for the full response before returning. Duration is typically 15–40 seconds for structured generation tasks. No streaming or polling is available.
- **No automatic retry.** `generateStructuredAIResponse<T>()` does not retry on failure. Failures throw with `.rawText` attached for debugging. API routes must handle these explicitly.
- **No streaming.** The current SDK integration is non-streaming. The `AIGenerationContext` in `app/layout.tsx` allows the user to navigate away while an in-flight request completes, but the server-side call still blocks.
- **Single entry point.** `generateStructuredAIResponse<T>({ systemPrompt, userPrompt, maxTokens })` is the only AI entry point. All Change Intelligence calls must use it without modification.
- **Model and key selection.** The model resolves via: database `integration_settings` → `ANTHROPIC_MODEL` env var → `claude-sonnet-4-6`. The API key resolves via: database `integration_settings` → `ANTHROPIC_API_KEY`. Change Intelligence inherits this chain automatically.

---

## Analysis Stages

A Change Intelligence analysis proceeds through the following conceptual stages. These are stages of the analysis workflow, not necessarily separate network calls to Claude. Implementation may combine stages into a single prompt or separate them based on token budget and output clarity.

### Stage 1 — Input Validation

Performed server-side before any AI call.

- Verify that at least one `pr_diff` input is present.
- Verify that at least one requirement source input is present.
- Estimate combined token count of all inputs.
- If inputs exceed `MAX_DIFF_TOKENS` (to be defined — see Pre-M2 Requirements), apply truncation or return a 400 with a descriptive message.

### Stage 2 — Requirement Extraction

Claude reads all requirement source inputs (PRD text, Jira story, acceptance criteria, Markdown spec, or pasted text) and produces a structured list of discrete, testable requirements.

Output per requirement:
- `requirement_text` — the extracted requirement statement
- `source_type` — which input type it came from
- `ordinal` — display order

### Stage 3 — Diff Summarization

Claude reads the PR diff and produces a structured change summary.

Output:
- `change_summary` — a concise prose summary of what the change does
- Key implementation areas touched (file paths, function names, components)

### Stage 4 — Requirement-to-Implementation Mapping

Claude compares the extracted requirements against the diff evidence and assigns a coverage status to each requirement.

Coverage statuses:
- `implemented` — the diff contains clear evidence that the requirement is satisfied
- `partial` — the diff contains partial evidence; some aspects of the requirement are covered
- `missing` — no evidence in the diff that this requirement was addressed
- `ambiguous` — the diff is ambiguous and could or could not satisfy the requirement
- `unable_to_verify` — insufficient runtime, repository, or contextual information to reach a conclusion

Output per requirement:
- `coverage_status`
- `evidence` — specific diff excerpt or reasoning
- `confidence` — `high`, `medium`, or `low`

### Stage 5 — Risk and Regression Analysis

Claude identifies risk findings: areas where the change could cause regression, data integrity issues, security concerns, or unexpected behavior.

Output per finding:
- `risk_category` — e.g., `data_integrity`, `auth`, `performance`, `ui_regression`, `unexpected_behavior`
- `impacted_area` — product area or component affected
- `description` — description of the risk
- `evidence` — diff excerpt or reasoning
- `confidence`

### Stage 6 — Test-Case Proposal Generation

Claude generates proposed manual test cases targeting the change, prioritizing:
- Coverage gaps (requirements with `missing` or `partial` status)
- Risk areas (high-confidence risk findings)
- Happy-path and edge-case variations

Output per proposal:
- `title`
- `steps` — JSONB in the same shape as `test_steps` rows: `[{step_order, description, expected_result}]`
- `expected_result`
- `requirement_id` — which requirement this test addresses (when applicable)

### Stage 7 — Playwright Proposal Generation

Claude generates Playwright (TypeScript) automation proposals for suitable test cases, prioritizing automatable scenarios identified in Stage 6.

Output per proposal:
- `description` — what the proposed test covers
- `code` — Playwright TypeScript code as a text string
- `requirement_id` — which requirement this tests (when applicable)

No generated code is executed, saved to disk, committed, or submitted as a PR.

### Stage 8 — Human Review

All AI outputs require explicit human review before any action is taken:
- Requirement coverage statuses can be overridden by a reviewer.
- Risk findings can be acknowledged or disputed.
- Generated test-case proposals require per-case approval before import.
- Playwright proposals require per-proposal acceptance, rejection, or deferral.

No AI output affects existing data without a human action.

---

## Prompt Strategy

All prompts follow the existing two-part structure:
- `systemPrompt` — role definition, output format specification, JSON schema for the expected response
- `userPrompt` — the actual user content (diff text, requirement text, etc.)

Prompt files live in `lib/ai/prompts/change-intelligence.ts`, following the pattern established by `lib/ai/prompts/prd-test-generation.ts`.

Prompts use structured output: the system prompt specifies a JSON schema, and `generateStructuredAIResponse<T>()` extracts and validates the JSON from the response. The function handles markdown code fences, preamble text, and trailing noise.

**Prompt versioning:** Each analysis row stores a `workflow_version` string (e.g., `"m2-v1"`). When prompts change, the version is bumped. This allows correlation of output quality with prompt versions during pilot analysis.

**Tone and instruction:** All prompts instruct Claude to:
- Cite specific diff excerpts or requirement text as evidence for every conclusion.
- Use `unable_to_verify` when evidence is insufficient — not a fabricated determination.
- Express confidence (`high`, `medium`, `low`) for each finding.

---

## Model Selection

| Task | Model |
|------|-------|
| All Change Intelligence AI calls | Resolves via: DB `integration_settings.anthropic_model` → `ANTHROPIC_MODEL` env → `claude-sonnet-4-6` |

Change Intelligence does not override model selection. The same model used for AI Studio generation is used for Change Intelligence analysis. This ensures consistency and avoids introducing a separate model configuration path.

---

## Human-in-the-Loop

| Point | Decision Required | Action if Not Approved |
|-------|-----------------|----------------------|
| Requirement coverage override | Reviewer may override AI-assigned status and note a reason | AI status is preserved alongside the override for comparison |
| Risk finding review | Reviewer must acknowledge or dispute each finding | Findings default to `unreviewed`; no automatic action is taken |
| Test-case proposal | Reviewer must approve each proposal before import | Proposal remains in `change_generated_test_cases` at `pending` status |
| Test-case import | Reviewer initiates explicit import action after approval | Approved cases are not automatically imported |
| Playwright proposal | Reviewer must accept, reject, or defer each proposal | Proposals are display-only; no code is executed or saved |

No AI-generated result enters a permanent table or is acted upon without an explicit human decision.

---

## Failure Modes

| Failure | Detection | Recovery |
|---------|-----------|---------|
| Anthropic API timeout | HTTP error or `generateStructuredAIResponse` throws | Return 500 with human-readable error; analysis saved as `status='error'`; no partial data committed |
| JSON parse failure | `extractJsonObject` throws with `.rawText` attached | Return 500 with partial raw text for debugging; log `[change-intelligence-api]` error |
| Oversized input (exceeds token limit) | Server-side token estimation before AI call | Return 400 with descriptive message; prompt user to reduce diff or requirement size |
| Structured response missing required fields | Zod or type validation fails | Return 500; log full raw response for debugging |
| AI returns `unable_to_verify` for all requirements | Normal output — not a failure | Display correctly; user can provide additional context and resubmit |
| Feature flag disabled during in-flight request | Flag check in API route handler | Return 403; existing requests in progress are unaffected |

All Change Intelligence failures are isolated. A failure in any Change Intelligence route must not affect any existing QA Center page or API.

---

## Pre-Milestone 2 Requirements

The following must be defined in the blueprint before Milestone 2 implementation begins:

- [ ] Maximum accepted diff character count and estimated token equivalent
- [ ] Maximum accepted total requirement source character count
- [ ] Token estimation approach (character-based heuristic or exact tokenizer)
- [ ] Behavior when inputs are oversized: truncate-with-warning, reject-with-400, or chunk
- [ ] Whether chunking is implemented as multiple sequential AI calls or a single call with a summarized diff
- [ ] Request timeout limit for the server-side AI call
- [ ] Behavior when partial stages complete before a failure (e.g., requirements extracted but risk analysis fails)
- [ ] Structured-response validation library and approach (Zod on server, or manual checks)

---

## Evaluation

AI output quality will be measured during the Milestone 9 pilot using the following metrics:

| Metric | Target | Evaluation Method |
|--------|--------|------------------|
| Requirement extraction accuracy | [OPEN — baseline from pilot] | Reviewer manually verifies extracted requirements against source |
| Coverage status accuracy | [OPEN — baseline from pilot] | Reviewer override rate; false-positive and false-negative counts |
| Risk finding quality | [OPEN — baseline from pilot] | Finding dispute rate in pilot data |
| Generated test case acceptance rate | [OPEN — baseline from pilot] | Approved vs. rejected counts from pilot data |
| Playwright proposal acceptance rate | [OPEN — baseline from pilot] | Accepted vs. rejected counts from pilot data |
| `unable_to_verify` rate | [OPEN — establish acceptable range] | Count per analysis |

Baselines and targets will be established from Milestone 9 pilot data. Values are not fabricated.
