# QA Center Blueprint

*AI-Native Quality Engineering Platform*

**QA Center** is an AI-powered Quality Engineering Platform being built using a Blueprint-first development methodology.

This repository is the **single source of truth** for the QA Center product — containing the vision, architecture, roadmap, feature specifications, implementation plans, acceptance criteria, and engineering decisions that define everything that gets built.

> Every feature begins here. No implementation code is written until its blueprint has been reviewed and approved.

---

## Quick Links

| Section | Purpose |
|---------|---------|
| [Project Status Dashboard](#project-status-dashboard) | Current milestone, sprint, and phase |
| [Current Milestone](#current-milestone) | What is being built right now |
| [Roadmap](#roadmap) | Phase-level plan for all capabilities |
| [Documentation Index](#documentation-index) | Every specification and architectural document |
| [Engineering Principles](#engineering-principles) | How we develop and make decisions |
| [Definition of Done](#definition-of-done) | Milestone completion criteria |
| [Implementation Repository](../qa-center) | Production application source code |

---

## Project Status Dashboard

| Field | Value |
|-------|-------|
| **Product Status** | Active Development |
| **Blueprint Version** | v0.6.0 |
| **Implementation Version** | — *(see [qa-center](../qa-center))* |
| **Current Phase** | Phase 1 — Change Intelligence Foundation |
| **Current Milestone** | M2 — Analysis Persistence Foundation |
| **Current Sprint** | Change Intelligence · Analysis Persistence |
| **Current Focus** | M2 specification complete — persistence layer, API contracts, and draft/ready lifecycle ready for implementation |
| **Last Updated** | 2026-07-27 |

---

## Overall Project Progress

Progress is tracked per product area. Baseline platform capabilities are live in production. New AI-native capabilities are in active development.

```
Core Platform              ████████████████████  Live in production
Test Management            ████████████████████  Live in production
Release Quality Reports    ████████████████████  Live in production
Development Gates          ████████████████████  Live in production
AI Studio                  ████████████████████  Live in production
MCP Server                 ████████████████████  Live in production
───────────────────────────────────────────────────────────────────
Change Intelligence        ████░░░░░░░░░░░░░░░░  M1 ✅  M2 📋  M3–M14 ⬜
Release Intelligence       ░░░░░░░░░░░░░░░░░░░░  Phase 2 — not started
AI Test Generation         ░░░░░░░░░░░░░░░░░░░░  Phase 2 — not started
AI Agents                  ░░░░░░░░░░░░░░░░░░░░  Phase 3 — not started
Enterprise Platform        ░░░░░░░░░░░░░░░░░░░░  Phase 3 — not started
───────────────────────────────────────────────────────────────────
Overall QA Center Platform ████████░░░░░░░░░░░░  6 of 11 areas complete
```

### Milestone Status

| Capability | Phase | Milestone Progress | Status |
|-----------|-------|------------------|--------|
| Core Platform | Baseline | — | ✅ Live |
| Test Management | Baseline | — | ✅ Live |
| Release Quality Reports | Baseline | — | ✅ Live |
| Development Gates | Baseline | — | ✅ Live |
| AI Studio | Baseline | — | ✅ Live |
| MCP Server | Baseline | — | ✅ Live |
| Change Intelligence | Phase 1 | M0 ✅ · M1 ✅ · M2 📋 · M3–M14 ⬜ | 🔄 Active |
| Release Intelligence | Phase 2 | — | ⬜ Planned |
| AI Test Generation | Phase 2 | — | ⬜ Planned |
| AI Agents | Phase 3 | — | ⬜ Planned |
| Enterprise Platform | Future | — | ⬜ Planned |

---

## Project Vision

QA Center is evolving from a test management tool into an **AI-native Quality Engineering Platform** — a system where AI understands every code change, predicts risk, generates test coverage, and surfaces release intelligence automatically.

### Platform Capabilities

| Capability | Description | Status |
|-----------|-------------|--------|
| **Test Management** | Create, version, prioritize, and bulk-manage test cases and suites | ✅ Live |
| **Test Runs** | Execute test suites, record step-level results, notes, and attachments | ✅ Live |
| **Release Quality Reports** | AI-assisted quality reports from Jira data with DRE, escape rate, and coverage metrics | ✅ Live |
| **Development Gates** | Six-phase development lifecycle tracking | ✅ Live |
| **AI Studio** | PRD-to-test-case generation with review, approval, and import | ✅ Live |
| **MCP Server** | Expose QA Center capabilities to Claude Desktop, Cursor, and other AI clients | ✅ Live |
| **Change Intelligence** | Compare PR diffs against requirements, identify gaps, generate targeted test proposals | 🔄 Phase 1 |
| **Release Intelligence** | Aggregate change-level findings into release-level risk scores | ⬜ Phase 2 |
| **AI Test Generation** | Autonomous test case generation from requirements and code changes | ⬜ Phase 2 |
| **AI Agents** | Delegate full PR review cycles to autonomous QA agents | ⬜ Phase 3 |
| **Enterprise Platform** | Multi-tenant, SSO, audit, compliance, and enterprise controls | ⬜ Future |

---

## Repositories

QA Center is developed across two repositories with a clear division of responsibility.

| | `qa-center-blueprint` *(this repo)* | `qa-center` |
|---|---|---|
| **Role** | Product definition | Production implementation |
| **Answers** | What gets built and why | How it gets built and deployed |
| **Contains** | Vision · Roadmap · Architecture · Feature specifications · Implementation plans · Decision records · Engineering principles | Application source code · Database schemas · API routes · Frontend components · AI integration · Docker configuration · Deployment manifests |
| **Authority over** | Scope, product decisions, and engineering standards | Technical implementation details and runtime behavior |

**Blueprint defines WHAT gets built. Implementation defines HOW it gets built.**

No production code is written without a corresponding approved blueprint. Every feature begins in this repository.

### Blueprint Repository Structure

```
qa-center-blueprint/
├── product/                             # Product specifications and feature blueprints
│   ├── roadmap.md                       # Phase-level product roadmap
│   ├── phase-1.md                       # Phase 1 scope, objectives, and exit criteria
│   └── features/
│       └── 0001-change-intelligence/    # Feature 0001 blueprint (10 documents)
├── architecture/                        # Technical architecture documents
│   ├── adr/                             # Architecture Decision Records
│   ├── data-models/                     # Database schema definitions and ER diagrams
│   ├── api-design/                      # API contracts and specifications
│   ├── ui-design/                       # UI component specs and design notes
│   ├── prompts/                         # Versioned AI prompt templates
│   └── diagrams/                        # Technical Mermaid diagrams
├── docs/                                # Platform vision — Volumes 00–12
├── decisions/                           # Cross-cutting engineering decision records
├── glossary/                            # Shared terminology and definitions
├── diagrams/                            # High-level editable Mermaid diagrams
├── templates/                           # Document templates (chapter, ADR, issue)
├── assets/                              # Images, branding, and screenshots
├── scripts/                             # Build and generation scripts
├── generated/                           # Build artifacts — do not edit directly
├── README.md                            # This file — project dashboard
├── CHANGELOG.md                         # Version history and release notes
├── CLAUDE.md                            # Engineering standards for AI assistants
└── CONTRIBUTING.md                      # Contribution guidelines and ADR process
```

---

## Development Workflow

Every feature follows this process from idea to release.

```
Idea or requirement
       ↓
  Research & scoping
       ↓
  Blueprint authored
  (product-spec · implementation-plan · decisions · data-model
   api-design · acceptance-criteria · ai-workflow · user-experience)
       ↓
  Architecture review
  (Milestone 0 — inspect the application before writing any code)
       ↓
  Blueprint reconciled
  (open decisions resolved · assumptions replaced with confirmed facts)
       ↓
  Implementation
  (milestone by milestone · additive only · regression verified at every milestone)
       ↓
  Validation
  (acceptance criteria · backward compatibility · regression checklist)
       ↓
  Documentation updated
  (CHANGELOG · README · release summary)
       ↓
  Git tag
       ↓
  Release
```

---

## How to Contribute

All contributions follow the blueprint-first workflow above. Before opening a pull request in either repository:

1. **Read [`CLAUDE.md`](CLAUDE.md)** — it defines the rules all contributors and AI assistants follow in this repository.
2. **Start with a blueprint** — new features and significant changes require a specification before implementation begins.
3. **Use ADRs for architecture decisions** — see [`architecture/adr/`](architecture/adr/) and the ADR template in [`templates/`](templates/).
4. **Apply content labels** — use `[PROPOSAL]`, `[DECISION]`, `[ASSUMPTION]`, `[FUTURE]`, and `[OPEN]` to mark the status of any content whose standing is ambiguous.
5. **Update CHANGELOG.md and README.md** — both are required as part of every milestone's Definition of Done.

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for full contribution guidelines.

---

## Roadmap

| Phase | Capability | Status |
|-------|-----------|--------|
| Baseline | Core Platform (auth, settings, navigation) | ✅ Complete |
| Baseline | Test Management (cases, suites, runs) | ✅ Complete |
| Baseline | Release Quality Reports | ✅ Complete |
| Baseline | Development Gates | ✅ Complete |
| Baseline | AI Studio | ✅ Complete |
| Baseline | MCP Server | ✅ Complete |
| **Phase 1** | **Change Intelligence** | 🔄 **Active — M2 specification complete** |
| Phase 2 | Release Intelligence | ⬜ Planned |
| Phase 2 | AI Test Generation | ⬜ Planned |
| Phase 3 | Autonomous QA Agents | ⬜ Planned |
| Phase 3 | AI Agent Platform | ⬜ Planned |
| Future | Enterprise Platform | ⬜ Planned |

---

## Current Milestone

### M2 — Analysis Persistence Foundation
**Feature:** Change Intelligence · **Phase:** 1

#### Objective
Introduce the first Change Intelligence database tables and the API layer for managing analysis records. M2 is a pure persistence and lifecycle milestone — no AI processing occurs. It establishes the draft/ready lifecycle, input management, and the six API endpoints that the M3 analysis pipeline will build on top of.

#### Completed Work

| Item | Status |
|------|--------|
| Architecture discovery (M0) — all application conventions confirmed | ✅ Done |
| Feature 0001 blueprint — 10 core documents authored | ✅ Done |
| Blueprint reconciliation — 17 decisions approved, all open questions resolved | ✅ Done |
| M1 Feature Shell — feature flag, navigation item, empty-state page | ✅ Done |
| M2 specification — full milestone spec with 37 sections, 36 acceptance criteria, 5 new decisions | ✅ Done |

#### Remaining Work

| Item | Priority |
|------|---------|
| Database migration `030_change_intelligence.sql` (`change_analyses`, `change_analysis_inputs`) | M2 |
| `GET /api/change-intelligence/analyses` — paginated list (no content) | M2 |
| `POST /api/change-intelligence/analyses` — create draft record | M2 |
| `GET /api/change-intelligence/analyses/:id` — retrieve with input metadata (no content) | M2 |
| `PATCH /api/change-intelligence/analyses/:id` — update title or transition status | M2 |
| `POST /api/change-intelligence/analyses/:id/inputs` — add input to draft | M2 |
| `DELETE /api/change-intelligence/analyses/:id/inputs/:inputId` — remove input from draft | M2 |
| API test coverage — all 6 endpoints × happy path + error cases | M2 |
| Full regression smoke test across all existing workflows | M2 |

#### Success Criteria
- A draft analysis record can be created, updated, and retrieved
- Inputs can be attached to and removed from draft analyses
- Inputs are locked when the analysis is promoted to ready
- All GET endpoints exclude input content from responses
- All 36 M2 acceptance criteria pass
- All existing QA Center features continue to operate without regression

#### Dependencies
- M1 complete (feature flag and `isChangeIntelligenceEnabled()` available) ✅
- Migration tested against a copy of the production schema before merge

#### Next Milestone After M2
M3 — Manual Input Analysis (UI form, AI pipeline, analysis result display)

---

## Completed Milestones

| Milestone | Version | Date | Description |
|-----------|---------|------|-------------|
| Repository Foundation | v0.1.0 | 2026-07-26 | Repository structure, CLAUDE.md, CONTRIBUTING.md, volume scaffolding |
| Change Intelligence Blueprint | v0.2.0 | 2026-07-26 | Feature 0001 fully specified across 10 blueprint documents |
| Architecture Discovery — M0 | v0.2.0 | 2026-07-26 | Application inspected; all tech conventions, auth patterns, and DB conventions confirmed |
| Blueprint Reconciliation | v0.2.0 | 2026-07-27 | 10 decisions approved; all pre-discovery open questions resolved |
| Feature Shell — M1 | v0.3.0 | 2026-07-27 | `CHANGE_INTELLIGENCE_ENABLED` flag, conditional nav item, `/change-intelligence` empty-state page |

---

## Documentation Index

Every specification, architectural document, and reference for the QA Center platform is linked below, organized by category.

### Product

| Document | Location | Contents |
|----------|----------|---------|
| Product Roadmap | [`product/roadmap.md`](product/roadmap.md) | Phase-level plan for all QA Center capabilities |
| Phase 1 Plan | [`product/phase-1.md`](product/phase-1.md) | Change Intelligence Foundation — scope, objectives, risks, exit criteria |
| Feature Index | [`product/features/`](product/features/) | One directory per feature |

### Feature 0001 — Change Intelligence

| Document | Location | Contents |
|----------|----------|---------|
| Product Spec | [`product/features/0001-change-intelligence/product-spec.md`](product/features/0001-change-intelligence/product-spec.md) | Problem statement, goals, feature overview, platform context |
| Implementation Plan | [`product/features/0001-change-intelligence/implementation-plan.md`](product/features/0001-change-intelligence/implementation-plan.md) | Milestone-by-milestone implementation plan (M0–M10) |
| Decisions | [`product/features/0001-change-intelligence/decisions.md`](product/features/0001-change-intelligence/decisions.md) | D-001–D-017 approved decisions; open and deferred decisions |
| Data Model | [`product/features/0001-change-intelligence/data-model.md`](product/features/0001-change-intelligence/data-model.md) | Proposed table schemas, FK conventions, import path |
| API Design | [`product/features/0001-change-intelligence/api-design.md`](product/features/0001-change-intelligence/api-design.md) | Endpoint contracts, response envelope, auth pattern |
| Acceptance Criteria | [`product/features/0001-change-intelligence/acceptance-criteria.md`](product/features/0001-change-intelligence/acceptance-criteria.md) | BC-01–BC-15 backward compatibility; M1-AC-01–M1-AC-18; M2-AC-01–M2-AC-36 |
| AI Workflow | [`product/features/0001-change-intelligence/ai-workflow.md`](product/features/0001-change-intelligence/ai-workflow.md) | 8 AI stages, synchronous constraints, pre-M3 requirements |
| User Experience | [`product/features/0001-change-intelligence/user-experience.md`](product/features/0001-change-intelligence/user-experience.md) | Personas, user journeys, M1 shell spec, accessibility |
| M2 Specification | [`product/features/0001-change-intelligence/milestones/m2-analysis-persistence-foundation.md`](product/features/0001-change-intelligence/milestones/m2-analysis-persistence-foundation.md) | Full M2 milestone spec — schema, lifecycle, 6 API endpoints, 36 acceptance criteria |

### Architecture

| Document | Location | Contents |
|----------|----------|---------|
| Architecture Decision Records | [`architecture/adr/`](architecture/adr/) | ADRs for significant technical decisions |
| Data Models | [`architecture/data-models/`](architecture/data-models/) | Database schema definitions and entity diagrams |
| API Contracts | [`architecture/api-design/`](architecture/api-design/) | API specifications and contracts |
| UI Design | [`architecture/ui-design/`](architecture/ui-design/) | UI component specs and design notes |
| AI Prompts | [`architecture/prompts/`](architecture/prompts/) | Versioned AI prompt templates |
| Technical Diagrams | [`architecture/diagrams/`](architecture/diagrams/) | Mermaid architecture diagrams |

### Standards & Reference

| Document | Location | Contents |
|----------|----------|---------|
| Decision Records | [`decisions/`](decisions/) | Cross-cutting engineering decisions |
| Glossary | [`glossary/`](glossary/) | Shared terminology and definitions |
| Diagram Sources | [`diagrams/`](diagrams/) | High-level editable Mermaid diagrams |
| Document Templates | [`templates/`](templates/) | Chapter, ADR, and issue templates |
| Platform Volumes | [`docs/`](docs/) | Platform vision — Volumes 00–12 |
| Changelog | [`CHANGELOG.md`](CHANGELOG.md) | Version history and release notes |
| Engineering Standards | [`CLAUDE.md`](CLAUDE.md) | AI assistant instructions and project rules |
| Contributing Guide | [`CONTRIBUTING.md`](CONTRIBUTING.md) | Contribution guidelines and ADR process |

---

## Engineering Principles

| Principle | Description |
|-----------|-------------|
| **Blueprint-first Development** | Every feature is fully specified before implementation begins. No code is written for an unapproved milestone. |
| **Documentation-driven Engineering** | Architecture decisions, data models, and API contracts live in this repository. The implementation follows the blueprint. |
| **Milestone-based Delivery** | Work is organized into milestones with explicit objectives, acceptance criteria, and regression requirements. |
| **Additive-only Changes** | No existing feature, route, table, or API contract is removed or renamed as a side effect of adding new capabilities. |
| **Validation before Merge** | Every milestone includes a regression smoke test of all existing workflows before the milestone is closed. |
| **Living Architecture** | Architecture decisions are recorded in ADRs and decision documents. Decisions are not embedded as prose and lost. |
| **Decision Records** | Significant choices are documented with rationale and consequences. Open decisions are tracked explicitly — not resolved by assumption. |
| **Continuous Improvement** | Each milestone feeds learning back into the next blueprint. Discovery findings update assumptions before implementation begins. |

---

## Definition of Done

A milestone is not complete until every criterion below is satisfied.

| # | Criterion |
|---|-----------|
| 1 | ✓ Blueprint approved and up to date |
| 2 | ✓ Implementation complete |
| 3 | ✓ All feature acceptance criteria pass |
| 4 | ✓ All backward compatibility criteria pass |
| 5 | ✓ TypeScript compilation passes |
| 6 | ✓ ESLint passes |
| 7 | ✓ Tests pass |
| 8 | ✓ Production build succeeds |
| 9 | ✓ Docker build succeeds |
| 10 | ✓ CHANGELOG updated |
| 11 | ✓ README updated |
| 12 | ✓ Git tag created |
| 13 | ✓ Release summary completed |

> **README and CHANGELOG updates are part of every milestone — not optional follow-up tasks.**

---

## Recent Progress

| Date | Achievement |
|------|-------------|
| 2026-07-26 | Blueprint repository initialized with full directory structure and standards |
| 2026-07-26 | Feature 0001 — Change Intelligence blueprint authored across 10 specification documents |
| 2026-07-26 | Milestone 0 complete — full architecture discovery of the production application |
| 2026-07-27 | Blueprint reconciliation complete — 10 decisions approved, all open questions resolved |
| 2026-07-27 | Milestone 1 complete — feature shell live with flag, navigation, and empty-state page |
| 2026-07-27 | M2 specification complete — Analysis Persistence Foundation fully specified with 37 sections, 36 acceptance criteria, 5 new decisions (D-018–D-022), and updated milestone sequence M2–M14 |

---

## Next Milestone

### M2 — Analysis Persistence Foundation

**Objective:** Implement the database schema and API layer for managing Change Intelligence analysis records. This milestone introduces the draft/ready lifecycle, input management, and the persistence surface that M3 will build on. No AI processing occurs in M2.

**Specification:** [`product/features/0001-change-intelligence/milestones/m2-analysis-persistence-foundation.md`](product/features/0001-change-intelligence/milestones/m2-analysis-persistence-foundation.md)

**Key deliverables:**

| Deliverable | Description |
|-------------|-------------|
| `030_change_intelligence.sql` | Additive migration introducing `change_analyses` (17 columns) and `change_analysis_inputs` (9 columns) with indexes and check constraints |
| `GET /api/change-intelligence/analyses` | Paginated list — excludes input content |
| `POST /api/change-intelligence/analyses` | Create draft analysis record |
| `GET /api/change-intelligence/analyses/:id` | Retrieve analysis with input metadata — excludes content |
| `PATCH /api/change-intelligence/analyses/:id` | Update title or transition status (draft → ready → cancelled) |
| `POST /api/change-intelligence/analyses/:id/inputs` | Add input to draft analysis (500,000 char limit, content_hash) |
| `DELETE /api/change-intelligence/analyses/:id/inputs/:inputId` | Remove input from draft analysis |
| API test coverage | All 6 endpoints × happy path and error cases |
| Regression verification | Full smoke test of all existing QA Center workflows |

**Expected outcome:** A QA engineer can create a draft analysis, attach a PR diff and requirement documents, promote the analysis to ready, and retrieve the record later — all without any AI processing occurring.

---

## Long Term Vision

QA Center is being built toward a future where quality engineering is a **strategic accelerator** — not a process bottleneck.

The long-term platform will function as an **AI Quality Operating System**:

| Capability | Vision |
|-----------|--------|
| **Change Intelligence** | Every PR is automatically understood through the lens of its requirements, before manual QA begins |
| **Release Intelligence** | Release risk is quantified automatically from change-level data — not assembled manually from test reports |
| **Autonomous QA Agents** | Full PR review cycles are delegated to AI agents that know the test suite, risk model, and release cadence |
| **AI Test Generation** | Test cases and automation proposals are generated from requirements and diffs, not written from scratch |
| **AI Memory** | Institutional knowledge about defect patterns, test history, and system behavior is retained and surfaced automatically |
| **Risk Prediction** | High-risk areas are identified before testing begins, based on historical patterns and change velocity |
| **Engineering Analytics** | Quality metrics inform architecture decisions, team health, and release confidence — not just test pass rates |
| **Enterprise Quality Platform** | Multi-tenant, compliance-ready, audit-capable quality infrastructure at organizational scale |

The goal is not to replace QA engineers. It is to give them the leverage of an entire AI team — so they can focus on judgment, strategy, and the edge cases that matter most.

---

<sub>QA Center Blueprint · v0.6.0 · Blueprint-first Development · MIT License</sub>
