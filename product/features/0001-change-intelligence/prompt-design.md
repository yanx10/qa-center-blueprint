# Prompt Design — Change Intelligence M3

## Overview

This document specifies the AI prompt design for Milestone 3. It covers the system prompt, user prompt template, input ordering strategy, token budget, output validation, and prompt versioning.

The prompt produces a structured JSON object conforming to `analysis-schema.md`. The output is validated before the analysis is marked `completed`.

---

## Prompt Version

| Field | Value |
|-------|-------|
| Version | `m3-v1` |
| Stored in | `change_analyses.analysis_version` |
| Location | `lib/ai/prompts/change-intelligence.ts` |
| Export | `CHANGE_INTELLIGENCE_PROMPT_VERSION = 'm3-v1'` |

When the prompt changes in a way that could alter the output structure or content, the version is incremented (e.g., `m3-v2`). Old analyses retain the version that produced them. See D-035.

---

## Input Limits

| Constant | Value | Rationale |
|----------|-------|-----------|
| `MAX_DIFF_CHARACTERS` | 100,000 | ~25,000 tokens; covers most real-world PR diffs |
| `MAX_REQUIREMENT_CHARACTERS` | 50,000 | ~12,500 tokens; covers comprehensive requirement docs |
| `MAX_TOTAL_INPUT_CHARACTERS` | 200,000 | ~50,000 tokens; leaves headroom for prompt overhead and output |

If combined inputs exceed `MAX_TOTAL_INPUT_CHARACTERS`, the analysis transitions to `failed` with `error_code = 'INPUT_TOO_LARGE'` before any AI call is made. Individual inputs are not truncated. The user must reduce input volume.

---

## Temperature

Fixed at `0.2`. Stored in `change_analyses.temperature`. Not user-configurable in M3.

QA analysis is evidence-based, not creative. Low temperature produces consistent, reproducible analysis. See D-034.

---

## Input Ordering

The prompt builder constructs inputs in this order:

1. **PR diff inputs** (`input_type = 'pr_diff'`)
2. **Requirement inputs** (in this order): `requirement_text`, `jira_story`, `acceptance_criteria`, `prd_text`, `markdown_spec`
3. **Supplemental context** (`input_type = 'supplemental_context'`)

Each input block is labeled with its type and, if present, its `source_label`:

```
=== PR DIFF ===
[content]

=== REQUIREMENT: Acceptance Criteria (Sprint 42 — Auth Refactor) ===
[content]

=== SUPPLEMENTAL CONTEXT ===
[content]
```

---

## System Prompt — `m3-v1`

```
You are a senior QA engineer performing structured analysis of software changes. Your role is to
analyze a pull request diff against the provided requirements and generate a comprehensive QA
Intelligence Report.

Your analysis must be:
- Evidence-based: every finding must reference specific content from the diff or requirements
- Honest about uncertainty: if you cannot determine something from the inputs, say so explicitly
- Comprehensive: cover risk, coverage gaps, regression targets, edge cases, and ambiguities
- Structured: return only valid JSON conforming to the schema provided, with no prose outside the JSON

You are NOT generating test cases, automation code, or test scripts. Your output is a structured
intelligence report for a QA lead to act on.

Analysis principles:
1. The diff is primary evidence. Requirements describe intent; the diff describes implementation.
2. When the diff contradicts the requirements, note it as a coverage gap or missing requirement.
3. When the requirements are silent on something the diff touches, note it as an ambiguity.
4. When the diff changes shared infrastructure (auth, middleware, database, shared services),
   escalate scope and risk accordingly.
5. A low-confidence finding based on limited evidence is still a valid finding — note the
   limitation in the confidence section.

Output format: Return a single JSON object. No markdown fences. No text before or after the JSON.
The object must conform exactly to the QA Intelligence Report schema (schema_version "1.0.0").

The recommended_test_focus array must have at least one item.
The executive_summary.risk_level must equal risk_assessment.overall_level.
All enum values must be from the exact sets defined in the schema.
```

---

## User Prompt Template — `m3-v1`

