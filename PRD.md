# Rillet AI Studio — Product Requirements Document

**Version**: 3.0
**Date**: 2026-03-13
**Owner**: Chief of Staff
**Status**: Live

---

## 1. Problem

As AI adoption grows organically across business units, companies face three compounding problems:

1. **No visibility** — Teams build AI workflows in isolation. Finance doesn't know Sales already solved a similar problem. Duplicate effort compounds.
2. **No governance** — Unvetted AI workflows ("shadow AI") run without human review gates, risk classification, or accountability. One bad output reaches a customer or board.
3. **No reuse** — A workflow built for one BU could serve three others, but there's no mechanism to discover, evaluate, or adopt it.

Without a central system, AI becomes a liability instead of leverage.

## 2. Product

**Rillet AI Studio** is a shared catalog of all AI-assisted workflows running across business units. It provides cross-BU visibility, governance enforcement, and a reuse framework so teams can discover, evaluate, and adopt automations instead of rebuilding them.

### What it is
- A single-page web dashboard deployed via GitHub Pages
- Source of truth: `data/registry.json`
- No backend, no build step, no authentication (internal tool)

### What it is not
- Not a workflow execution engine — it catalogs workflows, not runs them
- Not a prompt library — it tracks full pipelines (sources, logic, outputs, review gates)
- Not a project management tool — no sprints, no assignments, no timelines

## 3. Users

| Role | Primary use | Frequency |
|------|------------|-----------|
| **BU Lead** (VP Sales, Controller, Head of CS) | See what workflows exist in their BU and others, evaluate reuse candidates | Weekly |
| **Chief of Staff** | Governance oversight, track coverage across BUs, review requests | Daily |
| **Builder** (analyst, ops person, engineer) | Add new workflows, document how they work, comment on existing ones | As needed |
| **Cross-BU browser** (any employee) | Search for automations that could help their team, submit requests | Ad hoc |

## 4. Core Features

### 4.1 Workflow Registry Table

The primary interface. A sortable, filterable, searchable table of all workflows.

**Columns**: Icon, Name + Description, BU, Type (1: Deterministic / 2: Generative), Status (Live / Pilot), Time Saved

**Filters**: Business Unit, Status, Type
**Search**: Full-text across name and description
**Sort**: Name, BU, Type, Status, Saves

### 4.2 Expandable Detail Row

Click any workflow to reveal:

**Pipeline Visualization**
- Sources (inputs) → AI Logic (processing steps) → Outputs → Review Gate
- Color-coded chips per stage (slate/teal/plum/gold)

**Governance**
- Reviewer (role title of human who signs off)
- Risk level (Low / Medium)
- Rewrite rate (% of outputs requiring human edits)
- Reused By (list of BUs currently using this workflow)

**Reuse Intelligence**
- Readiness: High / Medium / Low — how portable the workflow is
- Design quality: Modular / Coupled / Fragile — how maintainable
- Output utility: Actionable / Informational / Raw — how directly usable the outputs are
- Stack: tools and integrations used (chips)
- Top risk: primary concern when adopting in another BU

**Comments**
- Thread per workflow, visible in detail row
- Any user can add comments (session-stored)
- Seed data shows real team feedback

**Other Use Cases**
- Suggested BU/scenario applications beyond current usage
- Displayed as chips for quick scanning

### 4.3 Sidebar Panels

**Cross-BU Reuse Map**
- Shows which BUs share workflows: `GTM / Sales → Implementation, Customer Success`
- Auto-computed from `reused_by` fields

**AI Studio Library**
- Chronological feed of registry events (promotions, reuse adoptions, notes)
- Queryable input field
- Requests from "Request Workflow" flow appear here

### 4.4 Add Workflow Modal

For builders to register a new workflow.

**Fields**:
- Name, BU, Description (required)
- Type, Risk (classification)
- Reviewer
- Details (free text — Claude auto-generates `prompt_summary` from this via API)

**Behavior**:
- Inline validation with error messages below fields
- Required field indicators (red asterisk)
- Submit button disabled during Claude generation
- Success toast on completion
- New workflow added as "Pilot" status

### 4.5 Request Workflow Modal

For any team member to request a new automation.

**Fields**:
- Business Unit (required)
- Task description (required) — "What do you want automated?"
- Time estimate
- Urgency (Normal / High)

**Behavior**:
- Auto-matches against existing workflows on blur — surfaces potential reuse before building new
- Submission logged to Studio Library feed
- Confirmation screen after submit

### 4.6 Stats Bar

Three metrics at the top of the page:
- **Workflows**: total count
- **Hrs Saved / Wk**: aggregate time savings across all workflows
- **Governance**: % of workflows with a named reviewer (not TBD)

### 4.7 Export

