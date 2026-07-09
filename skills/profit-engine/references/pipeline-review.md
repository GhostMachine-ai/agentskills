# Pipeline Review

Produce a weekly accounts-receivable pipeline snapshot.

## Inputs

- `Profit-Engine/02-pipeline/invoices.json` — all invoice records.
- `Profit-Engine/02-pipeline/aging-rules.md` — bucket definitions and escalation triggers.
- `Profit-Engine/02-pipeline/weekly-snapshot.md` — output template.

## Steps

1. Read `invoices.json` to load all invoices.
2. Read `aging-rules.md` for bucket boundaries and escalation triggers.
3. Classify each invoice into its aging bucket based on `days_outstanding`.
4. Calculate totals per bucket and the overall AR balance.
5. Identify escalation triggers:
   - Invoices > $10,000 with 15+ days outstanding → flag immediately.
   - Customers with 2+ open invoices in any bucket → flag for relationship review.
   - Invoices crossing from Overdue into Seriously Overdue → note for a collections letter.
6. Suggest specific next actions per at-risk account, referencing the `contact` field.
7. Fill in and output the snapshot using the `weekly-snapshot.md` template — fill every field.

## Output

End the snapshot with a prioritized **Actions Due Next Week** list, most urgent first.

## After review

Remind the user to:
- Update invoice statuses in `invoices.json` after any payment is received.
- Save the completed snapshot as `weekly-snapshot-YYYY-MM-DD.md` before the next run.
