# Follow-Up

Review leads and draft next follow-up actions based on cadence history.

## Inputs

- `Profit-Engine/03-crm/leads/<lead-id>.md` — the lead record(s).
- `Profit-Engine/03-crm/cadence-log.json` — outreach history.
- `Profit-Engine/04-coach/ghost-persona.md` — voice for drafted messages.
- From the user: a specific `lead-id`, or `all`.

## Steps

1. If a specific `lead-id` was given, read `leads/<lead-id>.md`. If `all`, list every file in `leads/` (skip `_template.md`) and process each.
2. Read `cadence-log.json` and filter entries for the relevant lead(s).
3. For each lead:
   a. Determine the current stage from the frontmatter.
   b. Find the most recent cadence entry and its `next_due` date.
   c. Check whether follow-up is overdue (`next_due` < today).
4. Draft a follow-up message in the ghost persona:
   - Email if the last channel was email and a response is still plausible.
   - Phone/LinkedIn if email has been tried twice with no response.
   - Adjust tone by stage: warmer for prospects, more direct in negotiation.
5. Output per lead:

   **Lead**: [name] @ [company] | Stage: [stage] | Last touch: [date] | Next due: [date]
   **Suggested channel**: [email / phone / LinkedIn]
   **Draft message**:
   > [draft follow-up text]

   **Cadence-log entry to add** (copy-paste ready):
   ```json
   { "lead_id": "...", "date": "...", "channel": "...", "summary": "...", "outcome": "pending", "next_due": "..." }
   ```

6. After the user approves, remind them to append the entry to `cadence-log.json`.
