# Rillet AI Ops — Product Requirements Document

**Version**: 5.0
**Date**: 2026-03-14
**Owner**: Founders Associate

---

## Thesis

**AI adoption creates invisible automation sprawl.** Every team builds their own Claude workflows, connects their own APIs, writes their own prompts — and nobody else knows it exists. Within months, a 100-person company has 20+ automations with no inventory, no reuse, no governance, and no way to answer "what AI is running in our company?"

**Companies need an AI operations control plane** to govern and reuse automation safely. Not another workflow builder — a registry that sits above the execution layer and answers: what exists, who owns it, does it work, and should we build something new or reuse what's already there?

## The Bigger Idea: AI Work Graph

The registry is stage one. But the system is quietly building something more powerful — **a structured map of how work actually happens in a company.**

Each workflow describes a work path: where data comes from, how it's processed, where AI is applied, where the output goes, where humans intervene. One workflow is a recipe. Hundreds of workflows become a graph — a living model of how the company operates.

```
HubSpot → processing → Claude → Slack
Zendesk → classifier → Salesforce
Google Sheets → Claude narrator → board deck
Stripe → threshold check → email alert
```

This graph answers questions no other system can:

- Where is AI touching the company?
- Where is work still manual?
- Which teams depend on the same data?
- Where are the bottlenecks?
- What automations should be built next?

And eventually, it can simulate: **if we automate this node, what downstream work disappears?**

That's organizational systems intelligence.

### Why This Is a Category

Most AI tooling focuses on building agents, writing prompts, or running workflows. Very few focus on **AI system management** — the layer above execution. But as AI adoption grows, that layer becomes critical.

The precedent exists in other domains:

| Domain | The "graph" tool | What it maps |
|--------|-----------------|--------------|
| Infrastructure | Datadog | How servers and services connect |
| Data | dbt + data catalogs | How data flows and transforms |
| Machine Learning | MLFlow | How models are trained and deployed |
| **AI Operations** | **This** | **How work flows across the company** |

### Who This Is For

Not engineers. **Operators.**

- Chief of Staff
- BizOps / RevOps
- Founders Associate
- Finance Ops

These people run the company but have no tooling for automation governance. That's the gap.

### Evolution Path

| Stage | What it does | Status |
|-------|-------------|--------|
| **1. Registry** | Catalog all AI automations | Built (this demo) |
| **2. Governance** | Track reliability, ownership, credentials, data classification | Built (this demo) |
| **3. Discovery** | Find reuse, match existing workflows, prevent duplication | Built (this demo) |
| **4. Work Graph** | Map how work flows across the company. Visualize dependencies. | Next |
| **5. Optimization Engine** | Recommend automations: "You spend 6 hrs/wk on this. Two workflows already produce 80% of the inputs. Build this." | Future |

## What This Is Today

An AI operations control plane. A registry — a catalog and governance layer — not a runtime. It doesn't execute workflows. Code lives elsewhere (GitHub, n8n, internal tools). This platform tracks what exists, who owns it, how well it works, and whether something new needs to be built.

## Problem

The CEO can't answer "how many AI automations are we running?" without Slacking 10 people. The Chief of Staff doesn't know if Sales and CS built the same workflow independently. Finance doesn't know which automations touch PII. Nobody tracks whether the AI output is actually good or just assumed to be.

At Rillet specifically: AI agents (accrual, audit, P&L flux, board decks) run inside the product, and additional Claude-powered workflows are being built across GTM, Finance, CS, and other teams. There's no single place to see all of them.

## ICP

Series A-C startups, 50-500 employees, already using LLMs across multiple teams but managing them through spreadsheets, Notion, or nothing. The buyer is the Chief of Staff, Head of BizOps, or Founders Associate — the person responsible for cross-functional ops and AI rollout.

---

## Core Object Model

