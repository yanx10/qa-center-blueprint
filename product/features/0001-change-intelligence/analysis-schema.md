# Analysis Schema — QA Intelligence Report

## Overview

This document is the canonical definition of the JSON structure stored in `change_analyses.analysis_json`. Every completed M3 analysis produces an object conforming to this schema. The schema is versioned; all M3 analyses use schema version `"1.0.0"`.

The schema is validated at runtime by the output parser before any analysis is marked `completed`. An analysis that returns a structurally invalid response is marked `failed` with `error_code = 'SCHEMA_VALIDATION_FAILED'` and `analysis_json` remains null.

---

## Schema Version

| Field | Value |
|-------|-------|
| `schema_version` | `"1.0.0"` |
| Introduced in | M3 |
| Prompt version | `m3-v1` |
| Validation timing | After AI response, before DB write |

---

## Top-Level Structure

All top-level fields are required. Arrays may be empty (`[]`) except `recommended_test_focus` (minimum one item).

```
{
  schema_version:              "1.0.0"
  executive_summary:           object (required)
  change_summary:              object (required)
  risk_assessment:             object (required)
  impacted_components:         array  (required, may be empty)
  regression_recommendations:  array  (required, may be empty)
  high_risk_scenarios:         array  (required, may be empty)
  missing_requirements:        array  (required, may be empty)
  ambiguities:                 array  (required, may be empty)
  edge_cases:                  array  (required, may be empty)
  recommended_test_focus:      array  (required, min 1 item)
  confidence:                  object (required)
  metadata:                    object (required)
}
```

---

## Field Reference

### `schema_version`

Type: `string`. Required. Value must be exactly `"1.0.0"`.

---

### `executive_summary`

High-level summary for fast consumption. Always rendered as the first section in the UI.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `headline` | `string` | Yes | One sentence summary. Stored in `change_analyses.change_summary`. |
| `risk_level` | `enum` | Yes | `low`, `medium`, `high`, or `critical`. Must equal `risk_assessment.overall_level`. |
| `primary_concern` | `string` | Yes | The single most significant gap or risk. Stored in `change_analyses.requirement_summary`. |
| `recommended_action` | `string` | Yes | One clear action the QA lead should take. |

---

### `change_summary`

Characterization of what the change does.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `description` | `string` | Yes | Prose description of the PR change. |
| `change_type` | `string[]` | Yes | At least one value. Allowed: `feature`, `bugfix`, `refactor`, `performance`, `security`, `config`, `dependency`. |
| `scope` | `enum` | Yes | `isolated`, `moderate`, `broad`, or `cross-cutting`. |
| `key_areas_modified` | `string[]` | Yes | Product areas or components affected. May be empty. |

**`scope` values:**

| Value | Meaning |
|-------|---------|
| `isolated` | Confined to a single component or function |
| `moderate` | Affects a few related components |
| `broad` | Affects many components or a significant subsystem |
| `cross-cutting` | Affects shared infrastructure used throughout the codebase |

---

### `risk_assessment`

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `overall_level` | `enum` | Yes | Must equal `executive_summary.risk_level`. Allowed: `low`, `medium`, `high`, `critical`. |
| `rationale` | `string` | Yes | Explanation of the overall risk determination. |
| `risk_factors` | `array` | Yes | Zero or more items. |

Each `risk_factors` item: `{ "factor": string, "impact": "low"|"medium"|"high", "explanation": string }`.

---

### `impacted_components`

Each item: `{ "name": string, "impact_type": "direct"|"indirect", "description": string }`.

May be empty if the change is too small to identify distinct components.

---

### `regression_recommendations`

Each item: `{ "area": string, "reason": string, "priority": "low"|"medium"|"high"|"critical" }`.

May be empty if no regression risk is identified.

---

### `high_risk_scenarios`

Each item: `{ "scenario": string, "likelihood": "low"|"medium"|"high", "impact": "low"|"medium"|"high", "mitigation": string }`.

May be empty if no high-risk scenarios are identified.

---

### `missing_requirements`

Each item: `{ "description": string, "evidence": string, "severity": "minor"|"moderate"|"major"|"blocking" }`.

May be empty if all requirements appear to be addressed.

---

### `ambiguities`

Each item: `{ "description": string, "context": string, "recommendation": string }`.

May be empty if no ambiguities are identified.

---

### `edge_cases`

Each item: `{ "scenario": string, "risk": "low"|"medium"|"high", "testing_guidance": string }`.

May be empty if no edge cases are identified.

---

### `recommended_test_focus`

Each item: `{ "area": string, "rationale": string, "priority": "low"|"medium"|"high"|"critical", "suggested_scenarios": string[] }`.

**At least one item is required.** An empty array fails schema validation.

---

