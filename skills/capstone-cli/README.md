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
- `notifications templates` (`create`/`get`/`list`/`save`/`delete`/`download-excel`/`upload-excel`/`tokens`/`lint`)
  administers notification templates, and `notifications rules`
  (`create`/`get`/`list`/`save`/`delete`/`download-excel`/`upload-excel`/`lint`) administers
  notification rules; `create` and `save` accept the output of `get --json` as is;
  see [reference/commands.md](./reference/commands.md#notification-templates).
  `notifications rules lint` checks a rule without saving it and, with `--json`, returns its
  next five sends (exit code 2 on errors; see
  [reference/commands.md](./reference/commands.md#notification-rules)).
- To check what a rule sent: `notifications rules runs <id>` lists its runs (people found and
  emails queued), `notifications delivery-log list --rule <id>` lists the emails, and
  `notifications rules recipients <id>` lists who the rule can reach. `security users get
  <id-or-email>` shows a user's template grants and "Authorize all" scopes; see
  [reference/commands.md](./reference/commands.md#notification-delivery-log).
- A notification template is **Written for** one event, one event's summary, or its type's digest.
  Its `eventType` names the event, for example `"eventType": {"id": 3, "name": "Input Value Captured"}`
  (the `id` is what is saved: 2 Change Request Validated, 3 Input Value Captured, 4 Validation
  Completed, 5 Data Lock Applied, 6 AI Key Failing, 7 AI Key Recovered, and the summaries 8 Input
  Value Captured Summary, 9 Validation Completed Summary, 10 Change Request Validated Summary,
  11 Data Lock Applied Summary). A null or omitted `eventType` makes
  it the type's digest: Pending capture digest, Pending validation digest, Pending change requests
  digest or Data feed changes digest. An Event Driven rule needs an event template (or, when it
  sends a summary, that event's summary template), and a Scheduled rule needs a digest. `notifications templates tokens` lists the tokens each
  one fills; `notifications templates lint` checks a template without saving it (exit code 2 on
  errors). A token the template's Written for does not fill, a digest table in an event template,
  and `{{ActionLink}}` or a table in the subject are refused on `save` and on `upload-excel`.
  While any rule uses a template, its `notificationType` and `eventType` cannot change.
  `isActive` ("Available to rules") only decides whether the rule picker offers the template;
  rules that use it keep sending. Templates have no `isDefault`. A refused save answers the
  field-keyed `{ "errors": { "field": ["..."] } }`. On `save`, the root `name`, `subject` and
  `body` are the text for the request language, so an edit to that language inside
  `translations[]` is ignored; edit the root fields instead.
- A scheduled notification rule fires on either a `cronExpression` or period-end
  offsets, never both. Period-end offsets are `timePeriodType` (Month, Quarter or
  Year; quarters and years follow the tenant's fiscal calendar),
  `periodEndOffsetDays` (signed days from the period's last day, -31 to 31, e.g.
  `[-5, -3, 0]`) and `fireTime` (`HH:mm`), all in the rule's `timeZoneId`. A
  reminder that fires after period end still chases the period that just ended.
  `rules list` shows each rule's schedule; `rules get` shows the fields. Trigger
  type Both is retired: existing Both rules became Scheduled, and a rule saved or
  uploaded as Both is refused. A refused rule save reports every issue at once as
  `field: message` lines (the field-keyed `{ "errors": ... }` body); `upload-excel`
  reports each issue on its row and column and saves the other rows.
- Notification type **Data Feed Changes** (`"notificationType": {"id": 5, "name": "Data Feed
  Changes"}`) mails the input values and narratives a data feed changed or cleared since the
  rule's previous send. Its templates are written for the Data feed changes digest (null
  `eventType`), use `{{DataFeedChangesTable}}` and no other table, and the seeded **Data Feed Changes** template is
  ready to use. Its rules must be Scheduled, with sends at most 120 days apart (`* * * * *` sends
  every minute). Recipients are the Data Owners in scope (a data permission that grants no access;
  they also need View), plus, when the rule's `notifyReplacedValueSavers` is true (the default,
  Excel column "Tell People Whose Value Was Replaced"), the person whose value the feed replaced,
  once per change. A template's type cannot change while a rule uses it, so it cannot move to or
  from Data Feed Changes.
- An Event Driven rule on Input Value Captured, Validation Completed, Change Request Validated or
  Data Lock Applied has **Send** (`priority`): `{"id": 0, "name": "Each time"}` mails each event within
  about a minute; `{"id": 1, "name": "Summary"}` mails each recipient one summary of the events since
  the previous send, on the rule's schedule (the same `cronExpression` or period-end fields and
  `timeZoneId` as a scheduled rule, sends at most 120 days apart). AI key rules always send each time.
  `rules list` shows it in the Send column. Changing Send restarts the rule from the save; earlier
  events are not sent again. Events are
  logged after each save, so a restart does not lose them; an each-time event older than 24 hours is
  skipped with `EVENT_TOO_OLD` in `notifications rules runs`. Rules that were *Digest* are now *Each
  time*, and Digest Group is gone.
- The notification rules workbook carries each rule's scope in the `Discipline` and `Org Node`
  columns (full paths) and its Send in the `Send` column (`Each time` or `Summary`; blank is Each
  time). A blank scope cell clears the value; a workbook without these columns keeps the stored
  scope. An older workbook's `Priority` and `Digest Group` columns are ignored.
- Capture and report templates have `showRowsWithValuesFilter` (let viewers
  switch between all rows and rows with values; `showOnlyRowsWithValues` is the
  default and applies as-is when the flag is false).
- Capture templates have `showPendingFilter` (offer a "Show pending only" toggle
  on the capture and validation grids) and `showOnlyPending` (open with it on:
  capture shows only values still to capture, validation only values waiting
  for the viewing user). Reminder email links open templates with it on.