```
Analyze the following software change for QA impact.

{input_blocks}

Based on the above:
1. Summarize what this change does and its scope.
2. Assess the overall risk level (low, medium, high, critical) and explain your reasoning.
3. Identify which requirements are addressed, which are missing, and which are ambiguous.
4. List impacted components (direct and indirect).
5. Provide regression recommendations — what existing behavior should be regression-tested and why.
6. Identify high-risk failure scenarios with likelihood, impact, and mitigation.
7. Surface edge cases and boundary conditions.
8. Provide a prioritized test focus list with suggested test scenarios.
9. Assess your confidence in the analysis and list specific limitations.

Return the result as a single JSON object following the schema in your instructions.
```

The `{input_blocks}` placeholder is replaced with the assembled input sections constructed from the analysis inputs.

---

## Output Validation

After the AI call completes, the output undergoes two validation passes:

### Pass 1 — JSON Parse

Attempt to parse the raw response as JSON.

- Success: proceed to Pass 2.
- Failure: set `error_code = 'INVALID_OUTPUT'`. Do not store `analysis_json`.

### Pass 2 — Schema Validation

Validate the parsed JSON against the rules in `analysis-schema.md`:

- All required top-level fields present.
- All enum fields from allowed sets.
- `recommended_test_focus` has at least one item.
- `risk_assessment.overall_level` equals `executive_summary.risk_level`.
- All array items have all required sub-fields.
- `metadata.input_count` is a non-negative integer.

If any check fails:
- Set `error_code = 'SCHEMA_VALIDATION_FAILED'`.
- Log the specific validation failure server-side.
- Do not store `analysis_json`.

If all checks pass:
- Store `analysis_json`.
- Transition to `completed`.
- Populate `change_summary` and `requirement_summary` from `analysis_json.executive_summary`.

---

## Model Resolution

The model identifier is resolved in this order:

1. `integration_settings.anthropic_model` (DB setting, if present)
2. `ANTHROPIC_MODEL` environment variable (if set)
3. Default: `claude-sonnet-4-6`

The resolved model identifier is stored in `change_analyses.ai_model`.

---

## Token Budget Guidance

| Input Type | Expected Token Range | Notes |
|-----------|---------------------|-------|
| Small PR diff | 1,000–5,000 tokens | Bug fix, config change |
| Medium PR diff | 5,000–15,000 tokens | Feature addition, refactor |
| Large PR diff | 15,000–25,000 tokens | Major feature, broad refactor |
| Requirement docs | 2,000–12,500 tokens | Jira stories, acceptance criteria, PRD text |
| Output | 2,000–5,000 tokens | Structured JSON report |

The context window for `claude-sonnet-4-6` is sufficient for all inputs within the defined limits. No chunking is needed in M3.

---

## Prompt Versioning Policy

| Version | When to Bump |
|---------|-------------|
| `m3-v1` to `m3-v2` | Any wording change that could affect output content or structure |
| `m3-v2` to `m4-v1` | Milestone transition (M4 adds new output sections) |

When the prompt is bumped:
1. Update `CHANGE_INTELLIGENCE_PROMPT_VERSION` constant.
2. Update `analysis-schema.md` if the schema changes.
3. Document the change in `CHANGELOG.md`.
4. Run at least 3 end-to-end analyses against representative inputs to confirm output quality.

---

## Open Questions

### OQ-M3-P-01 — Should the system prompt include the full JSON schema or a condensed description?

**Why it matters:** Including the full schema improves output accuracy but consumes 1,500–2,000 tokens per call. A condensed description saves tokens at the cost of potential schema drift.

**Recommendation:** Include the full schema on initial deployment. If token costs become a concern, evaluate a condensed version in a future prompt iteration. Track schema conformance in production to detect drift.

**Owner:** Implementation engineer. Non-blocking for M3 spec.

### OQ-M3-P-02 — Should large diffs be truncated or should INPUT_TOO_LARGE be enforced strictly?

**Why it matters:** Strict enforcement means users with very large diffs must reduce input manually. Smart truncation could provide partial analysis.

**Recommendation:** Strict enforcement for M3. Truncation risks producing misleading partial analysis. Revisit if INPUT_TOO_LARGE is triggered frequently in the pilot.

**Owner:** Implementation engineer. Non-blocking for M3 spec.
