CLAUDE.md — Rillet AI Registry
You are working inside the Rillet AI Workflow Registry — a shared catalog of all AI-assisted workflows running across business units at Rillet.
This file tells you exactly how to read, update, and maintain this registry. Read it fully before doing anything.

What This Repo Is

data/registry.json — the source of truth. All workflow entries live here.
index.html — the UI. Loads registry.json at runtime and renders it.
CLAUDE.md — this file. You read it every session.

Your job: keep registry.json accurate, add new workflows when asked, and write progress notes back to the second brain (Khoj) after every change.

Before Starting Any Task

Read data/registry.json to understand what workflows currently exist
Check for the workflow(s) relevant to the request
If querying Khoj MCP is available, search for prior context on the workflow or domain before making changes

bash# Always start with
cat data/registry.json | jq '[.[] | {id, name, bu, status}]'

Data Schema
Each entry in registry.json must have all of these fields:
FieldTypeRulesidnumberSequential. Next ID = max(existing ids) + 1namestringTitle case. Max 50 chars.descstringOne sentence. Max 120 chars.bustringMust match an existing BU or create a new one with justificationtypenumber1 = deterministic/rule-based, 2 = generative/LLMriskstring"Low" or "Medium" onlystatusstring"Live" or "Pilot" onlysavesstringFormat: "N hrs/wk" or "N hrs/mo". Use "—" if unknowndaynumberDay offset when workflow was added. Use current max + 5 if newprompt_summarystring2-3 sentences: inputs → logic → output → human review gate. See format below.reused_bystring[]Array of BU names reusing this workflow. [] if none.reviewerstringRole title of human reviewer. "TBD" if not yet assigned.rewrite_ratestringFormat: "N%". Use "—%" if unknown.
prompt_summary Format
Always follow this structure:
[What it ingests]. [What AI logic / rules it applies and what it outputs]. [Who reviews before any action is taken].
Example:

"Parses PDF invoices via OCR, extracts line items, and cross-references against approved PO database. Flags unit price variance >2%, quantity mismatches, and unapproved line items. Human reviewer required before any AP action."

Never start with "This workflow". Never use vague language like "processes data".

How to Add a New Workflow
When someone asks you to add a workflow, collect or infer:

Name, BU, description, how it works, who reviews it, estimated time savings

Then:

Read the current registry to get the next id and day values
Write a proper prompt_summary following the format above
Add the new entry to data/registry.json
Confirm the JSON is valid
Write a progress note (see "After Every Change" below)

bash# Validate JSON after editing
python3 -c "import json; json.load(open('data/registry.json')); print('✓ Valid JSON')"

How to Update an Existing Workflow
Common update requests:
Promote Pilot → Live:
bash# Find the workflow, change "status": "Pilot" → "status": "Live"
# Also update rewrite_rate and saves if now known
Add a BU to reused_by:
bash# Add the BU string to the reused_by array
# Example: "reused_by": ["Finance"] → "reused_by": ["Finance", "Legal"]
Update saves or rewrite_rate:
bash# Update the field with the new value
# Always use format "N hrs/wk" or "N%"
Always validate JSON after editing.

After Every Change
After completing any task, do all three:
1. Confirm the change
✓ Updated: [workflow name]
  Field: [what changed]
  Value: [old] → [new]
2. Write a progress note to Khoj (if MCP available)
Format:
Tag: #progress #[domain-tag]
Content: "[Workflow name] — [what changed] — [status]. [Any relevant context for future engineers.]"
Timestamp: [now]
Example:
Tag: #progress #finance
Content: "Invoice Discrepancy Flagger — promoted Pilot → Live. Rewrite rate settled at 5%. Controller is the reviewer. Reused by Procurement."
3. Show a summary diff
REGISTRY UPDATE
───────────────
Workflow: Invoice Discrepancy Flagger (#2)
Changed:  status: "Pilot" → "Live"
          rewrite_rate: "—%" → "5%"
Khoj:     ✓ progress note written · #finance #progress

Business Units (Existing)
Only use these unless explicitly asked to create a new one:

GTM / Sales
Finance
Customer Success
Implementation
Chief of Staff
People Ops
Product
Marketing
Legal
Procurement


Type Classification Guide
Type 1 — Deterministic / Rule-based

Uses fixed rules, thresholds, or regex matching
Output is predictable given same input
Examples: invoice matching, expense policy checking, ticket routing by keyword

Type 2 — Generative / LLM-based

Uses an LLM to generate, summarize, or classify with judgment
Output varies; requires more human review
Examples: deal summaries, RFP drafts, health score narratives

If unclear, ask. Do not guess.

Risk Classification Guide
Low — Output is informational or advisory. No financial, legal, or customer-facing action taken without human sign-off.
Medium — Output directly triggers or informs a consequential action (payment, customer communication, board-level data). Reviewer gate is mandatory.
Never assign risk based on the BU alone — assess the workflow's output and downstream action.

Things You Must Never Do

Never delete a workflow entry (mark as "status": "Archived" if needed — add this status only when explicitly asked)
Never change a workflow's id
Never set reused_by to a BU that doesn't exist in the BU list without flagging it
Never write an invalid prompt_summary (no vague language, must have reviewer gate)
Never leave reviewer as "" — use "TBD" if unknown
Never commit broken JSON


Example Session
User: "The RFP Response Drafter is now being reused by Legal too. Update it."
You:

Read registry.json, find id 11 (RFP Response Drafter)
Check current reused_by: ["Customer Success", "Implementation"]
Add "Legal" → ["Customer Success", "Implementation", "Legal"]
Validate JSON
Output:

REGISTRY UPDATE
───────────────
Workflow: RFP Response Drafter (#11)
Changed:  reused_by: added "Legal"
          Now: ["Customer Success", "Implementation", "Legal"]
Khoj:     ✓ progress note written · #gtm #progress

Khoj MCP Configuration
If the Khoj MCP server is connected, use it for:

Before tasks: search_documents(query="[workflow name] [domain]") — find prior context
After tasks: write a structured progress note (see format above)

Khoj is the second brain for this registry. Every change you make should be reflected there so the next engineer starting a new session has full context without asking anyone.