These are the foundational objects in the system. Everything in the UI is a view into one or more of these.

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│   Workflow   │──has──▶│    Block     │──uses──▶│     API     │
│              │  many │             │        │             │
│ name         │       │ name        │        │ service     │
│ description  │       │ type        │        │ credentials │
│ status       │       │ description │        │ data class  │
│ saves        │       │ model (AI)  │        │ scope       │
│ prompt ver   │       │ cost tier   │        │ health      │
│ quality      │       │             │        │ comments    │
│ lane         │       └─────────────┘        └─────────────┘
│ reused_by    │
└──────┬───────┘
       │
       │ belongs to          runs as            owned by
       ▼                       ▼                   ▼
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│ Business Unit│       │     Run     │       │    Owner     │
│              │       │             │       │             │
│ name         │       │ status      │       │ name        │
│ workflows[]  │       │ started_at  │       │ role        │
│              │       │ duration    │       │ bu          │
│              │       │ trigger     │       │             │
│              │       │ blocks_run  │       │             │
└─────────────┘       └─────────────┘       └─────────────┘
```

### Workflow
The central object. An automation that does a specific job — "Invoice Discrepancy Flagger", "Deal Summary Generator". Has a status (Live/Pilot), a pipeline of Blocks, an Owner, belongs to a Business Unit, and tracks quality over time (prompt version, acceptance rate, reliability label). Classified into a Work Lane (Product/Product Ops/Ops).

### Block
A logical component in a workflow pipeline. Comes in 5 types: Source, Processing, AI, Output, Approval. Each block connects to an API (or is "Built-in"). AI blocks have a model and cost tier. Blocks are reusable across workflows — the same "HubSpot CRM" block appears in multiple pipelines.

### API
An external service connection (Stripe, HubSpot, Anthropic, Slack). Has one or more credentials with lifecycle tracking (active/rotating/expired), data classification (PII/Financial/Internal), access scope (which BUs can use it), and permissions (read/write). Has a comment thread for team discussion. APIs are the security surface — this is where credential management, audit trails, and data governance live.

### Run
A single execution of a workflow. Tracks: status (success/warning/error), timestamp, duration, which blocks ran, what triggered it, and optional error notes. Runs are the evidence that automations are actually working.

### Owner
A person who creates, maintains, or reviews a workflow. Has a name, role, and optionally a BU. The Owner is who you go to when something breaks. For Medium-risk workflows, the Owner's role determines who reviews output before action.

### Business Unit
An organizational team (GTM/Sales, Finance, Customer Success, etc.). Workflows belong to a BU. APIs are scoped to BUs. The BU is the primary lens for filtering and access control. Cross-functional roles (Founders Associate) have no BU and see everything.

### Relationships

- A **Workflow** has many **Blocks** (ordered as a pipeline)
- A **Block** uses one **API** (or is Built-in)
- An **API** has one or more **Credentials** with data classification
- A **Workflow** belongs to one **Business Unit**
- A **Workflow** has one **Owner**
- A **Workflow** has many **Runs**
- A **Workflow** can be reused by other **Business Units**
- An **API** is scoped to specific **Business Units**

## How It Connects to Actual Automations

This is a registry, not a runtime:
- **Code lives elsewhere** — GitHub, internal tools, no-code platforms like n8n or Flowise. The registry links to it. Each workflow entry is metadata about the automation.
- **Rillet's product AI agents** — accrual, audit, P&L flux, board decks — run inside the product. This registry sits alongside them as the management layer.
- **No one uploads code.** The blocks represent logical components (data source, AI processing, output destination), not executable code.

---

## Core User Flow

### 1. Landing Page (Registry)

**What the user sees:**
- Stats bar: Hours Saved / Week, Live count, Pilot count
- Controls: "My team" toggle, search bar, status filter (Live/Pilot)
- Workflow table: expand arrow, Name (with description + data sensitivity tags), BU, Status (with last run timestamp + status dot), Time Saved
- Sidebar: Recent Runs, API Connections, Popular Across Teams, Open Source Toolbox

**Key behaviors:**
- Founders Associate (no BU) sees all workflows by default, "My team" button hidden
- Users with a BU see their team's workflows by default, "My team" button active
- Stats update based on active filter (team vs all)
- Each workflow row shows PII/Financial data tags inline for compliance visibility
- Last run status (green/amber/red dot + timestamp) shown on every row

### 2. Expanding a Workflow

**What the user sees (top to bottom):**
1. **How it works** — plain-English prompt summary (e.g. "Ingests weekly KPI spreadsheet. Generates a 3-paragraph narrative highlighting WoW changes. Tone-matched to company update style.")
2. **Pipeline** — visual block flow with arrows: Source → Processing → AI → Output → Approval
3. **Reliability** — one badge: "Working well", "Works, sometimes needs edits", "New — still being tested", or "Needs improvement"
4. **Also used by** — other teams reusing this workflow
5. **Also useful for** — other use case ideas
6. **Comments** — team discussion with input

**What was intentionally removed:**
- Reviewer line + risk badge (redundant — prompt summary already says who reviews)
- Reuse Intelligence section with Readiness/Maintainability/Output badges (abstract jargon)
- Stack chips (duplicated what the pipeline shows)

### 3. Clicking "+ New Automation"

**Step 1: Discover (full-screen chat)**
- Claude/ChatGPT-style full-screen interface. Messages centered at 640px, input pinned to bottom.
- Advisor greets user by first name
- Multi-turn conversation gathers: what they do manually, what systems are involved, how long it takes
- Time matching handles natural language ("about 3 hours a week", "half a day every month")
- BU inferred from keywords for cross-functional users, auto-filled for BU-specific users

**Before building anything new, the advisor checks:**
- **Work lane classification** — is this Product (redirect to feature request), Product Ops (tag as graduation candidate), or Ops (proceed)?
- **Existing workflow matching** — searches registry for workflows with overlapping blocks. If found, shows them in a split-view panel (chat stays on left, workflow detail on right). User can click "This works for me" or "Doesn't fit — build new"

**When ready (1+ blocks + time savings), the advisor generates a build spec:**
- Pipeline visualization with typed block chips
- Prompt summary in registry format
- Model selection guidance (why an LLM is needed, which model, cost tier)
- Credentials required with vault status
- Work lane tag (Ops/Product Ops)
- Owner (from session) and BU

**"Build new workflow" button transitions to Step 2.**

**Step 2: Build (form + composer)**
- Form fields pre-filled from chat: Name, BU (dropdown if user has no default), Description, Time This Takes Manually
- Staged pipeline composer below: block catalog on left (searchable), visual pipeline on right grouped by stage (Source → Processing → AI → Output → Approval) with arrows between stages
- Blocks show: name, API provider, credential status dot, model tag for AI blocks
- "Back to chat" returns to Step 1 without losing context

**Step 3: Review & Launch**
- Credential gate: checks every block's API key against the vault
  - Ready (green) — key exists, BU has access
  - Needs access (amber) — key exists but BU not in scope, "Request" button
  - Missing (red) — no key in vault, "Request" button
  - Warning: "You can still launch as Pilot — the workflow will be saved but won't execute until all credentials are ready"
- Review summary: name, BU, description, type, pipeline, credentials, models
- "Launch as Pilot" button

**Success screen:**
- Checkmark + workflow name
- Two metrics: Time to create (elapsed since modal opened) and Time saved (from form)
- "View in registry" (closes modal, scrolls to + expands the new row) or "Add another"

### 4. Clicking an API Name

Opens a drawer showing:
1. **Key status** — masked key, active/rotating/expired, expiration date, data classification tags (PII/Financial/Internal), scope, permissions
2. **Workflows** — who's using this API, with time saved per workflow, clickable
3. **Recent activity** — last 3 credential events (rotations, access changes, failures)
4. **Comments** — team discussion per API, add with Enter

API names are clickable everywhere: block cards, block drawer, composer, pipeline visualizations.

### 5. Sidebar Panels

**Recent Runs** — latest workflow executions with status dots, timestamps, trigger type.

**API Connections** — two tabs:
- Connections: each API service with status dot and workflow count, clickable to open detail
- Activity: recent credential events
- Badge shows total connected count or flags issues

**Popular Across Teams** — workflows sorted by reuse count.

**Open Source Toolbox** — curated list of relevant open-source projects (Claude Code, Langfuse, n8n, Flowise, Rivet, Portkey, MCP Servers, Anthropic SDK). Each shows what it's useful for at Rillet, GitHub stars, and tags. Click to open repo.

---

## Work Lane Framework

Every automation request is classified into one of three lanes:

| Lane | Who benefits | What happens | Example |
|------|-------------|--------------|---------|
| **Product** | End customer directly | Advisor redirects to file a feature request | "Add auto-reconciliation to Rillet" |
| **Product Ops** | Customer indirectly | Build spec tagged as graduation candidate | "Customer onboarding data migration" |
| **Ops** | Internal team only | Standard workflow creation | "Weekly sales summary from HubSpot" |

## Risk & Reviewer (Auto-detected)

- **Low risk**: no external-facing outputs. No reviewer needed. All teams encouraged to automate.
- **Medium risk**: pipeline sends emails, updates Salesforce/Zendesk, or posts to Slack. Reviewer auto-assigned from BU.
- Users never see risk level or reviewer fields.

---

## Block System

33 blocks across 5 types:

**Sources (11):** HubSpot CRM, Zendesk Tickets, Google Sheets, Stripe Data, PDF/OCR, Google Drive, BambooHR, RSS Monitor, Mixpanel Events, Delighted NPS, Knowledge Base

**Processing (6):** Sort by Category, Flag if Off, Translate Between Apps, Pull Specific Info, Check Against Rules, Combine Sources

**AI (5):** Claude Summarizer (Sonnet 4), Claude Classifier (Haiku 4.5), Claude Drafter (Sonnet 4), Claude Extractor (Haiku 4.5), Claude Narrator (Sonnet 4)

**Outputs (8):** Slack Alert, Sheets Update, Email Send, PDF Report, Notion Page, Zendesk Update, Salesforce Update, Data Export

**Approval (3):** Manager Approves, Skip Review if Confident, Two People Approve

All non-API blocks labeled "Built-in" instead of technical terms.

### Block Drawer

Click any block to see:
- Model lifecycle (Active/Deprecated) with cost tier and pricing for AI blocks
- Data classification (PII/Financial/Internal) with scope and permissions
- "Used in Your Team" vs "Used by Other Teams" with role-specific relevance hints

### Model Selection

- **Sonnet 4** ($3/$15 per 1M tokens): Summarization, drafting, narration — tasks needing nuance
- **Haiku 4.5** ($0.80/$4 per 1M tokens): Classification, extraction — structured tasks where speed matters

Cost is tracked at the API key level, not per workflow. The registry shows which model each workflow uses so spending can be traced back from the Anthropic dashboard.

---

## Data Model

### Workflow (registry.json)

| Field | Type | Notes |
|-------|------|-------|
| id | number | Sequential |
| name | string | Max 50 chars |
| desc | string | Max 120 chars |
| bu | string | Must match existing BU |
| type | number | 1 = rule-based, 2 = AI-powered. Auto-detected. |
| risk | string | "Low" or "Medium". Auto-detected from output blocks. |
| status | string | "Live" or "Pilot" |
| saves | string | "N hrs/wk" or "N hrs/mo" |
| prompt_summary | string | 2-3 sentences: ingests → logic → output → approval |
| reused_by | string[] | BU names |
| reviewer | string | Auto-assigned for Medium risk. Empty for Low. |
| rewrite_rate | string | "N%" or "—%" |

### Prompt Versioning (per workflow)

| Field | Purpose |
|-------|---------|
| version | Semantic versioning (X.Y.Z) |
| lastUpdated | Date of last change |
| changelog | What changed |
| previousVersions | Version history |

### Output Quality (per workflow)

| Field | Purpose |
|-------|---------|
| totalRuns | How many times it's run |
| accepted | Output used as-is |
| edited | Output used with changes |
| rejected | Output discarded |
| trend | improving / stable / needs attention / new |

Displayed as plain language: "Working well", "Works, sometimes needs edits", "New — still being tested", "Needs improvement". No percentages shown to users.

### API Connections

Each credential tracks: name, service, masked key, status (active/rotating/expired), who added it, expiration, data classification (PII/Financial/Internal), access scope (which BUs), permissions (read/write), last audit date.

Each API has a comment thread for team discussion.

---

## Business Units

GTM / Sales, Finance, Customer Success, Implementation, People Ops, Product, Marketing, Legal, Procurement

Chief of Staff is a role, not a BU. The Founders Associate has no default BU and sees all workflows.

---

## Session & Identity

Users are logged in with name, initials, BU (if applicable), and role. BU is auto-filled in forms. Users without a BU (cross-functional roles) see the BU dropdown in the build form and the chat infers BU from keywords.

Current demo user: Haruka Takamori, Founders Associate (no BU — sees everything).

---

## Known Technical Limitations

These are acknowledged constraints of the demo, framed as a production roadmap:

1. **Keyword matching is brittle** — the advisor uses keyword-based block matching. Production would use an actual Claude API call.
2. **No persistence** — all state resets on page refresh. Production needs a backend.
3. **No runtime connection** — the registry is metadata only. Production would integrate with the execution layer.
4. **No prompt drift detection** — monitoring rewrite rate trends would catch degrading prompts. Currently static data.
5. **Single API key per service** — production should support per-environment keys and per-team scoping.
6. **No cost tracking** — token usage can't be estimated from the UI. Requires runtime instrumentation.

---

## 16 Workflows in Demo

| ID | Name | BU | Status | Saves |
|----|------|----|--------|-------|
| 1 | Deal Summary Generator | GTM / Sales | Live | 2.5 hrs/wk |
| 2 | Invoice Discrepancy Flagger | Finance | Live | 4 hrs/wk |
| 3 | Customer Health Score Digest | Customer Success | Live | 3 hrs/wk |
| 4 | SOW Clause Extractor | Implementation | Live | 1.5 hrs/wk |
| 5 | Weekly Metrics Narrator | Finance | Live | 2 hrs/wk |
| 6 | Support Ticket Classifier | Customer Success | Live | 5 hrs/wk |
| 7 | Competitive Intel Summarizer | GTM / Sales | Live | 3 hrs/wk |
| 8 | Expense Policy Checker | Finance | Pilot | 2 hrs/wk |
| 9 | Onboarding Checklist Generator | People Ops | Live | 1.5 hrs/wk |
| 10 | Board Deck Data Puller | Finance | Pilot | 6 hrs/mo |
| 11 | RFP Response Drafter | GTM / Sales | Live | 8 hrs/wk |
| 12 | Contract Renewal Risk Scorer | Customer Success | Live | 3 hrs/wk |
| 13 | API Endpoint Provisioner | Product | Pilot | 4 hrs/wk |
| 14 | Investor Update Drafter | Finance | Live | 5 hrs/mo |
| 15 | GTM Efficiency Analyzer | GTM / Sales | Pilot | 3 hrs/wk |
| 16 | Cross-Functional Standup Digest | GTM / Sales | Live | 4 hrs/wk |

---

## Design Principles

1. **Don't show what users can't act on.** If it doesn't help the user make a decision or take an action, cut it. This is the single rule that drove every UI decision.
2. **No jargon.** The user is an operator, not an engineer. "Rule-based" not "Type 1". "Working well" not "83% accepted". "Translate Between Apps" not "Schema Mapper".
3. **Chat first, form second.** Discover before building. Check if it already exists before creating new. Don't start with a blank form — start with a conversation.
4. **Auto-detect everything possible.** Risk, reviewer, type, BU. Don't ask the user what the system can figure out from the pipeline.
5. **Every workflow is a data point in the work graph.** Each automation added to the registry makes the company's operational structure more legible. The value compounds.
