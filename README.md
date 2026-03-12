# AI Registry

Shared AI workflow registry for cross-BU visibility and reuse.

## Overview

Central catalog of all AI-assisted workflows running across business units. Tracks what's live, what's in pilot, who owns review, and where workflows are being reused across teams.

Currently tracking **12 workflows** across **6 business units**.

## How it works

Static site served via GitHub Pages. The UI loads workflow data from `data/registry.json` on page load and renders a filterable, searchable, sortable table. No backend, no build step.

Features:
- Filter by business unit, status, risk level, and Type 1/2 classification
- Full-text search across workflow names and descriptions
- Expandable rows with prompt details, review gates, and reuse info
- Live stats bar (total workflows, hours saved, avg rewrite rate)

## Data schema

Each workflow entry in `data/registry.json`:

| Field            | Type     | Description                                      |
|------------------|----------|--------------------------------------------------|
| `id`             | number   | Unique identifier                                |
| `name`           | string   | Workflow name                                    |
| `desc`           | string   | One-line description                             |
| `bu`             | string   | Business unit owner                              |
| `type`           | number   | `1` = deterministic/rule-based, `2` = generative |
| `risk`           | string   | `Low` or `Medium`                                |
| `status`         | string   | `Live` or `Pilot`                                |
| `saves`          | string   | Estimated time savings (e.g., `"4 hrs/wk"`)      |
| `day`            | number   | Day offset for timeline/sorting                  |
| `prompt_summary` | string   | How the prompt/pipeline works                    |
| `reused_by`      | string[] | Other BUs reusing this workflow                  |
| `reviewer`       | string   | Human reviewer role                              |
| `rewrite_rate`   | string   | Percentage of outputs requiring human edits      |

## Adding a workflow

1. Edit `data/registry.json`
2. Add a new object with all fields from the schema above
3. Assign the next sequential `id`
4. Open a pull request
5. CoS reviews for accuracy and classification
6. Merge to `main` → auto-deploys to Pages

## Deployment

Deployed automatically via GitHub Actions on push to `main`. No build step required.

To set up from scratch:
1. Create a GitHub repo
2. Push this directory to `main`
3. Go to **Settings → Pages → Source: GitHub Actions**
4. First deploy triggers automatically
