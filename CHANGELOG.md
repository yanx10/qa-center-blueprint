# Changelog

All notable changes to the QA Center Blueprint are documented in this file.

This changelog covers structural and content milestones. Minor edits (typos, formatting) are not recorded individually.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [Unreleased]

---

## [0.6.0] — 2026-07-27

### Summary
M2 — Analysis Persistence Foundation specification complete. Milestone sequence extended to M14 with new provider milestones (M9–M11) reserved for future GitHub integration.

### Added
- `product/features/0001-change-intelligence/milestones/m2-analysis-persistence-foundation.md` — full M2 milestone specification (37 sections, 36 acceptance criteria, Mermaid ER diagram, complete API contracts for 6 endpoints, migration specification, rollout plan, test strategy)

### Changed
- `product/features/0001-change-intelligence/implementation-plan.md` — inserted new M2 (Analysis Persistence Foundation); renumbered old M2→M3, M3→M4, M4→M5, M5→M6, M6→M7, M7→M8, M8→M12, M9→M13, M10→M14; added new future milestones M9 (Repository and Provider Foundation), M10 (GitHub Connection and Manual Synchronization), M11 (Automated Synchronization and Webhooks)
- `product/features/0001-change-intelligence/acceptance-criteria.md` — inserted M2-AC-01 through M2-AC-36 for Analysis Persistence Foundation; renamed old M2-AC-01–03 to M3-AC-01–03; renamed M3→M4, M4→M5, M5→M6, M6→M7 acceptance criteria sections; renamed M9-AC-01 to M13-AC-01
- `product/features/0001-change-intelligence/decisions.md` — appended D-018 (draft/ready lifecycle), D-019 (input immutability), D-020 (content excluded from GET responses), D-021 (retry creates new record), D-022 (500,000-character content limit)
- `product/features/0001-change-intelligence/README.md` — updated summary from [OPEN] placeholder to real description; updated status to "In Progress — M1 Complete, M2 Specification Ready"; added `milestones/` directory row to contents table
- `product/roadmap.md` — extended Phase 1 milestone table from M9 to M14; updated M1 and blueprint reconciliation status; added M2–M14 with correct statuses
- `README.md` — updated to v0.6.0; updated Current Milestone to M2 Analysis Persistence Foundation; updated progress bars; added M2 Specification row to Documentation Index; added M2 spec complete entry to Recent Progress; updated Next Milestone section

### Decisions recorded
- D-018: Analysis lifecycle starts in draft and locks inputs at ready
- D-019: Inputs are immutable once the parent analysis leaves draft
- D-020: Input content is excluded from all GET API responses
- D-021: A failed or cancelled analysis cannot be retried in place; a new record must be created
- D-022: Input content is limited to 500,000 characters

### Why
No implementation code should be written before the blueprint for that milestone is reviewed and approved. M2 is the persistence foundation that all subsequent Change Intelligence milestones depend on. The milestone sequence was extended to M14 to accommodate new provider-integration milestones (M9–M11) that were previously conflated with Phase 1 scope and are now correctly deferred to after the pilot (M13).

---

## [0.5.0] — 2026-07-27

### Summary
README refined with improved navigation, repository context, and milestone tracking.

### Changed
- `README.md` polished with nine targeted improvements:
  - Added subtitle *AI-Native Quality Engineering Platform* below the repository title
  - Added Quick Links section for direct navigation to key sections and the implementation repository
  - Added Milestone Status table showing per-capability milestone progress alongside the progress bars
  - Expanded Repositories section from a single-axis summary into a two-axis comparison table
  - Added Blueprint Repository Structure directory tree after the Repositories section
  - Added How to Contribute section near the Development Workflow
  - Added introductory line to the Documentation Index
  - Progress bars: milestone status indicators (`M1 ✅ M2 🔄 M3–M9 ⬜`) replace hardcoded percentages for in-progress capabilities
  - Blueprint version updated to v0.5.0

### Why
Targeted polish to make the README more useful as both an executive dashboard and a daily navigation hub. The milestone status table and Quick Links reduce the time to answer "where are we?" and "where is that document?"; the structure tree and expanded repository comparison give new contributors immediate orientation without reading multiple files.

---

## [0.4.0] — 2026-07-27

### Summary
README redesigned as the executive project dashboard for the QA Center Blueprint repository.

### Changed
- `README.md` fully replaced with an executive-grade project dashboard
- New structure: Project Status Dashboard · Overall Progress by product area · Project Vision · Repository relationship · Development Workflow · Roadmap · Current Milestone detail · Completed Milestones table · Documentation Index · Engineering Principles · Definition of Done · Recent Progress · Next Milestone · Long Term Vision
- Technical implementation details (Docker, Kubernetes, environment variables, deployment) removed — those belong in the `qa-center` repository
- Documentation Index added — every major document and directory linked by category
- Engineering Principles documented as a formal table
- Progress tracking expanded to eleven product areas individually

