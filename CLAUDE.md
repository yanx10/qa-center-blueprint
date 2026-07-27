# CLAUDE.md — Instructions for AI Assistants

This file governs how Claude and other AI assistants should behave when working in this repository. Read this file before making any changes.

---

## Repository Purpose

This is the QA Center Blueprint repository — the canonical source for a detailed, multi-volume reference document describing an AI-native Quality Engineering platform. Content is authoritative for the platform's vision, architecture, user experience, and implementation guidance.

---

## Absolute Rules

These rules apply to every task, with no exceptions:

### 1. Preserve Existing Content
- Never delete, overwrite, or restructure existing content without an explicit instruction to do so.
- If you are unsure whether a change would destroy content, ask before proceeding.

### 2. Do Not Renumber Volumes or Chapters Without Explicit Approval
- Volume numbers (00–12) and chapter numbers are stable identifiers used in cross-references throughout the document.
- Renumbering requires an explicit instruction that names the specific volumes or chapters to be renumbered.
- Do not renumber as a side effect of reordering, reorganizing, or refactoring.

### 3. Use American English
- Spelling, grammar, and vocabulary must follow American English conventions.
- Examples: "color" not "colour", "organize" not "organise", "catalog" not "catalogue".

### 4. Markdown Is the Canonical Format
- All content must be authored in Markdown (`.md`).
- Do not create or edit DOCX, PDF, or HTML files — those are generated artifacts in `generated/`.
- Do not add publishing toolchain configuration (MkDocs, Sphinx, Pandoc config files, etc.) unless explicitly instructed.

### 5. Use Mermaid for Editable Diagrams
- All diagrams that should be editable must use Mermaid syntax embedded in fenced code blocks (` ```mermaid `).
- Do not embed raster images as replacements for diagrams that should be editable.
- Pre-rendered diagram images may be placed in `assets/` as companions to Mermaid sources, not substitutes.

### 6. Keep Generated Files Out of Source Directories
- `generated/` is the only permitted location for built output (DOCX, PDF, HTML).
- Do not place build artifacts, compiled outputs, or generated content anywhere inside `docs/`, `templates/`, or `diagrams/`.

### 7. Architecture Decisions Must Use ADRs
- Any significant architectural choice must be documented using an Architecture Decision Record (ADR) in `architecture/adr/`.
- ADR files must follow the ADR template in `templates/`.
- Do not embed architectural decisions as prose in chapter text without a corresponding ADR.

### 8. Summarize Major Edits Before Completing
- Before completing any edit that spans more than one file or makes substantive changes to a chapter, produce a summary of what will change and confirm with the user.
- Minor edits (typos, formatting, small clarifications) do not require a pre-completion summary.

### 9. Never Fabricate Product Facts or Implementation Status
- Do not invent feature names, capability descriptions, metric values, or implementation timelines.
- If something is unknown or undecided, say so explicitly.
- Use appropriate labels (see below) to distinguish established facts from proposals or future ideas.

### 10. Label Proposals, Decisions, Assumptions, and Future Ideas
Use consistent inline labels throughout the document:

| Label | Meaning |
|-------|---------|
| `[PROPOSAL]` | Idea under consideration, not yet decided |
| `[DECISION]` | Adopted direction — do not reopen without justification |
| `[ASSUMPTION]` | Working assumption that may need validation |
| `[FUTURE]` | Capability or idea scoped out of current phase |
| `[OPEN]` | Unresolved question requiring a decision |

---

## File Organization

- `docs/NN-name/` — chapter Markdown files for each volume (e.g. `docs/01-product-vision/`)
- `docs/appendix/` — glossary, ADR index, reference tables
- `architecture/adr/` — Architecture Decision Records
- `architecture/diagrams/` — technical Mermaid diagrams
- `architecture/decisions/` — decision logs and rationale documents
- `architecture/data-models/` — data model definitions
- `architecture/api-design/` — API contracts and specifications
- `architecture/ui-design/` — UI design specs and component notes
- `architecture/prompts/` — AI prompt library and versioned templates
- `templates/` — document templates (chapter, ADR, issue)
- `diagrams/` — high-level editable Mermaid diagram sources
- `assets/` — images, screenshots, branding materials
- `scripts/` — build and generation scripts
- `generated/` — built output, do not edit

---

## Writing Style

- Write in clear, direct prose. Avoid filler phrases like "it is worth noting that" or "as previously mentioned."
- Prefer active voice.
- Use numbered lists for sequential steps. Use bullet lists for non-sequential items.
- Headings should be descriptive, not clever. Avoid puns or metaphors in headings.
- Every chapter should be self-contained enough to be read independently. Minimize unexplained forward references.

---

## Working with This Repository

When given a task:
1. Read the relevant existing files before writing anything.
2. Identify whether the task requires a summary before execution (see Rule 8).
3. Check whether an ADR is required (see Rule 7).
4. Apply all labels (see Rule 10) where content status is ambiguous.
5. Confirm the task is complete by listing what was created or changed.
