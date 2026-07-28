# Feature Overview — Change Intelligence

## What It Does

Change Intelligence generates actionable QA analysis from engineering artifacts. A QA lead provides a PR diff and requirement documents; the system returns a structured report that identifies risk areas, coverage gaps, regression targets, and edge cases — in under two minutes.

The goal is not to replace QA judgment. It is to compress the time required to form an informed judgment about what to test.

---

## The Problem

When a PR is raised, a QA engineer must answer several questions before deciding what to test:

- What does this change actually do?
- Which requirements does it address — and which does it miss?
- What might break that wasn't intentionally changed?
- What edge cases aren't covered?
- Where should regression testing focus?

Answering these questions manually takes 30–90 minutes per PR. On high-velocity teams, this bottleneck forces a choice between shallow coverage or delayed releases.

Change Intelligence eliminates this research phase. The report it produces answers all five questions in a consistent, structured format.

---

## Who It Is For

**Morgan — QA Engineer.** Runs analyses during active development cycles. Morgan inputs a PR diff and requirement text, receives an analysis, and uses the report to design test scenarios. The two-minute turnaround fits sprint velocity.

**Alex — QA Lead.** Reviews completed analyses and tracks the team's coverage posture. Alex cares about the risk level summaries, missing requirements, and recommended test focus areas.

Both personas are described in detail in `user-experience.md`.

---

## How It Works

1. **Create an analysis.** The user creates a draft analysis record with a title and optional project or release association.

2. **Add inputs.** The user attaches a PR diff, requirement documents (Jira stories, acceptance criteria, PRD text, or plain requirement text), and any supplemental context. Inputs are locked when the analysis is marked ready.

3. **Generate.** The user triggers AI processing. The system constructs a prompt from the inputs, calls Claude, and validates the structured output. Processing takes 15–60 seconds.

4. **Review the report.** The completed analysis displays the QA Intelligence Report: an executive summary, change analysis, risk assessment, impacted components, regression recommendations, high-risk scenarios, missing requirements, ambiguities, edge cases, and recommended test focus areas.

5. **Act on findings.** The QA engineer uses the report to design test scenarios, prioritize regression coverage, and escalate missing requirements to the engineering team.

---

## Milestone Sequence

| Milestone | What It Delivers |
|-----------|-----------------|
| M0 | Application discovery — confirmed architecture before writing any code |
| M1 | Feature shell — navigation item, feature flag, empty-state page |
| M2 | Persistence foundation — database tables, API endpoints, draft/ready lifecycle |
| **M3** | **AI analysis — generates the QA Intelligence Report from inputs (current)** |
| M4 | Requirement comparison — individual finding review actions |
| M5 | Risk review — acknowledge or dispute risk findings |
| M6 | Manual test generation — proposed test cases with human approval gate |
| M7 | Playwright proposals — reviewable automation code |
| M8 | Existing context integration — link to projects, releases, PRDs |
| M9–M11 | GitHub integration — auto-fetch PR diffs from connected repositories |
| M12 | Auditability — full retention policy and audit records |
| M13 | Controlled pilot — metrics collection and evaluation |
| M14 | Expansion decisions — based on pilot data |

---

## What M3 Produces

The M3 AI analysis returns a structured report with the following sections:

| Section | What It Contains |
|---------|-----------------|
| Executive Summary | Risk level, headline, primary concern, recommended action |
| Change Summary | Description, change type, scope, key areas modified |
| Risk Assessment | Overall risk level, rationale, individual risk factors |
| Impacted Components | Direct and indirect components affected by the change |
| Regression Recommendations | Areas to regression-test and why |
| High Risk Scenarios | Scenarios that could fail, with likelihood and mitigation |
| Missing Requirements | Requirements implied by the change but not addressed |
| Ambiguities | Areas where the requirements are unclear |
| Edge Cases | Scenarios the change may not handle correctly |
| Recommended Test Focus | Prioritized list of areas to test, with suggested scenarios |
| Confidence and Limitations | Overall confidence, completeness of inputs, known gaps |

---

## What M3 Does Not Do

M3 generates the QA Intelligence Report only. The following are explicitly out of scope:

- Test case generation (M6)
- Playwright or other automation code generation (M7)
- GitHub PR fetching (M9–M11)
- Jira automatic linking
- Requirement comparison with human review actions (M4)
- Multi-agent orchestration
- Streaming responses
- Editable prompts

---

## Key Design Decisions

**JSONB storage (D-028).** The complete AI output is stored as a single JSONB field. This decouples the prompt structure from the database schema — prompt improvements don't require migrations.

**Synchronous execution (D-029).** The generate endpoint waits for AI completion. No background jobs or polling in M3. The UI shows a loading state for 15–60 seconds.

**In-place retry (D-032).** A failed analysis can be retried up to 3 times using the same inputs, updating the same record. This keeps the analysis history clean.

**Temperature 0.2 (D-034).** Fixed at 0.2 for consistent, evidence-based analysis. QA analysis is not a creative task.

---

## Related Documents

| Document | Purpose |
|----------|---------|
| `milestones/m3-ai-change-analysis.md` | Full M3 milestone specification |
| `prompt-design.md` | AI prompt design and versioning |
| `analysis-schema.md` | JSON schema for the QA Intelligence Report |
| `ui-design.md` | UI states and report rendering specification |
| `data-model.md` | Database schema for all milestones |
| `api-design.md` | API endpoint contracts |
| `decisions.md` | All architectural decisions |
| `acceptance-criteria.md` | Verifiable acceptance criteria |
| `user-experience.md` | Personas, journeys, and UX spec |
