# Contributing to the QA Center Blueprint

Thank you for contributing to the QA Center Blueprint. This document explains how to submit changes, propose new content, and work within the established conventions.

---

## Before You Start

1. Read [CLAUDE.md](CLAUDE.md) — it defines the rules all contributors and AI assistants must follow.
2. Read [AGENTS.md](AGENTS.md) if you are operating as or directing an AI agent.
3. Familiarize yourself with the volume structure in [README.md](README.md).

---

## Types of Contributions

### Content Edits

Corrections to existing text: grammar, spelling, factual accuracy, clarity improvements.

- Edit the relevant `.md` file directly.
- Keep changes minimal and focused.
- Do not alter the surrounding document structure unless that is the purpose of the edit.

### New Content

Adding chapters, sections, or appendix entries.

- Confirm the target volume and chapter number before creating a file.
- Do not create new volume directories or change existing volume names without approval.
- Follow the chapter template in `templates/`.

### Diagrams

Adding or updating diagrams.

- Use Mermaid syntax in fenced code blocks.
- Place standalone diagram files in `diagrams/`.
- Embed inline diagrams directly in the relevant chapter file.

### Architecture Decisions

Any contribution that involves an architectural choice.

- Create an ADR in `docs/appendix/` using the template in `templates/adr-template.md`.
- Reference the ADR from the relevant chapter.
- Do not embed architectural decisions solely in prose without a corresponding ADR.

---

## Content Labeling

Use these labels when content status is not fully settled:

| Label | Meaning |
|-------|---------|
| `[PROPOSAL]` | Idea under consideration, not yet decided |
| `[DECISION]` | Adopted direction |
| `[ASSUMPTION]` | Working assumption that may need validation |
| `[FUTURE]` | Out of scope for the current phase |
| `[OPEN]` | Unresolved question requiring a decision |

---

## Writing Standards

- American English spelling and grammar.
- Active voice preferred.
- Clear, direct prose — no filler phrases.
- Numbered lists for sequential steps; bullets for unordered items.
- Descriptive headings — avoid puns or metaphors.

---

## What Not to Submit

- Changes to files in `generated/` — those are build artifacts.
- Publishing toolchain configuration files unless explicitly requested.
- Raster image replacements for Mermaid diagrams.
- Content that fabricates feature names, timelines, or metric values.
- Renumbering of volumes or chapters without prior approval.

---

## Submitting Changes

1. Create a branch from `main`.
2. Make your changes in the Markdown source files.
3. Run any available linting or validation scripts from `scripts/` before opening a pull request.
4. Open a pull request with a clear title and description of the change.
5. For changes that affect more than one chapter or volume, include a summary in the PR description.

---

## Questions

If you are unsure whether a proposed change is appropriate, open an issue or draft PR to discuss it before investing time in the full edit.
