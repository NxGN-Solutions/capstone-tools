# Capstone CLI Documentation — Start Here

The Capstone CLI (`cap`) manages and analyzes data in the Capstone platform.
This folder is the agent- and human-facing documentation for it.

## Connect First

Before any discovery or data command, establish a connection — empty results
almost always trace back to auth/tenant, not missing data:

```bash
cap config set api-url <URL>   # 1. Point at the Capstone server
cap auth login                 # 2. Identity OIDC login (opens browser; --device for SSH)
cap auth whoami                # 3. Confirm identity, tenant, and accessible tenants
```

If something looks wrong (or every command returns empty), run `cap status` and
`cap auth doctor` — `auth doctor` diagnoses authentication state and prints the
recovery commands to run.

## Where to Go Next

| I want to... | Read |
|--------------|------|
| Get the agent on-ramp (shell detection, setup, workspace cache, error contract) | [dist-CLAUDE.md](./dist-CLAUDE.md) |
| Teach an agent the CLI (concept glossary, domain routing, command patterns, error handling) | [SKILL.md](./SKILL.md) |
| Follow a step-by-step task workflow | [recipes/README.md](./recipes/README.md) |
| Build a great-looking dashboard (visual standard, brand tokens, widget variants) | [reference/dashboard-design-system.md](./reference/dashboard-design-system.md) |
| Look up a specific command | [reference/commands.md](./reference/commands.md) |
| Translate syntax across Windows/macOS/Linux shells | [reference/platform-guide.md](./reference/platform-guide.md) |
| Map vocabulary (KPI → Metric, site → Org Node, …) | [reference/glossary.md](./reference/glossary.md) |

## Discovering Commands at Runtime

The CLI is self-describing — prefer these over guessing command names or flags:

```bash
cap schema --json     # Full machine-readable command index + automation contracts
cap concepts --json   # Compact Capstone domain concepts for planning
cap meta lookups list # All known lookup names across CLI domains (then: meta lookups get)
```

`cap schema --json` is authoritative; it includes exit-code semantics and the
`upsertIdentity` contract used for Excel uploads.

## Notes

- Most reporting commands require `--data-interval` and `--periods`.
- `notifications templates` (`create`/`get`/`list`/`save`/`delete`/`download-excel`/`upload-excel`)
  administers notification templates, and `notifications rules`
  (`create`/`get`/`list`/`save`/`delete`/`download-excel`/`upload-excel`) administers
  notification rules; `create` and `save` accept the output of `get --json` as is;
  see [reference/commands.md](./reference/commands.md).
- A notification template's `eventType` makes it an event template for exactly that event, for
  example `"eventType": {"id": 3, "name": "Input Value Captured"}` (the `id` is what is saved:
  2 Change Request Validated, 3 Input Value Captured, 4 Validation Completed, 5 Data Lock Applied).
  A null or omitted `eventType` makes it a scheduled reminder. An Event Driven rule needs an event
  template, and a Scheduled or Both rule needs a scheduled reminder. An event template cannot
  contain `{{DataCaptureTable}}`, `{{ValidationTable}}` or `{{ChangeRequestTable}}`.
  `{{ActionLink}}` is a styled link for the email body only; a subject that contains it is
  rejected on `save` and on `upload-excel`. On `save`, the root `name`, `subject` and `body` are
  the text for the request language, so an edit to that language inside `translations[]` is
  ignored; edit the root fields instead.
- A scheduled notification rule fires on either a `cronExpression` or period-end
  offsets, never both. Period-end offsets are `timePeriodType` (Month, Quarter or
  Year; quarters and years follow the tenant's fiscal calendar),
  `periodEndOffsetDays` (signed days from the period's last day, -31 to 31, e.g.
  `[-5, -3, 0]`) and `fireTime` (`HH:mm`), all in the rule's `timeZoneId`. A
  reminder that fires after period end still chases the period that just ended.
  `rules list` shows each rule's schedule; `rules get` shows the fields.
- Capture and report templates have `showRowsWithValuesFilter` (let viewers
  switch between all rows and rows with values; `showOnlyRowsWithValues` is the
  default and applies as-is when the flag is false).
- Capture templates have `showPendingFilter` (offer a "Show pending only" toggle
  on the capture and validation grids) and `showOnlyPending` (open with it on:
  capture shows only values still to capture, validation only values waiting
  for the viewing user). Reminder email links open templates with it on.