### `confidence`

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `overall` | `enum` | Yes | `low`, `medium`, or `high`. |
| `diff_completeness` | `enum` | Yes | How complete the PR diff appeared to be. Same allowed values. |
| `requirement_completeness` | `enum` | Yes | How complete the requirement documentation appeared to be. Same allowed values. |
| `context_sufficiency` | `enum` | Yes | Whether sufficient context was available to make reliable assessments. Same allowed values. |
| `limitations` | `string[]` | Yes | Specific limitations. May be empty. |

---

### `metadata`

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `input_types_used` | `string[]` | Yes | Input types included in the prompt (e.g., `["pr_diff", "acceptance_criteria"]`). |
| `input_count` | `integer` | Yes | Total number of input documents. Non-negative. |
| `analysis_notes` | `string or null` | Yes | Notes about the analysis process, or null. |

---

## Validation Rules

Any violation causes `error_code = 'SCHEMA_VALIDATION_FAILED'` and `analysis_json` remains null.

1. `schema_version` must equal `"1.0.0"`.
2. All required top-level fields must be present.
3. All enum fields must contain only allowed values.
4. `recommended_test_focus` must have at least one item.
5. `risk_assessment.overall_level` must equal `executive_summary.risk_level`.
6. All array items must contain all required sub-fields.
7. `metadata.input_count` must be a non-negative integer.

---

## Schema Evolution

When the prompt changes in a way that alters the output structure, the schema version is incremented and a new prompt version is released.

| Change Type | Action |
|-------------|--------|
| New optional field added | Bump minor version (e.g., `1.0.0` to `1.1.0`); validator accepts both |
| Required field added or enum value added | Bump minor version; update validator |
| Field renamed or removed | Bump major version (e.g., `1.x.x` to `2.0.0`); document migration |
| Prompt reworded with no structural change | No schema version change; bump prompt version only |

Old analyses retain their original `schema_version`. The UI must handle all schema versions that appear in historical records.

---

## Abbreviated JSON Example

The following is a structurally complete but abbreviated example. See `milestones/m3-ai-change-analysis.md` Section 10 for a full annotated example.

```json
{
  "schema_version": "1.0.0",
  "executive_summary": {
    "headline": "Medium-risk change adding email notification preference — unsubscribe flow requires validation",
    "risk_level": "medium",
    "primary_concern": "Unsubscribe link behavior when user has no email address is not addressed in requirements",
    "recommended_action": "Verify preference persistence across sessions and test all unsubscribe edge cases"
  },
  "change_summary": {
    "description": "Adds optional email notification preference toggle to user settings.",
    "change_type": ["feature"],
    "scope": "moderate",
    "key_areas_modified": ["User Settings", "Email Dispatch Worker"]
  },
  "risk_assessment": {
    "overall_level": "medium",
    "rationale": "Touches user data persistence and async worker. Unsubscribe edge case elevates risk.",
    "risk_factors": [
      {
        "factor": "Email worker dependency",
        "impact": "medium",
        "explanation": "Worker reads preference column; if migration not applied, all users receive emails."
      }
    ]
  },
  "impacted_components": [
    { "name": "User Settings Page", "impact_type": "direct", "description": "New toggle added." },
    { "name": "Email Dispatch Worker", "impact_type": "direct", "description": "Reads new preference field." }
  ],
  "regression_recommendations": [
    { "area": "User Settings — existing preferences", "reason": "Form refactor may affect other toggles.", "priority": "high" }
  ],
  "high_risk_scenarios": [
    { "scenario": "Worker reads preference before migration applied", "likelihood": "low", "impact": "high", "mitigation": "Default NULL to opted-out." }
  ],
  "missing_requirements": [
    { "description": "Unsubscribe behavior for users with no email", "evidence": "Endpoint added but NULL email case not specified.", "severity": "major" }
  ],
  "ambiguities": [
    { "description": "Default preference for existing users", "context": "Migration adds column but requirements do not specify default.", "recommendation": "Confirm with product: opted-in or opted-out?" }
  ],
  "edge_cases": [
    { "scenario": "User with no verified email enables notification preference", "risk": "medium", "testing_guidance": "Attempt without verified email; verify system blocks or explains requirement." }
  ],
  "recommended_test_focus": [
    {
      "area": "Email preference persistence",
      "rationale": "Core feature — must survive reload and re-login.",
      "priority": "critical",
      "suggested_scenarios": ["Enable preference, reload, verify still enabled", "Enable, logout, login, verify still enabled"]
    }
  ],
  "confidence": {
    "overall": "medium",
    "diff_completeness": "high",
    "requirement_completeness": "medium",
    "context_sufficiency": "medium",
    "limitations": ["Worker full implementation not in diff", "No test files included"]
  },
  "metadata": {
    "input_types_used": ["pr_diff", "acceptance_criteria"],
    "input_count": 2,
    "analysis_notes": null
  }
}
```
