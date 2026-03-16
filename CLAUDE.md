CLAUDE.md — Rillet AI Ops

Internal AI workflow registry for Rillet. Catalog, govern, and reuse AI automations across every team.

What This Repo Is

- `data/registry.json` — source of truth for all workflow entries
- `index.html` — single-page UI, loads registry.json at runtime
- `CLAUDE.md` — this file

What This Platform Is (and Isn't)

**This is a registry** — a catalog and governance layer for AI automations. It is NOT a runtime. It does not execute workflows.

How it connects to actual automations:
- **Code lives elsewhere** (GitHub, internal tools, no-code platforms like n8n/Flowise). The registry links to it. Each workflow entry is metadata about the automation — what it does, who owns it, how well it works.
- **Rillet's product AI agents** (accrual, audit, P&L flux, board decks) run inside the product. This registry sits alongside them as the management layer — tracking what exists, who owns it, and how well it works.
- **No one uploads code to the registry.** The blocks represent logical components (data source, AI processing, output destination), not executable code. In production, the block composition would map to a runtime configuration.

Architecture Overview

1. **Session & Identity** — Users are logged in with name, BU, and role (`SESSION_USER`). BU is auto-filled if the user belongs to one. Founders Associate has no BU (cross-functional role, sees all workflows).
2. **Automation Advisor (Chat)** — Full-screen Claude-style chat interface (Step 1). Gathers context across turns, checks for existing workflows, classifies the work lane, decomposes multi-workflow problems, generates build specs.
3. **Work Lane Classification** — Every request is classified as Product, Product Ops, or Ops.
4. **Existing Workflow Matching** — Before building anything new, the advisor searches the registry for workflows that might already solve the problem. Shows matches in a split-view panel so the user can evaluate without losing their conversation.
5. **Block Composer (Step 2)** — Form fields + searchable block catalog + staged pipeline builder. Pre-populated from the advisor chat.
6. **Credential Gate (Step 3)** — Before launch, checks that all required API keys exist and the user's BU has access. Shows ready/needs-access/missing status per credential with request buttons.
7. **Review & Launch (Step 3)** — Summary view + credential gate. Launches as Pilot. Success screen shows time-to-creation and time-saved metrics.
8. **Block Drawer** — Right sidebar showing block details, model lifecycle (active/deprecated), cost tier (Low cost/Standard), data classification, and usage by team.
9. **API Detail View** — Click any API name to see: key status, workflows using it, recent activity, and team comments.
10. **API Connections Panel** — Sidebar panel with two tabs: Connections (all APIs with status dots, clickable) and Activity (recent credential events).
11. **Registry Table** — Main workflow table with search, filters, security grade, expandable detail rows with reliability indicators and comments.

Work Lane Framework

Every automation request is classified into one of three lanes:

**Product** — End customer interacts with this directly. Should be a product feature, not a workflow. The advisor redirects.
**Product Ops** — Between product and ops. Supports product experience but isn't core. Graduation candidate — track usage, may become a feature.
**Ops** — Pure internal team efficiency. Customer never sees it. This is what the registry is for.

Risk & Reviewer (auto-detected, not user-facing):
- **Medium**: Pipeline sends emails, updates Salesforce/Zendesk, or posts to Slack. Reviewer auto-assigned from BU.
- **Low**: Everything else. No reviewer needed. All teams encouraged to automate.

Automation Advisor (Chat System)

Full-screen chat interface, Claude/ChatGPT-style. Messages centered at 640px max-width, input pinned to bottom.

The advisor:
1. Greets the user by first name
2. Gathers context across multiple turns — tracks sources, processing, outputs, time savings
3. Checks for existing workflows that match — shows them in a split-view panel before building new
4. Classifies the work lane (Product/Product Ops/Ops)
5. Detects multi-workflow problems — splits compound requests into separate build specs
6. Generates build specs when ready (1+ blocks + time savings)
7. Transitions to build mode (Step 2) when user clicks "Build new workflow"

The advisor does NOT:
- Ask about BU (known from session)
- Ask about risk/reviewer (auto-detected from blocks)
- Nag about output destination if the user doesn't mention one
- Dump everything after one message

Time matching handles natural language: "about 3 hours a week", "takes 4 hours per month", "half a day every week".

Block System

33 blocks across 5 types:
- **Sources** (11): HubSpot, Zendesk, Sheets, Stripe, PDF/OCR, Drive, BambooHR, RSS, Mixpanel, Delighted, Knowledge Base
- **Transforms** (6): Keyword Classifier, Threshold Checker, Schema Mapper, Regex Extractor, Policy Validator, Signal Aggregator
- **AI** (5): Claude Summarizer (Sonnet 4), Classifier (Haiku 4.5), Drafter (Sonnet 4), Extractor (Haiku 4.5), Narrator (Sonnet 4)
- **Outputs** (8): Slack Alert, Sheets Update, Email Send, PDF Report, Notion, Zendesk Update, Salesforce Update, JSON Export
- **Gates** (3): Manager Review, Confidence Gate, Dual Review

Block drawer shows:
- Model lifecycle status (Active/Deprecated) with cost tier (Low cost = Haiku, Standard = Sonnet) and pricing
- Data classification (PII/Financial/Internal) with scope and permissions
- Usage split: "Used in Your Team" vs "Used by Other Teams" with role-specific relevance hints

API names are clickable throughout the UI — opens the API detail view.

API Connections

Sidebar panel with two tabs:
- **Connections** — Each API service with status dot (green/amber/red) and workflow count. Clickable to open detail.
- **Activity** — Recent credential events (rotations, access changes, failures).

API Detail View (in drawer):
- Key status: masked key, active/rotating/expired, expiration date, data tags, scope
- Workflows using this API (clickable)
- Recent activity (3 entries)
- Comments — team discussion per API (add comments with Enter)

Badge shows total connected APIs count, or flags issues (rotating/expired keys).

Prompt Versioning

Each workflow tracks:
- Current prompt version (semantic versioning: X.Y.Z)
- Last updated date
- Changelog (what changed in this version)
- Previous versions list

Shown in the workflow detail row as "Last updated: [date]".

Model Lifecycle

Each AI model tracks:
- Status: `active`, `deprecated`, or `sunset`
- EOL date and successor model (when applicable)
- Cost tier: Low cost (Haiku: $0.80/$4 per 1M tokens) or Standard (Sonnet: $3/$15 per 1M tokens)

When a model is deprecated, all workflows using it can be identified via the block system.

Output Quality

Each workflow tracks acceptance rates across all runs:
- Accepted (used as-is), Edited (used with changes), Rejected (discarded)
- Displayed as plain-language reliability: "Reliable — 83% used as-is", "Too early to tell — only 8 runs", "Needs work — only 50% used as-is"

Known Technical Limitations (for production roadmap)

1. **Keyword matching is brittle** — The advisor uses keyword-based block matching. A production system would use an actual Claude API call to understand user intent.
2. **No persistence** — All state resets on page refresh. Production needs a backend with auth and database.
3. **No real runtime connection** — The registry is metadata only. Production would integrate with the actual workflow execution layer.
4. **No prompt drift detection** — Monitoring rewrite rate trends over time would catch degrading prompts. Currently static.
5. **Single API key per service** — Production should support per-environment keys (prod/staging) and per-team scoping.
6. **No cost tracking** — Token usage cannot be estimated from the UI. Requires runtime instrumentation (Langfuse, Portkey, or custom logging).

Data Schema

Each entry in registry.json:
- `id` (number) — Sequential, next = max + 1
- `name` (string) — Title case, max 50 chars
- `desc` (string) — One sentence, max 120 chars
- `bu` (string) — Must match existing BU
- `type` (number) — 1 = Irreversible/High Stakes (requires human sign-off), 2 = Reversible/Low Stakes (AI can own or draft). Auto-detected from blocks.
- `risk` (string) — "Low" or "Medium". Auto-detected from output blocks.
- `status` (string) — "Live" or "Pilot"
- `saves` (string) — "N hrs/wk" or "N hrs/mo"
- `day` (number) — Day offset when added
- `prompt_summary` (string) — 2-3 sentences: ingests → logic → output → review gate
- `reused_by` (string[]) — BU names reusing this workflow
- `reviewer` (string) — Auto-assigned for Medium risk. Empty for Low risk.
- `rewrite_rate` (string) — "N%" or "—%"

Business Units

GTM / Sales, Finance, Customer Success, Implementation, People Ops, Product, Marketing, Legal, Procurement

Things You Must Never Do

- Never delete a workflow entry (use "Archived" status if needed)
- Never change a workflow's id
- Never set reused_by to a BU that doesn't exist
- Never write vague prompt_summary language
- Never commit broken JSON