### Why
The blueprint repository is the executive source of truth for the QA Center project. Its README must answer the questions any stakeholder or contributor needs answered in five minutes: what is QA Center, where is it today, what has been completed, what is actively being built, and where is every important document.

---

## [0.3.0] — 2026-07-27

### Summary
Project Dashboard — README redesigned as the daily project entry point; repository structure completed.

### Added
- `README.md` redesigned as a full Project Dashboard covering: project vision, progress bars, current focus, feature roadmap, milestone timeline (completed and upcoming), repository organization, development workflow, definition of done, project statistics, next development session guidance, and future vision
- `decisions/` top-level directory — reserved for cross-cutting decision records
- `glossary/` top-level directory — reserved for shared terminology and definitions
- `CHANGELOG.md` entries for v0.2.0 and v0.3.0

### Changed
- `README.md` supersedes the previous GitHub landing page; it is now the canonical project dashboard opened at the start of every development session

### Why
The repository is evolving from a document store into the source of truth for the entire QA Center project. The README must reflect current project state at a glance — what is done, what is next, and where to continue.

Going forward, updating `README.md` and `CHANGELOG.md` is part of every milestone's Definition of Done.

---

## [0.2.0] — 2026-07-26

### Summary
Feature 0001 — Change Intelligence blueprint complete and reconciled against the actual application.

### Added
- `product/` top-level directory — product specifications, roadmaps, and feature blueprints
- `product/roadmap.md` — phase-level product roadmap
- `product/phase-1.md` — detailed Phase 1 scope, objectives, exit criteria, and risks
- `product/features/0001-change-intelligence/` — complete Feature 0001 blueprint:
  - `product-spec.md` — problem statement, goals, feature overview, platform context
  - `implementation-plan.md` — 11-milestone plan (M0–M10)
  - `decisions.md` — D-001–D-017 approved decisions; OD-001–OD-015 open/deferred decisions
  - `data-model.md` — proposed schema for 5 new tables; confirmed DB conventions
  - `api-design.md` — endpoint contracts for M1–M6; confirmed response envelope
  - `acceptance-criteria.md` — BC-01–BC-15 backward compatibility; M1-AC-01–M1-AC-18 Milestone 1 criteria
  - `ai-workflow.md` — 8 sequential AI stages; confirmed synchronous constraints; pre-M2 requirements
  - `user-experience.md` — personas, user journeys, M1 shell page spec, edge cases, accessibility

### Changed
- Milestone 0 (architecture discovery) completed — all 10 inspection areas documented in `qa-center/docs/discovery/change-intelligence/`
- Blueprint reconciled against confirmed application architecture:
  - Migration number corrected from `020` to `030_change_intelligence.sql`
  - Response envelope confirmed as `{ success, data/error }`
  - `requireAuth()` per-route pattern confirmed (no middleware auth on `/api/*`)
  - AI Studio review component confirmed as not reusable (D-009)
  - AI Studio tables confirmed as incompatible — separate CI tables required (D-008)
  - `CHANGE_INTELLIGENCE_ENABLED` server-side env flag adopted (D-012)
  - `created_by UUID REFERENCES users(id)` pattern adopted (D-011)
  - Synchronous-only AI integration confirmed for Phase 1 (D-016)
- All [OPEN] questions from `api-design.md`, `data-model.md`, `product-spec.md` resolved

### Decisions recorded
- D-008: Separate persistence tables — no AI Studio table reuse
- D-009: Separate review interface — no AI Studio review component reuse
- D-010: Migration file `030_change_intelligence.sql`
- D-011: `created_by UUID REFERENCES users(id)` for all CI tables
- D-012: `CHANGE_INTELLIGENCE_ENABLED` server-side environment flag
- D-013: Route `/change-intelligence`, API `/api/change-intelligence/`
- D-014: `requireAuth()` explicit in every CI API route
- D-015: Standard response envelope `{ success, data/error }`
- D-016: Synchronous Anthropic pattern retained for Phase 1
- D-017: Full PR diff storage temporarily permitted

### Why
No implementation code should be written before the blueprint for that milestone is reviewed and approved. Milestone 0 confirmed the application architecture; the reconciliation step resolved all pre-discovery assumptions that had been recorded as open questions.

---

## [0.1.0] — 2026-07-26

### Added
- Initial repository structure with root configuration files
- `docs/NN-name/` directory scaffold for all thirteen volumes (00–12) and appendix
- `architecture/` tree: `adr/`, `diagrams/`, `decisions/`, `data-models/`, `api-design/`, `ui-design/`, `prompts/`
- `templates/`, `diagrams/`, `assets/`, `scripts/`, `generated/` directories
- `README.md`, `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, `LICENSE`
- `.gitignore`, `.editorconfig`
- `.github/workflows/` and `.github/ISSUE_TEMPLATE/` directories (reserved for future configuration)

---

<!-- Future releases will be recorded above this line. -->
