---
name: profit-engine
description: Sales-operations toolkit for a services business. Use when the user needs to build a margin-safe customer quote, run a weekly accounts-receivable (AR) pipeline / invoice aging review, draft cadence-aware lead follow-ups, or run a sales-coaching roleplay, script review, or call debrief. Triggers on quote, pricing, margin, floor price, invoice, AR aging, collections, pipeline review, lead follow-up, cadence, objection handling, and sales coaching.
license: Apache-2.0
metadata:
  author: ghostmachine-ai
  version: "1.0"
---

# Profit Engine

A reusable toolkit for running the revenue side of a services business: quoting, cash collection, lead follow-up, and sales coaching. Each workflow reads from a shared `Profit-Engine/` data workspace and follows a documented reference procedure.

## Data layout

The workflows expect a `Profit-Engine/` directory in (or near) the working directory with this structure. If it is missing, tell the user which files are needed before proceeding.

```
Profit-Engine/
├── 01-cost-sheet/
│   ├── price-book.json      # [{sku, name, unit, cogs, list_price, floor_price, active}]
│   └── margin-rules.md      # margin tiers, floor-price formula, discount approval policy
├── 02-pipeline/
│   ├── invoices.json        # [{id, customer, amount, issue_date, due_date, status, days_outstanding, contact}]
│   ├── aging-rules.md       # AR buckets, escalation triggers, late-fee policy
│   └── weekly-snapshot.md   # fillable template for the weekly review
├── 03-crm/
│   ├── leads/<lead-id>.md   # one file per lead (frontmatter + activity log)
│   └── cadence-log.json     # {entries: [{lead_id, date, channel, summary, outcome, next_due}]}
└── 04-coach/
    ├── ghost-persona.md     # sales voice & tone guide
    ├── objection-scripts.md # common objections + scripted responses
    └── product-translator.md# tech-speak → customer-benefit translations
```

## Workflows

Pick the workflow that matches the user's request and follow its reference file step by step. Load only the one you need.

| User wants to… | Read |
|----------------|------|
| Build a customer quote priced above margin floor | [references/cost-quote.md](references/cost-quote.md) |
| Run the weekly AR / invoice-aging pipeline review | [references/pipeline-review.md](references/pipeline-review.md) |
| Draft next follow-up actions for one or all leads | [references/follow-up.md](references/follow-up.md) |
| Roleplay, review a script, prep, or debrief a sales call | [references/sales-coach.md](references/sales-coach.md) |

## Notes

- Never quote or approve a price below a SKU's `floor_price` without explicit sign-off per `margin-rules.md`.
- After a workflow proposes data changes (a new cadence entry, an invoice status update), the user applies them — surface a copy-paste-ready snippet and remind them to save it.
- Stay in the ghost persona voice (`04-coach/ghost-persona.md`) for any customer-facing copy the skill drafts.
