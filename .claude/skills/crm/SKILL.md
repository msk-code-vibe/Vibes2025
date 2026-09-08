---
name: crm
description: Manage a personal/small-business CRM stored in this repo at crm/CRM.xlsx (Contacts, Interactions, Deals sheets). Use when the user wants to add or look up a contact, log a call/email/meeting, track a deal's stage and value, find follow-ups that are due, or otherwise read/update their CRM data. Trigger on "CRM", "add a contact", "log an interaction", "update the deal", "who do I need to follow up with", etc.
---

# CRM skill

Manages a lightweight CRM stored as a spreadsheet at `crm/CRM.xlsx` in this
repository. The workbook is the source of truth — always read the current
file before editing it, and use the `xlsx` skill's tooling (openpyxl /
pandas + `recalc.py` if formulas are involved) rather than hand-rolling
spreadsheet edits.

## Data model (`crm/CRM.xlsx`)

**Contacts** — Name, Company, Email, Phone, Status
(`Lead|Prospect|Customer|Churned`), Tags, Notes, Created Date, Last Contact
Date.

**Interactions** — Date, Contact Name, Type (`Call|Email|Meeting|Note`),
Summary, Next Steps, Follow-up Date.

**Deals** — Deal Name, Contact Name, Stage
(`Lead|Qualified|Proposal|Negotiation|Won|Lost`), Value, Expected Close
Date, Notes.

Row 2 in each sheet is a clearly-marked "(example)" row — skip it in
reports/summaries, and leave it in place unless the user asks to remove it.
Data validation dropdowns already exist for Status/Type/Stage columns; keep
new values within that set unless the user explicitly wants to extend it
(update both the data validation list and this file if so).

## Operations

- **Add a contact**: append a row to `Contacts`, filling `Created Date` and
  `Last Contact Date` with today's date unless told otherwise.
- **Log an interaction**: append a row to `Interactions`. If it mentions a
  contact not yet in `Contacts`, ask whether to add them first (or add them
  if the user's intent is clear). Update that contact's `Last Contact Date`
  in `Contacts` to match.
- **Track a deal**: append or update a row in `Deals`. Match existing deals
  by `Deal Name` (and `Contact Name` if ambiguous) before creating a
  duplicate.
- **Follow-ups due**: filter `Interactions` where `Follow-up Date` is today
  or earlier and report Contact Name, Follow-up Date, and Next Steps.
- **Pipeline / status summaries**: group `Deals` by `Stage` (or `Contacts`
  by `Status`) and summarize counts/values.

## Implementation notes

- Use Python (`openpyxl` for targeted edits/appends, `pandas` for bulk
  reads/filters) against `crm/CRM.xlsx`. Preserve existing formatting —
  don't rewrite the whole sheet when appending a row.
- This workbook has no formulas, so `recalc.py` isn't required after edits;
  if formulas are later added, run it per the `xlsx` skill's rules.
- After editing, briefly confirm what changed (e.g. "Added contact Jane
  Doe to Contacts" or "Logged a call with Acme Corp, follow-up 2026-02-01").
- This file lives in the git repo, not on any particular machine's desktop.
  If the user wants it on their local Desktop too, that's a manual export/
  sync step outside this skill — don't assume a local file path exists.
