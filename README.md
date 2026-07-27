# QA Center Blueprint

**QA Center** is an AI-native Quality Engineering platform designed to unify testing strategy, automation, AI-driven analysis, and release intelligence into a single coherent system.

This repository is the **canonical source** for the QA Center Blueprint — the definitive reference document describing the platform's vision, architecture, capabilities, and implementation guidance.

---

## Source of Truth

Markdown (`.md`) files in this repository are the authoritative source. All other formats are generated artifacts:

| Format | Location | How produced |
|--------|----------|--------------|
| Markdown | `docs/` | Edited directly — canonical source |
| DOCX | `generated/docx/` | Generated from Markdown |
| PDF | `generated/pdf/` | Generated from Markdown |
| HTML | `generated/html/` | Generated from Markdown |

Do not edit generated files directly. Changes made outside of Markdown source will be lost on the next build.

---

## Blueprint Structure

The Blueprint is organized into thirteen volumes plus appendices:

| Number | Title |
|--------|-------|
| 00 | Foundation |
| 01 | Product Vision |
| 02 | User Experience |
| 03 | AI Platform |
| 04 | AI Agents |
| 05 | Memory |
| 06 | Testing Platform |
| 07 | Automation Platform |
| 08 | Release Intelligence |
| 09 | Platform Architecture |
| 10 | Integrations |
| 11 | Engineering |
| 12 | Future Vision |
| Appendix | Reference material, glossary, ADR index |

---

## Project Status

**This project is currently in review-stage development.**

Content is being drafted, reviewed, and refined. Volumes may be incomplete. Proposals and future ideas are clearly labeled within each document. Do not treat any section as an implementation commitment unless it is explicitly marked as an adopted decision.

---

## Repository Layout

```
qa-center-blueprint/
├── docs/                    # Markdown source — Volumes (canonical)
│   ├── 00-foundation/
│   ├── 01-product-vision/
│   ├── 02-user-experience/
│   ├── 03-ai-platform/
│   ├── 04-ai-agents/
│   ├── 05-memory/
│   ├── 06-testing-platform/
│   ├── 07-automation-platform/
│   ├── 08-release-intelligence/
│   ├── 09-platform-architecture/
│   ├── 10-integrations/
│   ├── 11-engineering/
│   ├── 12-future-vision/
│   └── appendix/
├── architecture/            # Engineering source of truth — implementation detail
│   ├── adr/                 # Architecture Decision Records
│   ├── diagrams/            # Technical diagrams (Mermaid)
│   ├── decisions/           # Decision logs and rationale
│   ├── data-models/         # Data model definitions
│   ├── api-design/          # API contracts and specifications
│   ├── ui-design/           # UI design specs and component notes
│   └── prompts/             # AI prompt library and versioned templates
├── templates/               # Reusable document templates
├── diagrams/                # High-level editable diagram sources (Mermaid)
├── assets/
│   ├── branding/
│   ├── ui/
│   └── screenshots/
├── scripts/                 # Build and generation scripts
├── generated/               # Output artifacts — do not edit
│   ├── docx/
│   ├── pdf/
│   └── html/
└── .github/                 # GitHub configuration
    ├── workflows/
    └── ISSUE_TEMPLATE/
```

### docs/ vs architecture/

`docs/` contains the **vision** — what QA Center is, why it exists, and what it does.

`architecture/` contains the **implementation** — data models, API contracts, UI specs, ADRs, and the AI prompt library. As the project grows, `architecture/` becomes the engineering team's working reference while `docs/` remains the narrative record of intent.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on submitting edits, proposing changes, and working with the ADR process.

---

## License

MIT License — Copyright (c) 2026 Yan Xia. See [LICENSE](LICENSE) for full terms.
