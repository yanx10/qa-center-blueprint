# AGENTS.md — Instructions for AI Agents

This file provides behavioral guidelines for any AI agent (Claude, Codex, Gemini, or other) operating in this repository. Agents must read and comply with these rules before taking any action.

---

## Repository Identity

**Name:** QA Center Blueprint  
**Type:** Technical reference document — multi-volume Markdown book  
**Status:** Review-stage development  
**Canonical format:** Markdown (`.md`)

---

## Mandatory Rules

### Rule 1 — Preserve Existing Content

Do not delete, truncate, rewrite, or overwrite existing content unless the user instruction explicitly names the content to be removed or replaced.

When in doubt: **ask before changing.**

### Rule 2 — No Volume or Chapter Renumbering Without Approval

Volume identifiers (00–12) and chapter numbers are stable. They appear in cross-references, file names, and external citations. Renumbering any of them without explicit written approval from the repository owner is forbidden.

### Rule 3 — American English Only

All prose must use American English spelling, grammar, and vocabulary. Do not use British, Australian, or Canadian English variants.

### Rule 4 — Markdown Is the Only Editable Format

- Source content lives in `.md` files inside `docs/`, `templates/`, and `diagrams/`.
- Files in `generated/` are build artifacts. Agents must not create or modify files there.
- Do not create publishing toolchain config files (e.g., `mkdocs.yml`, `conf.py`, `book.toml`) unless explicitly instructed.

### Rule 5 — Diagrams Use Mermaid

All diagrams intended to be editable must be authored in Mermaid syntax inside fenced code blocks. Do not substitute a static image for a diagram that should be editable.

### Rule 6 — Generated Artifacts Stay in `generated/`

Build outputs (DOCX, PDF, HTML) belong exclusively in `generated/`. No generated content may appear in `docs/`, `templates/`, `diagrams/`, or the repository root.

### Rule 7 — Architectural Decisions Require ADRs

When a task involves an architectural choice (data model, integration pattern, platform component, major process change), create an ADR in `architecture/adr/` using the ADR template from `templates/`. Do not embed the decision solely in chapter prose.

### Rule 8 — Summarize Before Major Edits

If a task requires changes to more than one file, or substantive changes to any chapter, produce a clear summary of intended changes and wait for user confirmation before executing. Minor corrections (typos, formatting) may proceed without a summary.

### Rule 9 — Never Fabricate

Do not invent:
- Feature names or capability descriptions not present in existing content
- Implementation timelines or completion percentages
- Metric values, performance claims, or benchmark numbers
- Integration specifics not already documented

If information is missing, say it is missing. Use `[OPEN]` to mark unresolved questions.

### Rule 10 — Label Content Status

Use these labels consistently throughout all documents:

| Label | Use when |
|-------|----------|
| `[PROPOSAL]` | An idea has been raised but not decided |
| `[DECISION]` | A direction has been formally adopted |
| `[ASSUMPTION]` | A working assumption not yet validated |
| `[FUTURE]` | Scoped out of the current phase |
| `[OPEN]` | A question that still needs an answer |

---

## Safe Operations (No Confirmation Required)

- Reading any file in the repository
- Creating `.gitkeep` files in empty directories
- Fixing spelling, grammar, or formatting errors in existing content
- Adding new content to a file that already has a clear established structure
- Creating new files explicitly requested by the user

## Operations Requiring Confirmation

- Deleting any file
- Renaming or moving any file in `docs/`
- Changing volume or chapter numbering
- Modifying `CLAUDE.md` or `AGENTS.md`
- Adding or removing top-level directories
- Any action that cannot be easily reversed

---

## Content Standards

- Write in plain, direct prose. Prefer active voice.
- Avoid filler, hedging language, and vague qualifiers.
- Headings should be descriptive. Avoid puns or metaphors.
- Numbered lists for sequential steps; bullet lists for unordered items.
- Every section should be readable without requiring the reader to have read previous sections.

---

## Repository Map

```
docs/                        ← Markdown source — Volumes (edit here)
  00-foundation/
  01-product-vision/
  02-user-experience/
  03-ai-platform/
  04-ai-agents/
  05-memory/
  06-testing-platform/
  07-automation-platform/
  08-release-intelligence/
  09-platform-architecture/
  10-integrations/
  11-engineering/
  12-future-vision/
  appendix/
architecture/                ← Engineering source of truth (edit here)
  adr/                       ← Architecture Decision Records
  diagrams/                  ← Technical Mermaid diagrams
  decisions/                 ← Decision logs and rationale
  data-models/               ← Data model definitions
  api-design/                ← API contracts and specifications
  ui-design/                 ← UI design specs and component notes
  prompts/                   ← AI prompt library
templates/                   ← Reusable templates
diagrams/                    ← High-level Mermaid sources
assets/                      ← Images and branding
scripts/                     ← Build scripts
generated/                   ← DO NOT EDIT — build output only
  docx/
  pdf/
  html/
.github/
  workflows/
  ISSUE_TEMPLATE/
```

---

## Compliance

Failure to follow any rule in this file may result in content loss, broken cross-references, or misinformation in the published document. When a rule and a user instruction conflict, surface the conflict explicitly rather than silently choosing one over the other.