JSON export of the full registry for downstream use.

## 5. Data Schema

Each workflow entry in `data/registry.json`:

| Field | Type | Rules |
|-------|------|-------|
| `id` | number | Sequential, never reused |
| `name` | string | Title case, max 50 chars |
| `desc` | string | One sentence, max 120 chars |
| `bu` | string | Must match existing BU list |
| `type` | number | 1 = deterministic, 2 = generative |
| `risk` | string | "Low" or "Medium" |
| `status` | string | "Live" or "Pilot" |
| `saves` | string | "N hrs/wk" or "N hrs/mo" or "—" |
| `day` | number | Day offset for ordering |
| `prompt_summary` | string | Structured: inputs → logic → output → reviewer |
| `reused_by` | string[] | BU names reusing this workflow |
| `reviewer` | string | Role title, "TBD" if unknown |
| `rewrite_rate` | string | "N%" or "—%" |

### Supplementary Data (client-side)

| Dataset | Per-workflow fields |
|---------|-------------------|
| **Reuse Intelligence** | stack, reusable components, BU-specific items, readiness (H/M/L), days to live, reuse setup days, risks[] |
| **Quality Signals** | design (Modular/Coupled/Fragile) + note, output (Actionable/Informational/Raw) + note |
| **Pipeline** | sources[], ai[], outputs[], gate |
| **Icons** | 2-letter abbreviation, background color |
| **Comments** | author, text, timestamp (session-stored) |
| **Use Cases** | string[] of suggested applications |

## 6. Business Units

GTM / Sales, Finance, Customer Success, Implementation, Chief of Staff, People Ops, Product, Marketing, Legal, Procurement

## 7. Classification Framework

### Type
- **Type 1 — Deterministic**: Fixed rules, thresholds, regex. Output is predictable given same input.
- **Type 2 — Generative**: LLM-based generation, summarization, classification. Output varies; more human review needed.

### Risk
- **Low**: Output is informational or advisory. No action taken without human sign-off.
- **Medium**: Output directly triggers or informs a consequential action. Reviewer gate mandatory.

### Reuse Readiness
- **High**: Portable with minimal config changes. Prompt templates and logic transfer directly.
- **Medium**: Requires integration work (API credentials, schema mapping) but core logic transfers.
- **Low**: Deeply tied to source BU's data, terminology, or tools. Major customization needed.

### Design Quality
- **Modular**: Easy to maintain, extend, and configure. Components are decoupled.
- **Coupled**: Works but tightly integrated with specific systems. Changes ripple.
- **Fragile**: Hardcoded assumptions. Difficult to modify without breaking.

### Output Utility
- **Actionable**: Outputs directly drive decisions or trigger downstream workflows.
- **Informational**: Outputs support decisions but require human interpretation.
- **Raw**: Outputs need significant processing or context before they're useful.

## 8. Architecture

```
index.html          — Single-file app (HTML + CSS + JS)
data/registry.json  — Source of truth
.github/workflows/  — GitHub Actions deploy to Pages on push to main
```

- **No backend**: Static site, data loaded at runtime from JSON
- **No build step**: Ship the HTML directly
- **No auth**: Internal tool, access controlled at network/repo level
- **Claude API**: Optional integration for auto-generating prompt summaries (requires API key in localStorage)

## 9. Non-Goals (v3)

- User authentication or role-based access
- Workflow execution or scheduling
- Real-time collaboration (comments are session-scoped)
- Notifications or alerts
- Integration with Slack, Jira, or other tools
- Mobile-optimized layout (responsive but desktop-first)
- Audit trail or change history (git log serves this purpose)

## 10. Future Considerations

- **Persistent comments**: Store in JSON or external service instead of session
- **Request tracking**: Status pipeline for workflow requests (Submitted → Assessed → Building → Live)
- **Reuse adoption flow**: One-click "I want this for my BU" with automated setup checklist
- **Usage metrics**: Actual run counts, error rates, time saved per execution
- **Slack integration**: Surface new workflows and requests in relevant channels
- **Search by similarity**: Semantic search across workflow descriptions and prompt summaries
- **Version history**: Track changes to individual workflows over time

## 11. Success Metrics

| Metric | Target | How measured |
|--------|--------|-------------|
| Workflows cataloged | 20+ within 90 days | Count in registry |
| Cross-BU reuse | 30%+ of workflows reused by at least 1 other BU | `reused_by` field |
| Governance coverage | 100% of workflows have named reviewer | Governance stat |
| Time saved | 40+ hrs/wk aggregate | Saves field sum |
| Request-to-live | < 14 days from request to pilot | Day field tracking |
| Shadow AI reduction | 0 unvetted workflows discovered per quarter | Studio Library signals |
