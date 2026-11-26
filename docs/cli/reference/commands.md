# Command Quick Reference

> Quick lookup for Capstone CLI commands organized by what you want to do.

---

## Common Flags

These flags work across most commands:

| Flag | Description | Example |
|------|-------------|---------|
| `--json` | Output as JSON (machine-readable) | `cap model metrics list --json` |
| `--limit N` | Limit results to N items | `cap model metrics list --limit 10` |
| `--offset N` | Skip first N results | `cap model metrics list --offset 20` |
| `--file path` | Read input from file | `cap model inputs create --file input.json` |
| `--periods "<names>"` | Comma-separated period names | `cap reporting widgets table <id> --periods "Q1 FY 25"` |

> **Tip:** Use `cap data time-periods list --data-interval <interval>` to discover selectable reporting period names before running reporting commands. Use `cap data time-periods diagnose <yyyy-MM-dd> --data-interval <interval> --json` when a seed `startDate` does not appear to be selectable.

## Naming Conventions

Use the same option names across commands:

| Concept | Option | Notes |
|---------|--------|-------|
| Reporting period interval | `--data-interval <day|week|month|quarter|year>` | Used for input values, computed values, dashboards, and typed widgets |
| One org node | `--org-node <id>` | Used when the API accepts exactly one org node |
| One or more org nodes | `--org-nodes <id>[,<id>]` | Comma-separated tree node IDs |
| Discipline tree filters | `--discipline-nodes <id>[,<id>]` | Use this for discipline node IDs, even when filtering by one node |
| Framework tree filters | `--framework-nodes <id>[,<id>]` | Use this for framework node IDs, even when filtering by one node |
| Capture or report template | `--template <id>` | Command context determines capture vs report template |

---

## Performance Diagnostics

Use the perf wrapper to run a CLI/API workflow while sampling memory from either the child command or a target process such as the running API.

```bash
# Measure the child cap command process itself
cap perf memory-throughput \
  --arguments "masterdata org-nodes download-excel --output /tmp/org.xlsx --json" \
  --rows 130000 \
  --json

# Measure the API process while the CLI drives the route
cap perf memory-throughput \
  --arguments "masterdata org-nodes download-excel --output /tmp/org.xlsx --json" \
  --pid <api-process-id> \
  --rows 130000 \
  --json

# Measure async tree upload end-to-end through the CLI
cap perf memory-throughput \
  --arguments "masterdata disciplines upload-excel \"artifacts/performance/excel/Discipline Structure.xlsx\" --timeout-seconds 2100 --json" \
  --pid <api-process-id> \
  --rows 130000 \
  --json
```

Useful flags:

| Flag | Description |
|------|-------------|
| `--arguments "<cap args>"` | Command arguments passed to the current `cap` executable |
| `--executable <path>` | Override the executable, useful when running from source with `dotnet` |
| `--pid <id>` | Sample a specific process, usually the API process |
| `--process-name <name>` | Sample the largest process with this name when PID is not known |
| `--rows <n>` | Adds `rowsPerSecond` throughput to JSON output |
| `--sample-ms <n>` | Sampling interval, default `250` |

---

## Output Modes

| Mode | When to Use | Format |
|------|-------------|--------|
| **Table** (default) | Human reading, quick checks | Formatted table with columns |
| **JSON** (`--json`) | AI/automation, parsing, piping | Structured JSON object |

**Tip:** Always use `--json` when the output will be processed programmatically.
Set `CAPSTONE_OUTPUT_VERSION=2` to opt into the shared `{ ok, data, error,
warnings, meta }` JSON envelope on commands that use the shared formatter.

---

## Agent Discovery Commands

These commands are embedded in the binary so agents can inspect the CLI without reading repository docs.

```bash
cap schema --json                  # Command names, args, defaults, mutating marker, output models
cap concepts --json                # Compact Capstone domain ontology
cap workflows list --json          # Workflow index for common agent tasks
cap workflows show inspect-model --json
cap meta lookups list --json       # Cross-domain lookup metadata
cap version --json                 # CLI/runtime/install context
cap update check --json            # Explicit update availability check; no install is performed
```

`cap update check --json` uses `CAPSTONE_CLI_RELEASE_MANIFEST_URL` or `--manifest-url` when provided, otherwise it checks GitHub Releases metadata.

---

## Schema-Derived Command Surface

`cap schema --json` is the authoritative argument-level contract. This table is
a compact prefix map refreshed from schema output during documentation updates so
public users and agents can see the implemented command families without source
access. Use the action-first sections below when choosing a workflow; use this
map when checking whether a command family exists.

| Prefix | Commands |
|--------|----------|
| `auth` | `doctor`, `languages`, `login`, `logout`, `switch-tenant`, `tenants`, `whoami` |
| `auth apikey` | `create`, `list`, `prune`, `rotate`, `delete` |
| `config` | `get`, `list`, `set`, `show`, `unset` |
| `data` | `availability` |
| `data bulk-validation` | `preview`, `validate` |
| `data change-requests` | `create`, `delete`, `get`, `list`, `save`, `validate` |
| `data data-lock` | `lock`, `unlock` |
| `data input-values` | `create`, `download-excel`, `get`, `history`, `list`, `save`, `upload-excel`, `upload-status`, `validate` |
| `data input-values attachments` | `download`, `upload` |
| `data lookups` | `get` |
| `data narrative-values` | `get`, `get-by-context`, `get-by-metric-context`, `history`, `list`, `save`, `submit`, `validate` |
| `data recalculation` | `status`, `wait` |
| `data narrative-aggregation` | `status`, `wait`, `log` |
| `data time-periods` | `diagnose`, `list` |
| `masterdata data-sources` | `create`, `delete`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `masterdata discipline-attribute-types` | `create`, `delete`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `masterdata disciplines` | `create`, `delete`, `download-excel`, `get`, `list`, `move`, `save`, `upload-excel`, `upload-status` |
| `masterdata framework-attribute-types` | `create`, `delete`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `masterdata frameworks` | `create`, `delete`, `download-excel`, `get`, `list`, `move`, `save`, `upload-excel`, `upload-status` |
| `masterdata org-node-attribute-types` | `create`, `delete`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `masterdata org-nodes` | `create`, `delete`, `download-excel`, `get`, `list`, `move`, `save`, `upload-excel`, `upload-status` |
| `masterdata units` | `create`, `delete`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `meta lookups` | `get`, `list` |
| `model calculation-overrides` | `copy`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `model calculations` | `create`, `delete`, `download-excel`, `get`, `get-bulk`, `impact`, `lint`, `list`, `save`, `upload-excel`, `upload-status` |
| `model disciplines` | `list` |
| `model formula-validation` | `validate` |
| `model frameworks` | `list` |
| `model input-overrides` | `copy`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `model inputs` | `create`, `delete`, `download-excel`, `get`, `get-bulk`, `impact`, `lint`, `list`, `save`, `upload-excel`, `upload-status` |
| `model lookups` | `get`, `list` |
| `model metric-attribute-types` | `create`, `delete`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `model metric-framework-nodes` | `copy`, `download-excel`, `list`, `save`, `upload-excel` |
| `model metric-org-node-exclusions` | `copy`, `download-excel`, `list`, `save`, `upload-excel` |
| `model metrics` | `delete-eval`, `delete-status`, `explain`, `get`, `graph`, `list` |
| `model narrative-attribute-types` | `create`, `delete`, `download-excel`, `get`, `get-all`, `list`, `lookup`, `save`, `upload-excel` |
| `model narrative-org-node-exclusions` | `copy`, `download-excel`, `list`, `save`, `upload-excel` |
| `model narrative-overrides` | `copy`, `create`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `model narratives` | `ai-status`, `create`, `delete`, `delete-preview`, `delete-start`, `delete-status`, `distinct-values`, `download-excel`, `get`, `list`, `lookup`, `regenerate-ai`, `save`, `upload-excel`, `upload-status` |
| `model org-nodes` | `list` |
| `notifications delivery-log` | `list` |
| `notifications rules` | `create`, `delete`, `download-excel`, `get`, `lint`, `list`, `recipients`, `runs`, `save`, `upload-excel` |
| `notifications templates` | `create`, `delete`, `download-excel`, `get`, `lint`, `list`, `save`, `tokens`, `upload-excel` |
| `perf` | `memory-throughput` |
| `reporting computed-values` | `audit`, `download-document`, `download-excel`, `list`, `query` |
| `reporting completion` | `follow-ups` |
| `reporting dashboards` | `download-document`, `get-data`, `get-insights`, `header` |
| `reporting widgets` | `get-data`, `info-card`, `pie-chart`, `table`, `text-block`, `textblock`, `xy-chart` |
| `root` | `concepts`, `schema`, `status`, `version` |
| `security users` | `download-excel`, `get`, `upload-excel` |
| `system tenants` | `ai-config get`, `ai-config models`, `ai-config remove-key`, `ai-config set`, `ai-config test`, `ai-config usage`, `bootstrap-local`, `create`, `delete`, `delete-status`, `delete-watch`, `fiscal-config update`, `get`, `list`, `sample`, `save`, `schema`, `teardown` |
| `templates capture-templates` | `create`, `delete`, `download-excel`, `get`, `lint`, `list`, `resolve-content`, `sample`, `save`, `schema`, `upload-excel` |
| `templates dashboard-templates` | `audit`, `create`, `delete`, `download-excel`, `get`, `lint`, `list`, `rename`, `sample`, `save`, `schema`, `upload-excel` |
| `templates lookups` | `get` |
| `templates org-node-templates` | `create`, `delete`, `download-excel`, `get`, `lint`, `list`, `sample`, `save`, `schema`, `upload-excel` |
| `templates report-templates` | `create`, `delete`, `download-excel`, `get`, `lint`, `list`, `resolve-content`, `sample`, `save`, `schema`, `upload-excel` |
| `templates widget-templates` | `create`, `delete`, `download-excel`, `get`, `get-bulk`, `lint`, `list`, `preview`, `resolve-content`, `sample`, `save`, `schema`, `upload-excel` |
| `update` | `check` |
| `workflows` | `list`, `show` |

The three override `list` commands page on the server: `--limit` and `--offset` pick the page and `pagination.total` counts every match. `--org-nodes` and `--discipline-nodes` narrow the list (disciplines match exactly, not their sub-disciplines); with no `--org-nodes`, every org node's overrides are listed, each row with its `orgNode`. Overrides of a metric excluded at an org node are listed too. `copy` without `--ids` copies every override at the source org node.

---

## Discovery Commands

Commands for exploring and querying data.

### List All Items

```bash
cap <domain> <entity> list [--json]
```

| Entity | Command | Notes |
|--------|---------|-------|
| Metrics (all) | `cap model metrics list` | Includes inputs + calculations; output includes `friendlyName`; use `--rich` for formulas and aggregation metadata |
| Inputs | `cap model inputs list` | Input metrics only; output includes `friendlyName` |
| Calculations | `cap model calculations list` | Calculated metrics only; output includes `friendlyName` |
| Narrative Definitions | `cap model narratives list` | Text narrative definitions |
| Narrative Attribute Types | `cap model narrative-attribute-types list` | Custom narrative attributes |
| Narrative Overrides | `cap model narrative-overrides list` | Org-node narrative customizations |
| Narrative Exclusions | `cap model narrative-org-node-exclusions list --org-nodes <id> [--excluded-only]` | Narratives included or excluded at one org node; `--excluded-only` lists only the excluded ones; JSON `counts` and the footer give both totals (`all`, `excluded`) |
| Metric Exclusions | `cap model metric-org-node-exclusions list --org-nodes <id> [--excluded-only]` | Metrics included or excluded at the org node(s); `--excluded-only` (exactly one org node) lists only the excluded ones; JSON `counts` and the footer give both totals (`all`, `excluded`) |
| Metric Attribute Types | `cap model metric-attribute-types list` | Custom metric attributes |
| Metric Framework Nodes | `cap model metric-framework-nodes list --framework-nodes <id> [--search "<name>"] [--linked-only]` | One page of metrics with `isSelected` = linked to the framework node(s), or to any framework node when none is given; `--search` matches names on the server; `--linked-only` lists only the linked metrics; the footer and `pagination.total` give the real total, and JSON `counts` gives both totals (`all`, `linked`); rows carry `disciplineName` and `disciplinePath` |
| Model Org Nodes | `cap model org-nodes list` | Model exploration alias for org structure |
| Model Disciplines | `cap model disciplines list` | Model exploration alias for discipline tree |
| Model Frameworks | `cap model frameworks list` | Model exploration alias for framework tree |
| Org Nodes | `cap masterdata org-nodes list` | Hierarchical tree |
| Disciplines | `cap masterdata disciplines list` | Metric categories (tree) |
| Frameworks | `cap masterdata frameworks list` | Reporting standards (tree) |
| Units | `cap masterdata units list` | Units of measure |
| Data Sources | `cap masterdata data-sources list` | External integrations |
| Discipline Attribute Types | `cap masterdata discipline-attribute-types list` | Custom discipline attributes |
| Framework Attribute Types | `cap masterdata framework-attribute-types list` | Custom framework attributes |
| OrgNode Attribute Types | `cap masterdata org-node-attribute-types list` | Custom org node attributes |
| Capture Templates | `cap templates capture-templates list` | Data entry forms |
| Report Templates | `cap templates report-templates list` | Report layouts |
| Widget Templates | `cap templates widget-templates list` | Dashboard widget configs |
| Dashboard Templates | `cap templates dashboard-templates list` | Dashboard layouts |
| Org Node Templates | `cap templates org-node-templates list` | Alternate org views |
| Input Values | `cap data input-values list --template <id> --data-interval week --periods "..."` | Requires template + data-interval + periods |
| Narrative Values | `cap data narrative-values list --capture-interval quarter --periods "..."` | Text capture/validation rows |
| Change Requests | `cap data change-requests list --search "<metric, person or CR number>"` | Governed input and narrative edit requests |
| Data Locks | `cap data data-lock lock --data-interval quarter --periods "Q1 FY 25" --org-nodes <id> --discipline-nodes <id>` | Lock/unlock scoped data periods |

> Template `list` commands return every template by default (up to 1000); `--limit` and `--offset` page on the server, and `--search` (widget and dashboard templates) filters names on the server. `--name` keeps only the exact name, ignoring case.

### Get Single Item

```bash
cap <domain> <entity> get <id> [--json]
```

`cap masterdata org-nodes get <id> --json` returns selected attribute values by default. Add `--include-values` only when you need the full available value lists for each attribute type.

Attribute type JSON keeps the stable name-only `values` field and adds
`valueDetails` when IDs are available. Use `valueDetails[].id` for
org-node-template attribute filters:

```bash
cap masterdata org-node-attribute-types get <attribute-type-id> --json
cap masterdata org-node-attribute-types list --json
```

The five attribute type `list` commands (org node, discipline, framework, metric, narrative) page and search on the
server: `--search` (`-s`) keeps the types whose name contains the text, `--limit` (`-l`, 1–1000, default 100) and
`--offset` (`-o`) pick the page, and the footer and `pagination.total` count every match. `--search` matches names
only; to find a type by one of its values, list without `--search` and read `values`.

```bash
cap model metric-attribute-types list --search param --limit 20 --offset 20 --json
```

**Example:**
```bash
cap model metrics get <id> --json
```

### Metric Formula Exploration

Use these commands when diagnosing a model or preparing AI analysis over calculations.

```bash
# Find metrics with formula and aggregation metadata
cap model metrics list --search "Sample Metric" --rich --json

# Explain one metric's direct and transitive dependencies/dependents
cap model metrics explain <metric-id> --max-depth 3 --json

# Build a graph from one or more roots
cap model metrics graph --roots <metric-id>[,<metric-id>] --direction dependencies --json
cap model metrics graph --roots <metric-id> --direction both --max-depth 2 --json

# Validate a formula against a named calculation for circular dependency checks
cap model formula-validation validate "Total Revenue" --formula "[Revenue] + [Other Revenue]" --json

# Validate syntax/dependencies without a real calculation name
cap model formula-validation validate --formula "[Revenue] + [Other Revenue]" --json
```

`model metrics get` and rich list output include `friendlyName`, `metricType`,
`formula`, `calculationPhase`, `orgStructureAggregationMethod`, and
`timePeriodAggregationMethod`. Input and calculation `get`/`list` text output
shows a **Friendly Name** field when one is set; JSON payloads include
`friendlyName` for create/save round trips.
Formula validation lists each error under `Errors (N):` by its code (for example
`[FormulaUnknownMetric]`) with its line and column. It checks the formula text
only: an empty formula is valid there, while `calculations lint` and `save`
refuse it as `FormulaRequired`. It can omit the positional calculation name for syntax-only or
dependency validation; the CLI sends an internal collision-resistant sentinel
name so circular-dependency detection does not match a real metric. Formula
graph commands resolve both ID tokens and display-name tokens such as `[Sample
Metric]`.

### Input Impact and Lint

```bash
cap model inputs get <id> [--json]
cap model inputs impact <id> [--json]
cap model inputs lint (--file input.json | --id <id> | --all [--details]) [--json]
```

`lint` checks an input definition without saving it (`--id` lints a saved input as it is stored), with the codes the web editor and Excel upload use: errors are exactly what `save` would reject (exit code 2). Warnings and notes never block a save: a warning is a risk (values deleted or wrong, or a value let through unchecked), a note is worth knowing but harmless (for example a name close to an existing metric's) and prints as a `Note` line. `create` and `save` print the same warnings after saving. `--all` lints every input as it is stored; see [Lint every metric](#lint-every-metric---all).

`impact` shows what a saved input holds and what uses it: values by status (locked, approved, pending, future), calculated values, sites with values, the period range, the calculations, limits and unit conversions that use it, and the sites that override it. Run it before changing an input's data interval: that change deletes every one of these values. After such a save, `save` and `create` print what was deleted ("Data interval Month → Quarter deleted … values"; with `--json`, a `notice` field).

`get` includes the input's org-node overrides in full (`input.siteOverrides` with `--json`; a `Site overrides: N` table otherwise), so `get --json` → edit → `save --file` keeps them: a `siteOverrides` list is saved as the full set, and an absent or `null` field leaves the stored overrides as they are. See [Model Building](./model-building.md#org-node-overrides-inputs-and-calculations) for the row shape.

### Calculation Impact and Lint

```bash
cap model calculations get <id> [--json]
cap model calculations impact <id> [--json]
cap model calculations lint (--file calculation.json | --id <id> | --all [--details]) [--json]
```

`lint` checks a calculation without saving it, with the codes the web editor and Excel upload use. The file is the same payload as `save` (or the output of `get --json`); the formula may use metric names or the stored ids. `--id` lints a saved calculation as it is stored. Errors are exactly what `save` would reject (exit code 2): the formula's own errors (`FormulaRequired`, `FormulaSyntax`, `FormulaUnknownMetric`, `FormulaUnknownFunction`, `FormulaFuturePeriod`, `FormulaCircular`, each with its line and column except an empty formula's `FormulaRequired`; a circular formula is placed on the reference that closes the cycle), `CalculationPhaseRequired`, a time and org aggregation when results add up (`TimeAggregationRequired`, `OrgAggregationRequired`), and the shared metric rules (name, unit, discipline, decimals 0 to 6, `DataIntervalNotEnabled`, ranges, site overrides). Warnings never block a save:

| Code | Means |
|---|---|
| `PhaseDiffersFromSuggestion` | The formula's shape suggests the other phase, for example a ratio set to add up its results. The message gives the reason. |
| `BeforeOperandBlank` | The calculation adds up its results but uses a calculation worked out again from totals, or one that adds up at another interval: that operand is blank. |
| `BeforeInputLongerThanInterval` | The calculation adds up its results but uses an input captured less often with time aggregation Sum, Last value or None: the input is split, put in the last period or left blank before the formula runs. Average, Min and Max repeat the value, which is what a factor needs, so they don't warn. |
| `BeforeRollDownNotReported` | The calculation adds up its results with org aggregation Roll Down: reports show nothing for it. |

With `--json` the result carries `counts`: `references` (each metric the formula uses, with its `metricType`, `dataInterval`, `calculationPhase` and both aggregations), `suggestedPhase`, `suggestionReason` and `suggestionReasonCode`. `create` and `save` lint the draft first and print its warnings and notes after saving (`create --json` carries them as `warnings` and `notes`). When lint finds errors, `create` and `save` (of an input too) don't send the draft: they exit with code 2 and list the errors under `lintErrors`, each with its code and, for a formula, its line and column.

`get` includes the calculation's org-node overrides in full (`calculation.siteOverrides` with `--json`; a `Site overrides: N` table otherwise), so `get --json` → edit → `save --file` keeps them, as for inputs: a `siteOverrides` list is saved as the full set (`[]` deletes them all), and an absent or `null` field leaves the stored overrides as they are.

`impact` lists what uses a saved calculation (calculations, limits and unit conversions): they recalculate after it is saved. An id that is not a calculation is reported as not found (exit code 2).

### Lint every metric (`--all`)

`cap model calculations lint --all` and `cap model inputs lint --all` read every saved calculation (or input) and lint each one as it is stored, then print a summary: how many were linted, each code with how often it occurs and where, and the ones `save` would refuse (`Failing:`). `--details` also prints each report. With `--json` the summary is `{linted, withErrors, withWarnings, codes:[{code, severity, count, names}], failing:[{id, name, codes}], unread, reports}`; `unread` is left out when every one was read and linted, and `reports` comes only with `--details`. The exit code is 2 when any would be refused, and 6 when some could not be read or linted.

Use it as the phase audit: the `PhaseDiffersFromSuggestion` line names every calculation whose phase differs from what its formula suggests. Fixing them is an authoring decision; lint does not change anything.

### Get Multiple Items (Bulk)

```bash
cap templates widget-templates get-bulk -i <id1>,<id2>,<id3> [--json]
cap model inputs get-bulk --ids <id1>,<id2> --json
cap model calculations get-bulk --ids <id1>,<id2> --json
```

> IDs are passed as a **comma- or whitespace-separated list on one `--ids` (alias
> `-i`) flag**. ConsoleAppFramework last-wins on repeated flags, so
> `--ids <id1> --ids <id2>` returns only the last id. Positional arguments
> (`get-bulk <id1> <id2>`) are not accepted.

Retrieves multiple full entities in one CLI invocation. Returns partial results — valid entities are returned alongside errors for invalid or inaccessible IDs. Model `get-bulk` commands return the same editable DTOs as `get` (without input site overrides), so automation can hydrate list results before analysis or save-round-tripping without spawning one process per entity.

By default `templates widget-templates get-bulk` returns compact widget summaries.
Add `--full` to return the full editable widget template DTOs — including nested
`valueSelectionConfig` — matching the `get` payload shape, so the output can be
edited and piped back to `save --file`:

```bash
cap templates widget-templates get-bulk -i <id1>,<id2> --full --json
```

**Supported entities:** `templates widget-templates`, `model inputs`, `model calculations`

### Identity For Excel Uploads

The CLI uses one canonical identity contract for Excel upload workflows, the same
one the web app Excel upload uses:

| Entity shape | Match key | Rule |
|--------------|-----------|------|
| Flat entities | Exact full name | Names are unique within the entity type. Leading/trailing whitespace is ignored and matching is case-sensitive. A matching full name updates the existing entity; no match creates a new entity. |
| Tree entities | Exact full path | Paths are unique for tree entities. Leading/trailing whitespace is ignored and matching is case-sensitive. A matching full path updates the existing node; no match creates a new node at that path. |

Do not use fuzzy matching, aliases, partial names, or tree-node short names for
upsert identity. Agents can inspect the `upsertIdentity` section from
`cap schema --json` before planning uploads. Its `surfaces` list has a single
`excel-upload` entry: bulk loading, backup/restore, and bulk edits all go through
`download-excel` / `upload-excel`. Create or update a single item with
`create --file` or `save --file`.

Masterdata and model commands take the workbook path positionally; template
commands take it with `-f`:

```bash
cap masterdata units upload-excel units.xlsx --json
cap masterdata org-nodes upload-excel org-nodes.xlsx --json
cap masterdata disciplines upload-excel disciplines.xlsx --json
cap masterdata frameworks upload-excel frameworks.xlsx --json
cap model inputs upload-excel inputs.xlsx --json
cap model calculations upload-excel calculations.xlsx --json
cap templates widget-templates upload-excel -f widget-templates.xlsx --json
cap templates dashboard-templates upload-excel -f dashboard-templates.xlsx --json
```

### Moving Tree Nodes

`move` puts org nodes, disciplines or framework nodes, each with everything under it, under a new parent or at the
top level. The nodes move together in one save: if one is refused, none moves. A node listed under another listed
node moves with it.

```bash
cap masterdata org-nodes move --ids <id>,<id> --parent <new-parent-id> --json
cap masterdata disciplines move --ids <id> --top-level --json
cap masterdata frameworks move --ids <id> --parent <new-parent-id>
```

Give exactly one of `--parent` and `--top-level`. A refused move exits non-zero with the reason, for example
`HTTP 422 A tree node cannot be moved under itself or one of its descendants.` or a 409 when the new parent already
has a node with that name. Moving a site recalculates totals and narrative roll-ups for the whole workspace, and
access granted on a site follows the sites moved under it.

### Clearing A Tree-Node Attribute

Org nodes, disciplines and frameworks keep an attribute value until you clear it:

- **Save or create:** send the attribute with nothing selected. Key the entry by
  `id` or by `attributeTypeId`, the field `get --json` prints. An entry with
  neither is refused before anything is sent.

  ```json
  "attributeValues": [{ "attributeTypeId": "<attribute-type-id>", "selectedValues": [] }]
  ```

- **Attributes you leave out** of `attributeValues` keep their values, so
  `"attributeValues": []` changes nothing.
- **Excel upload:** a blank attribute cell clears that row's value. A workbook
  without the attribute column keeps every value. An invalid value is reported as
  an upload error, and the row keeps its stored value.

An attribute type cannot be deleted, and a save or Excel upload cannot drop one
of its values, while records still use it. The API answers `409` with a summary
that counts the uses per kind, and the CLI prints every use under `uses:` (the
value or type, the kind of record, and that record's name or path). Clear those
first.

### Backup and Restore

Back up and restore a tenant's model configuration (masterdata, model, templates) with the same
Excel download and upload the web app uses. There is no separate CLI export format. Excel files
reference other entities by name, not by id, so a backup restores into the same tenant or into
another one. Captured values use their own capture-template workflow.

```bash
cap workflows show backup-restore --json
```

See [Backup and Restore Recipe](../recipes/data-management/backup-restore.md) for
the download commands and the upload order.

### Discover Time Periods

```bash
cap data time-periods list --data-interval <type> [--json]
cap data time-periods diagnose <yyyy-MM-dd> --data-interval <type> [--strict-selectable-periods] [--json]
cap data availability --data-interval month [--org-nodes "<id>"] [--json]
cap data availability --data-interval month --dashboard-template <id> [--org-node-template <id|physical>] [--org-nodes "<id>"] [--json]
```

| Type | Command |
|------|---------|
| Monthly | `cap data time-periods list --data-interval month` |
| Quarterly | `cap data time-periods list --data-interval quarter` |
| Yearly | `cap data time-periods list --data-interval year` |
| Weekly | `cap data time-periods list --data-interval week` |
| Daily | `cap data time-periods list --data-interval day` |

`time-periods list --json` returns an array of selectable
`TimePeriodTreeNodeDto` items with `name`, `startDate`, `endDate`, and
`timePeriodType`. Text output also prints the selectable range for the tenant.
Use `time-periods diagnose --json` for machine-readable range diagnostics:
`reportingSelectable`, `selectableRange`, `selectablePeriod`,
`nearestSelectablePeriods`, `mismatchReason`, and `seedValidity`. When a date is
outside the selectable range, extend the tenant's configured reporting period
range before using that `startDate` in strict seed loads.

### Lookup Enums

```bash
cap meta lookups list --json
cap meta lookups get <name> [--domain model|data|templates|meta] --json
cap <domain> lookups get <name> [--json]
```

| Domain | Available Lookups |
|--------|-------------------|
| `model` | `metric-types`, `time-period-types`, `time-period-aggregation-methods`, `org-structure-aggregation-methods`, `calculation-phases` |
| `templates` | `widget-sizes`, `widget-types`, `data-grouping-types`, `data-range-modes`, `metric-selection-modes`, `metric-partitioning-modes`, `partitioning-rank-modes`, `org-node-row-selection-modes`, `org-node-template-types`, `org-node-template-visibility-modes`, `pie-chart-types`, `xy-chart-data-item-types` |
| `data` | `change-request-reasons`, `change-request-status-types`, `change-request-validation-levels` |
| `meta` | `color-tokens` |

`model lookups get time-period-types` and `templates lookups get data-intervals`
return only the current tenant's enabled reporting periods, not every interval.

Use `meta lookups` when building scripts or agents that need one discovery
surface for all enum/reference values. The domain-specific commands remain
available for direct lookup calls.

`cap meta lookups get color-tokens --json` is offline and does not require API
configuration. Use it to discover the shared dashboard/widget color registry,
deprecated legacy tokens, and aliases. Template `schema --json` color fields
list the 22 canonical non-deprecated token names in `values[]`.

Some widget-template enum sets are schema-owned rather than standalone lookup
endpoints. For Pie/Donut center, legend-value, slice-label, shape (`pieShape`) and Other-slice (`otherSliceMode`) modes, read
`cap templates widget-templates schema --widget-type pie --json` and inspect
`.fieldLookups[]` for `widgetTemplate.centerMode`,
`widgetTemplate.legendValueMode`, `widgetTemplate.sliceLabelMode`,
`widgetTemplate.pieShape` and `widgetTemplate.otherSliceMode`.

### Reporting Commands

Computed values, dashboards, and widgets. `--periods` is required for live data. `--data-interval` is the request grain: `day`, `week`, `month`, `quarter`, `year`, or `match-selected`. Period names are resolved preferring that grain, then other types (`month` + `FY 26` works). Omit `--data-interval` on dashboards to use `defaultDataInterval`, else match-selected. Typed widget commands omit `--data-interval` to use a following widget's default interval, else match selected periods. A widget that fixes its interval ignores the request interval; a following widget's interval is only the default. `computed-values query` and `computed-values audit` can resolve recent periods with `--period-count`.

```bash
# Computed values (via report template filter lens)
cap reporting computed-values list --template <report-template-id> --data-interval day --periods "Jan 25" [--json]

# Same report on another org structure (an org node template ID, or physical)
cap reporting computed-values list --template <report-template-id> --org-node-template <org-node-template-id> --data-interval month --periods "Jan 25" --json

# Framework-grouped report template, optionally narrowed to a framework node
cap reporting computed-values list --template <framework-report-template-id> --framework-nodes <framework-node-id> --data-interval year --periods "FY 2026" --json

# Computed values (ad-hoc metric query, no saved report template required).
# With --org-nodes, rows are grouped by org node and the table's Org Node column shows each row's org node path.
cap reporting computed-values query --metrics <metric-id>[,<metric-id>] --data-interval month --period-count 3 --include-metric-details --json

# Computed values audit for post-build verification and CI smoke checks
cap reporting computed-values audit --metrics <metric-id>[,<metric-id>] --data-interval month --periods "Jan 25" --target-model-version <version> --strict --json

# The report view as PDF or Word
cap reporting computed-values download-document --template <report-template-id> --data-interval month --periods "Oct 2025,Nov 2025" [--org-nodes <id>] [--format pdf|word] [-o report.pdf]
```

`download-document` refuses a report too large to build in time with HTTP 400 and a message saying what to narrow, for example `Failed to download document: HTTP 400 This report prints more than 50,000 rows with its calculations expanded, too many for a PDF. Download it as Word or Excel, or choose fewer sites or filters.` The default limits are 50,000 printed rows or 650,000 cells for PDF, and 150,000 rows or 2,000,000 cells for Word. Printed rows include every expanded calculation child and group heading, so they can far exceed the grid's row count. Choose fewer `--org-nodes`, fewer periods or a coarser interval. `download-excel` builds the same rows and has Word's limits (150,000 rows, 2,000,000 cells) plus Excel's own 16,384 columns; a PDF too large is refused with advice to download Word or Excel instead.

| Flag | Description | Required |
|------|-------------|----------|
| `--template <id>` | Report template whose filters determine visible metrics | Yes |
| `--metrics "<ids>"` | Metric IDs for ad-hoc `computed-values query` or `audit` | Yes for `query`, `audit` |
| `--data-interval <interval>` | day, week, month, quarter, year | Yes |
| `--periods "<names>"` | Comma-separated period names | Yes for `list`; optional for `query`/`audit` when `--period-count` is supplied |
| `--period-count <n>` | Resolve the most recent N periods for `computed-values query`/`audit` | No |
| `--org-nodes "<id>"` | Filter to specific org node | No |
| `--org-node-template <id\|physical>` | Org structure to run on: an org node template ID or `physical`. On `list` and `download-excel`, omit it for the template's default structure; on `query` and `audit`, omit it for the physical structure | No |
| `--discipline-nodes "<id>"` | Narrow the report template discipline scope | No |
| `--framework-nodes "<id>"` | Narrow the report template framework scope; framework-grouped reports render metrics under framework paths | No |
| `--metric-types "<types>"` | `input`, `calculation` (comma-separated) | No |

`--org-nodes`, `--discipline-nodes` and `--framework-nodes` are held inside the
template's Limit to lists (`orgNodes`, `disciplines`, `frameworks`), or inside
its default picks when the template fixes that filter. Picks outside are
dropped, and no rows return when none remain. Omit them to use the template's default picks (`defaultOrgNodes`, else
`orgNodes`, and so on). `data input-values list` follows the same rule.

When the template places grouping levels on columns, each table row of `list`
takes its org node, discipline and framework from the column's group, and the
table adds a Discipline or Framework column for those levels. With Site on
columns, a **Row Group** column keeps the row's own (innermost) grouping level,
such as its discipline. Cells that do not apply (`isNotApplicable`, shown as
n/a in the web grid) are layout padding and print no row. A narrative row prints
one row per column group it applies in, with the narrative name under Metric.
`data input-values list` prints the same way. `--json` returns the full
response, including `columnGroups` (the column-group tuples that each column's
`groupIndex` points into), per-period `narrativeValue` (no level on columns) or
`narrativeColumns` (one cell per `columnGroups` entry), and `isNotApplicable`
on cells that do not apply.

Both `--json` responses also carry `orgNodes` (org node id → name for the
sites on the page). `data input-values list --json` adds `users`
(user id → display name), `validationLevels` (id → `name`, `rule`) and
`lockOperations` (id → `description`, `byId`, `date`), and on each value cell
`lastChangeById` and `lastChangeDate` (a data feed saves as the empty GUID),
`lastValidation` (`levelId`, `byId`, `date`, `approved`, and
`rejectionReason` on a rejection), `awaiting` (`levelId`, `approved`,
`required`, only while In Progress), `lockOperationId` and
`latestAttachmentDate`. Narrative cells (`narrativeValue`, `narrativeColumns`)
carry the same fields. Each field is left out when the API does not send it.

`--org-node-template` on `list` and `download-excel` must name one of the
report template's allowed org structures; anything else returns no rows. A
template that fixes its org structure (`showOrgNodeTemplateFilter` false)
allows only its default structure. Org node IDs belong to one structure, so
`--org-nodes` must name nodes of the chosen structure; IDs from another
structure are dropped, and no rows return when none remain.

`templates org-node-templates delete` fails with 409 while a report template
lists the org structure as allowed, or uses it as the default of a template
that lets viewers switch. The error names those templates; remove the
structure from them first.

`computed-values audit --json` returns `ComputedValueAuditResult` with
`passed`, `summary`, `modelState`, `findings`, and `missingMetricIds`. Findings
include stale model state, no data rows, all-null values, and requested metrics
missing from the response. Add `--strict` when CI should receive exit code `6`
for audit findings while still getting the JSON body.

```bash
# Widget data (type-specific — returns typed JSON for rendering)
cap reporting widgets info-card <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" [--json]
cap reporting widgets pie-chart <widget-template-id> --org-nodes <id> --data-interval month --periods "FY 26" [--json]
cap reporting widgets xy-chart <widget-template-id> --org-nodes <id> --data-interval match-selected --periods "FY 26" [--json]
cap reporting widgets table <widget-template-id> --org-nodes <id> --data-interval quarter --periods "Q1 FY 25" [--json]
cap reporting widgets text-block <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" [--json]

# Widget data (legacy CSV compatibility payload, not dashboard Table render JSON)
cap reporting widgets get-data <widget-template-id> --org-node <id> --periods "FY 24, FY 25" [--json]

# Dashboard data — CSV columns are the union of every widget's effective interval/range
cap reporting dashboards get-data <dashboard-template-id> --org-node <id> --periods "FY 26" [--json]
cap reporting dashboards get-data <dashboard-template-id> --org-node <id> --data-interval month --periods "FY 26" [--json]
cap reporting dashboards get-data <dashboard-template-id> --org-node <id> --data-interval match-selected --periods "FY 26" [--json]
cap reporting dashboards get-insights <dashboard-template-id> <layout-node-id> --org-node <id> --periods "FY 26" [--json]
# Prints the summary's title, then each insight's type, severity (Good, Watch, Bad) and text. --json returns
# {title, subtitle, footnote, styleConfiguration, insights[]} as the dashboard shows it.

# Dashboard header resolved for a selection (eyebrow, title, subtitle, badge + token warnings)
cap reporting dashboards header <dashboard-template-id> --org-node <id> --periods "Q1 FY 26, Q2 FY 26" [--data-interval quarter] [--json]

# Dashboard as a PDF, Word or PowerPoint document (charts and AI Summaries print as placeholders: they are drawn in the browser).
# PowerPoint gives each section that holds widgets its own 16:9 slides, after a title slide.
cap reporting dashboards download-document <dashboard-template-id> --org-node <id> --periods "Q2 FY 27" [--data-interval quarter] [--format pdf|word|pptx] [--formatting on|off] [-o dashboard.pdf] [--json]

# Same dashboard on another org structure, when the template lets viewers switch it (an org node template ID, or physical)
cap reporting dashboards get-data <dashboard-template-id> --org-node <id> --org-node-template <org-node-template-id> --periods "FY 26" [--json]

# Dashboard template static audit before live data checks
cap templates dashboard-templates audit <dashboard-template-id> --strict --json
```

> **Note:** Type-specific widget commands (`info-card`, `pie-chart`, `xy-chart`, `table`, `text-block`) use `--org-nodes` (plural). `--data-interval` is the request grain (`day|week|month|quarter|year|match-selected`). Names prefer that grain, then other types. Omit `--data-interval` to match selected periods (a widget that fixes its interval ignores the request interval, and a following widget's interval is only the default; a table or pie that follows the dashboard, `showDataIntervalFilter: true`, reads at `--data-interval` and uses its own interval only when it is omitted). `dashboards get-data` omit `--data-interval` uses the template `defaultDataInterval`, else match-selected. Its CSV is a **union of columns** across widgets — pinned or Static-range widgets can add extra period columns; use typed widget commands to inspect one widget. CSV `widgets get-data` uses `--org-node` (singular) and returns the legacy CSV envelope. Prefer `reporting computed-values` or typed widget commands for automation-safe checks.

`--org-node-template <id|physical>` on `dashboards get-data`, `get-insights`, `header` and `download-document` picks the org structure for a dashboard with `showOrgNodeTemplateFilter: true`. Omit it for the dashboard's default structure. The key must be one of the dashboard's allowed structures (`orgNodeTemplates`, or every structure you can access when that list is empty, plus `physical` when `allowPhysicalStructure` is true), else the API answers 403. A fixed dashboard only accepts its own structure. Pass an `--org-node` of the chosen structure. A node of a structure you have no grant for is refused (403), whatever key you name: on dashboard, widget and header requests alike.

### Dashboard Template Layout Metadata

Dashboard template commands understand the same layout metadata contract across JSON, Excel, audit, schema, and sample surfaces:

```bash
cap templates dashboard-templates schema --json
cap templates dashboard-templates sample --json
cap templates dashboard-templates get <id> --json
cap templates dashboard-templates lint --file dashboard.json      # or --id <id>: what save would refuse, with codes
cap templates dashboard-templates save --file dashboard.json --json
cap templates dashboard-templates audit <id> --strict --json
cap templates dashboard-templates download-excel -o dashboards.xlsx
cap templates dashboard-templates upload-excel -f dashboards.xlsx --json
```

`lint` checks a file or a saved template (`--id`) against the server's rules without saving. Each error prints as `Error   [Code] field: message`, and the command exits 2 when save would refuse it. `create` and `save` lint first and send nothing lint refuses. `audit` reports the same lint codes, plus a `NO_WIDGETS` warning. Codes include `UnsafeText` (header or callout text holding `<`, `>`, `{`, `}`, `;`, `javascript:`, `expression(` or `url(`; words such as "description" are fine), `OutOfBounds`, `WidgetTemplateNotFound`, `UnknownOrgStructure`, `DefaultOrgNodeOutsideStructure`, and the site limit codes `SiteLimitRequiresFixedStructure`, `DefaultOutsideLimit` and `SiteNotInStructure` (`orgNodes`, the sites viewers may pick). Excel upload errors carry the same codes, at the column that holds the setting.

The JSON contract adds `showDataIntervalFilter` (default `false`) and `defaultDataInterval` (`{ "id": 2, "name": "Month" }`, omit for Match selected periods) at the template root. For the org structure it has `orgNodeTemplate` (`{id}`, null = physical): the default structure. Set `showOrgNodeTemplateFilter: true` to let viewers switch between `orgNodeTemplates` (`[{id}]`; empty = every structure the viewer can access) plus the physical structure when `allowPhysicalStructure` is true (default). The default must be one of them ("The default org structure must be one of the allowed org structures."), and a switchable structure needs `showOrgNodeFilter: true` ("Viewers who can switch the org structure need the Sites filter."). `defaultOrgNode` belongs to the default structure and applies there only. Excel carries the same settings in the "Show Org Node Template Filter", "Org Node Templates" (semicolon-separated names) and "Allow Physical Structure" columns. The template root also has `dashboardStyle` and the following optional groups on `treeItems`:

| JSON group | Applies to | Fields |
|------------|------------|--------|
| `dashboardStyle` | Dashboard canvas | `backgroundColor`, `foregroundColor`, `contentPadding`, `contentWidth`, `contentAlignment` |
| `nodeLayout` | Structural nodes only | `layoutMode`, `horizontalAlignment`, `verticalAlignment`, `minHeight`, `headerBehavior`, `spacing.padding`, `spacing.margin`, `spacing.gap`, `responsive.stackBelow`, `responsive.columns`, `responsive.wrap` |
| `nodeStyle` | Structural nodes only | `backgroundColor`, `foregroundColor`, `titleColor`, `headerBackgroundColor`, `contentBackgroundColor`, `typography`, `borderColor`, `borderRadius` |
| `placementLayout` | Widget and narrative placements only | `widthBehavior`, `horizontalAlignment`, `verticalAlignment`, `minHeight`, `spacing.padding`, `spacing.margin`, `spacing.gap`, `responsive.fullWidthBelow` |
| `callout` | Structural nodes and Narrative placements only | `variant`, `severity`, `icon`, `collapsible`, `titleTranslations`, `bodyTranslations` |

Widget placement is **layout-only** — the placement wrapper contributes width,
alignment, minimum height, spacing, and responsive behavior. All widget chrome
(background, border, shadow, corner radius, padding, accent) is owned by each
widget's own `styleConfiguration`, managed through `templates widget-templates`.

Allowed enum/token values are:

| Field family | Values |
|--------------|--------|
| Canvas content width | Default, FullWidth, Constrained, Wide |
| Canvas content alignment | Default, Start, Center, Stretch |
| Layout mode | Default, Stack, Grid, Inline, Column (Tabs, id 4, is rejected on sections) |
| Alignment | Default, Start, Center, End, Stretch |
| Minimum height | None, S, M, L |
| Header behavior | Default, Visible, Hidden, Collapsible, CollapsedByDefault |
| Spacing token | None, XS, S, M, L, XL |
| Responsive breakpoint | Never, Sm, Md, Lg |
| Typography | Default, Compact, Standard, Emphasis |
| Width behavior | WidgetSizeDefault, Auto, Fill |
| Callout variant | None, Info, Success, Warning, Critical, Narrative |
| Callout severity | Default, Low, Medium, High, Critical |

Use safe color values only: registry design tokens from `cap meta lookups get color-tokens`, `#rgb`, `#rrggbb`, or `#rrggbbaa`. Invalid identifiers, `rgb(...)`/`rgba(...)`, `var(...)`, raw CSS, HTML, scripts, callbacks, and URL values are rejected on strict save. Dashboard and widget Excel uploads warn at the cell level and ignore invalid color cells instead of failing the row. Use safe icon tokens containing letters, numbers, underscores, or hyphens.

Structured lengths are JSON objects, not CSS strings. Use a single `value` or side-specific `top`, `right`, `bottom`, and `left` numbers with `unit: { "id": 0, "name": "Px" }` or `unit: { "id": 1, "name": "Rem" }`. Free-text CSS shorthands such as `"1rem 2rem"` are rejected.

Excel dashboard-template workbooks include `Dashboard Style JSON` on the dashboard row and row-level metadata columns named `nodeLayoutJson`, `nodeStyleJson`, `placementLayoutJson`, and `calloutJson`. These cells contain the same JSON objects used by `save`.

`WidgetSize` remains the default width for widget and narrative placements. Use `placementLayout.widthBehavior` only when a placement should Auto-size or Fill its available track. Widget-template internals — including all chrome — are configured through `templates widget-templates`; dashboard placement layout only controls how that widget is sized and positioned within its track.

**Widget command discovery for agents:**

```bash
cap templates widget-templates --help
cap templates widget-templates list --help
cap templates widget-templates get --help
cap templates widget-templates get-bulk --help
cap templates widget-templates schema --help
cap templates widget-templates sample --help
cap templates widget-templates schema --widget-type info --json
cap templates widget-templates sample --widget-type info --json
cap templates widget-templates create --help
cap templates widget-templates save --help
cap templates widget-templates lint --help
cap templates widget-templates preview --help
cap templates widget-templates resolve-content --help
cap templates widget-templates download-excel --help
cap templates widget-templates upload-excel --help
cap templates dashboard-templates upload-excel --help
cap templates dashboard-templates audit --help
cap reporting widgets --help
cap reporting widgets info-card --help
cap reporting widgets pie-chart --help
cap reporting widgets table --help
cap reporting widgets text-block --help
cap reporting widgets xy-chart --help
cap reporting widgets get-data --help
```

Use `cap reporting widgets info-card ... --json` when an agent needs resolved
Info Card text, values, trend tokens, warnings, and returned
`styleConfiguration` for dashboard-render parity. When `valueState` is `NoData`,
`value`, `formattedValue`, and `rawValue` are null. `Inaccessible` still returns
`value: "Restricted"`. `NotApplicable` (`5`) also applies to metrics: an Info Card
item whose metric is excluded at the org unit, or a metric token whose metric is
excluded at every selected org unit, has value "n/a" and no warning. Narrative
tokens add two value states: `NotApplicable`
(`5`, "Not applicable": excluded for the selected org unit) and
`AmbiguousOrgNode` (`6`, "Select a single org unit": zero or several
`--org-nodes`). Pass exactly one `--org-nodes` id to resolve narrative text. Use `cap reporting widgets
pie-chart ... --json` when an agent needs the shared pie/donut render contract
with returned `styleConfiguration`, `pieShape`, `centerNumberFormat`, resolved `center`,
`labelDisplay`, `legend`, `dataItems[].presentation`,
`dataItems[].numberFormat`, `dataItems[].legendValue`, and
`dataItems[].sliceLabel` metadata, the shares (`dataItems[].share`, `formattedShare`), the Other slice (`isOther`, `otherMembers`, `otherMemberCount`), plus `dataWarnings` (`PieNegativeValuesLeftOut`,
`PieManySlices`, `PieNoValues`, `PieMixedUnits`: what the data leaves the pie unable to draw
well), `dataItems[].notDrawn` (a negative value the pie leaves out of the ring;
only the legend lists it) and `dataItems[].dataItemIndex` (the position in the
template's `dataItems` of the row the slice came from; none for a filtered
pie's slices). In non-JSON mode, the command prints
`Center: <label>: <value>` when configured and includes `Share`, `Legend` and `Slice
Label` columns; a negative slice's value reads `(not drawn)`, Other's label reads `Other (5 slices)`, and the data
warnings follow the table under `Warnings:` as `- [Code] message`. Use `cap reporting widgets table ... --json` when an agent
needs the shared dashboard table contract with `timePeriodColumns`,
`metricColumns`, `metricNumberFormats`, `gridRows`, `metadataColumns`,
`selectionDiagnostics`, `styleConfiguration`, `warnings`, `totalCount`, and
paging support. Without `--take` it returns every row: the CLI pages through the table 1000 rows at a time
until `totalCount` (the API returns every row to a request without `take`, too; `pageSize` names the table's Rows per page).
With `--take` it returns that page only; `--skip` needs `--take`. In text mode, narrative rows print each period's narrative on
one line (line breaks become spaces), cut to 80 characters; `--json` keeps the
full text with its line breaks. A table with periods as rows
(`showTimePeriodsInRows`) prints one line per row and period, with a `Period`
column before the metric columns, as the dashboard shows it. A column under a
level on columns names its groups first (`Jan 2026 / Site A / Energy`). Use `cap reporting widgets text-block ... --json` when an agent
needs the shared TextBlock render contract with resolved `titleContent`,
`subtitleContent`, `descriptionContent`, `footnoteContent`, token spans,
`styleConfiguration`, `warnings`, and `diagnostics`. Narrative spans (in
`text-block` and `info-card` output) have `tokenKind: 4` and carry
`sourceNarrativeId`; their `text` is the narrative value's plain text, with
line breaks kept. Narrative warnings (`TEXTBLOCK_TOKEN_NARRATIVE_*`,
`INFOCARD_TOKEN_NARRATIVE_*`) also carry `sourceNarrativeId`, and text mode
appends `narrative: <guid>`. Use `cap reporting widgets
get-data ... --json` when an agent needs the legacy CSV compatibility envelope
for analysis workflows.

#### Lint a widget template

```bash
cap templates widget-templates lint (--file widget.json | --id <guid>) [--json]
```

`lint` checks a widget template without saving it. Give exactly one of
`--file` (the same payload as `create` and `save`; the CLI wraps a bare
template in `{"widgetTemplate": {...}}` and runs the same local checks `save`
runs first, so a malformed payload is reported the same way) or `--id` (a saved
template, read first and checked as stored). **Errors** are exactly what `save`
would reject; **warnings** and **notes** never block a save. Each issue has a
stable `code`, a `message`, the top-level `field` it belongs to and an editor
`stage` (`data`, `layout`, `look` or `style`; empty for the name and the
discipline); some also carry `params`. Text output prints one line per issue,
errors first, as `Error   [Code] field: message`, then a count line. The exit
code is `2` when there is at least one error and `0` otherwise. `--json` prints
the API result unchanged: `{ "errors": [...], "warnings": [...], "notes": [...] }`.
It needs `Templates.Manage`.

Table templates report, among others, `NameRequired`, `NameDuplicate` (another
template in the tenant has the name), `MetricsRequired` (Static selection with
filters but no metrics listed), `NarrativesRequired`, `FrameworkGroupingNeedsMatching`,
`OrgNodeAttributeFilterIncomplete` and `UnknownMetrics` (a metric that does not
exist in the tenant). Info card templates (stages `data`, `look` and `style`)
report, among others, `DataItemDuplicate` (the same metric twice with the same
period, sites and combining), `UnknownMetrics` and `MetricNotViewable` on a row,
`StyleSlotValueInvalid` and `StyleSlotPropertyUnsupported` (with `path` and
`allowed`), the warnings `DataItemLabelDuplicate`, `DataItemWindowSinglePeriod`
and `TrendIconWithoutLabel`, and the notes `OverrideFieldsCleared` and
`TrendRangesGiveNoVerdict` (a `#trend` coloured by direction because its
metric's ranges give no verdict); see
[Create Widget Template](../recipes/configuration/create-widget-template.md#infocard-lint).
Pie templates (stages `data`, `layout`, `look` and `style`) report, among
others, `PieChartTypeRequired`, the center codes (`PieCenterNeedsDonut`,
`PieCenterModeReserved`, `PieCenterFieldsNeedMode`, `PieCenterTextRequired`,
`PieCenterFieldNotForMode`, `PieCenterMetricRequired`, `PieCenterMetricNotInPie`,
`PieCenterShareAlwaysFull`), `PieOtherThresholdInvalid`, `EnumValueInvalid`,
`TimePeriodAggregationRequired` (a warning on a pie already saved without a
method), `UnknownMetrics` and `MetricNotViewable` on a slice or `centerMetricId`,
and the notes `PieCenterNumberFormatFolded` and `PieSliceBandsFolded` (what
`save` removes); see
[Create Widget Template](../recipes/configuration/create-widget-template.md#piechart-lint).
XY templates keep their existing codes (`DASHSELECT_*`, `WIDGET_TOKEN_*`,
`XYSTYLE_*`). A metric token in text block or info card text, or in a card's row
label, reports the shared `WIDGET_TOKEN_*` codes (for example
`WIDGET_TOKEN_METRIC_REFERENCE_INVALID`, `_MISSING`, `_INACCESSIBLE`), which
replace the retired `INFOCARD_*` token codes; see
[Create Widget Template](../recipes/configuration/create-widget-template.md#textblock-token-lint).
Raw JSON text may name a metric (`[Diesel Consumption]`) instead of giving its
ID; `lint`, `save` and `preview` resolve the name and `save` stores the ID. Each code has
one English message, the same in the web editor, the CLI, MCP and the Excel
upload. An info card saved with a repeated row gets a `DataItemDuplicate`
warning instead of an error. A table saved
before these checks that already breaks `MetricsRequired` or `NarrativesRequired`
gets a warning instead, so it still saves; a new break, and any other rule, is
an error. A table that carries `metricTypeFilters` gets the note
`MetricTypesFolded`: tables no longer read metric types, so `save` stores them
as the row types (`includedDataTypes`) the note names and clears them.

`create` and `save` lint the template first. A template with lint errors is not
sent; the CLI exits with code 2 and lists each error with its code under
`lintErrors`. After a successful save they print
each lint warning to stderr as `Warning: [Code] field: message` and each note
as `Note: [Code] field: message`; with `--json`, the result adds `lintWarnings`
and `lintNotes` (issue objects) next to `warnings`, and `lintFailure` when the
check could not run. A server without the Lint
endpoint (an older release) answers 404: the CLI then saves without lint and
says nothing. If the check fails for another reason, the save still succeeds
and the CLI says it could not check. A refused save looks like this:

```text
Error: Failed to save widget template: 1 lint error(s).
  lintErrors:
    - [NameDuplicate] name: Another widget template is already called Site emissions.
```

Without lint (an older server), a save the server rejects lists each field's
messages under `errors` instead.

#### Preview a draft table, info card, text block, pie chart or XY chart widget template

```bash
cap templates widget-templates preview --file widget.json --org-nodes <guid> --periods "Jan 2026,Feb 2026" [--data-interval day|week|month|quarter|year] [--json]
```

`preview` renders a table widget template without saving it, for the org nodes
and the named periods, with your own data permissions. The options are the same
as `cap reporting widgets table`. `--file` is the same payload as `save` and gets
the same local checks first. `--data-interval` sets the request interval (omit it to match the selected periods; a table that fixes its
interval keeps it, and one that follows reads at `--data-interval`, else at its own default interval). When the
table fixes its interval and `--data-interval` asks for another one, the JSON has `fixedDataInterval`. The preview shows every
row: the CLI pages through the preview 1000 rows at a time until `totalCount` (the Preview API alone returns only the
table's first page, `pageSize` rows, to a request without `take`). Text mode prints the same grid as `cap reporting widgets table`; `--json`
prints the same table render contract. Table, info card, text block, pie chart and XY chart templates preview
(the CLI reads the type from `widgetType` in the file); any other type is
refused (HTTP 400). A draft wider than the report grid's cell
limit is refused with code `WideGridCellLimit`. It needs `Templates.Manage` and
`Reporting.View`.

An info card previews in one request (a card has no pages) and renders exactly
as `cap reporting widgets info-card` renders it once saved: rows the file adds
render too, and a row's own `bands` colour its value. Text mode prints each
item's label, value and unit; `--json` prints the info card render contract the
server returned (the same shape as `reporting widgets info-card --json`, without
the CLI's `noData` hint). A draft with an error that stops it rendering (no name,
no period window) is refused with HTTP 400, each message under its field.
A card with `countByReading: true` also returns `readingCounts`: `good`, `watch`
and `bad` hold the indexes of the data items whose value falls in that reading's
range, and `notCounted` lists each other item with its `reason` (1 no value,
2 not reported, 3 no access, 4 no range reading).

A text block previews in one request and renders exactly as
`cap reporting widgets text-block` renders it once saved. Metric names in its
text resolve to IDs first, as `save` stores them; a following text block reads at
`--data-interval`. Text mode prints the resolved title, subtitle, description and
footnote and any warnings; `--json` prints the text block render contract, where
each value span carries the `periodName` it was read at. A text block that would
not render (no text at all, no period window) is refused with HTTP 400, each
message under its field.

A pie previews in one request and renders exactly as
`cap reporting widgets pie-chart` renders it once saved. Text mode prints the
center, the slices (a negative one with `(not drawn)`) and the data warnings;
`--json` prints the pie render contract the server returned (the same shape as
`reporting widgets pie-chart --json`, without the CLI's `noData` hint), with
`dataWarnings`, `dataItems[].notDrawn` and `dataItems[].dataItemIndex`. A pie
that fixes its interval ignores `--data-interval`; one that follows reads at it.
A filtered pie whose filters match no metric is refused with HTTP 400 "No
metrics match this widget's filters."

An XY chart previews in one request and renders exactly as
`cap reporting widgets xy-chart` renders it once saved, including axes and series
the file adds before they have ids. Text mode prints the series (with their axis
and type), the axes in drawing order and the render notes; `--json` prints the XY
render contract the server returned (the same shape as `reporting widgets
xy-chart --json`), where each series carries the `dataItemIndex` of the row it
came from. The notes include `LowContrast` for a series colour that is hard to
read on white, `ExcessiveSeriesExpansion` over 40 series, and `XyNoValues` when no
series has a value for the periods at the site. A chart that fixes its interval
ignores `--data-interval`; one that follows reads at it.

#### Resolve a widget template's content

```bash
cap templates widget-templates resolve-content (--file widget.json | --id <guid>) [--json]
```

`resolve-content` lists the metrics and narratives a table widget template
resolves to, without saving it, the same way `report-templates
resolve-content` does. Give exactly one of `--file` (the same payload as
`save`) or `--id` (a saved template, as stored). Text mode prints
`Metrics (n)` and `Narratives (n)` with one name per line; `--json` prints
`{ "metrics": [{id, name}], "narratives": [{id, name}] }`. Table, info card and pie chart
templates resolve. An info card that picks its metrics lists them; one that
filters metrics lists every metric its filters match, whatever your data
permissions (the card's rows override these by metric id); it lists no
narratives. A pie lists its slice metrics the same way (picked, or matched by
its filters), not its center metric. It needs `Templates.Manage`.

**Info Card style discovery for agents:**

```bash
cap templates widget-templates sample --widget-type info --json
cap templates widget-templates schema --widget-type info --json
cap templates widget-templates get <widget-template-id> --json
cap templates widget-templates save --file info-card.json --json
cap reporting widgets info-card <widget-template-id> --org-nodes <id> --data-interval quarter --periods "Q3 FY 25" --json
```

The widget-type variant is `info`; `info-card` is accepted as a readability
alias.

The sample and schema document the `styleConfiguration` slots (`panel`, `title`,
`value`, `unit`, `label`, `description`, `footnote`, `separators`, `trend`, and
`stateMessages`), bounded typography and surface fields, panel-only
`padding`/`gap`/accent fields, per-direction trend `color`/`icon`, and Info Card
data-item `numberFormat`. For worked style patterns, use
[Configure Info Card Styles](../recipes/configuration/configure-info-card-styles.md).

**Pie style discovery for agents:**

```bash
cap templates widget-templates sample --widget-type pie --json
cap templates widget-templates schema --widget-type pie --json
cap templates widget-templates get <widget-template-id> --json
cap templates widget-templates save --file pie-chart.json --json
cap reporting widgets pie-chart <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" --json
```

The widget-type variant is `pie`; `pie-chart` is accepted as a readability
alias.

For automation, use these JSON roots:

| Command | JSON Root |
|---------|-----------|
| `cap templates widget-templates list --json` | `.data[]` contains compact grid rows. |
| `cap templates widget-templates sample --widget-type pie --json` | `.payload.widgetTemplate` contains the save-ready sample body. |
| `cap templates widget-templates schema --widget-type pie --json` | `.fieldLookups[]` contains enum values, including schema-only Pie center/legend/slice modes. |
| `cap templates widget-templates get <id> --json` | `.widgetTemplate` contains the editable full DTO. |
| `cap templates widget-templates save --file pie-chart.json --json` | Top-level acknowledgement with `success`, `id`, child data-item counts, and `warnings`; run `get` for the full updated DTO. |
| `cap reporting widgets pie-chart <id> ... --json` | Top-level render contract with `id`, `title`, `pieChartWidgetType`, `styleConfiguration`, `center`, `legend`, `labelDisplay`, `dataWarnings`, and `dataItems[]` (with `notDrawn` and `dataItemIndex`). |

The sample and schema document Pie `styleConfiguration` slots (`panel`,
`title`, `description`, `footnote`, `chartArea`, `slices`, `sliceBorders`,
`sliceLabels`, `ticks`, `tooltip`, `legend`, `legendMarkers`, `legendLabels`,
`legendValues`, `donutCenter`, and `stateMessages`), `centerNumberFormat`,
`dataItems[].presentation`, `dataItems[].numberFormat`, and existing
`valueSelectionConfig`. `styleConfiguration` slots are the chart-wide ("all
slices") base; `dataItems[].presentation` overrides them per slice. The cascade
is per-slice presentation → chart-wide slot → theme/palette. `legendMarkers`
accepts `borderColor`/`borderWidth`/`borderRadius`/`opacity` only — a marker's
colour always follows its slice, so there is no chart-wide marker fill. Set
per-slice fill with `dataItems[].presentation.slice.backgroundColor`; legacy
workbook `Colour` values are import-only aliases into that slot. Keep
style/format fields separate from `valueSelectionConfig`; raw CSS, HTML,
JavaScript, callbacks, unsafe URLs, arbitrary chart config, `styleBag`, and
`propertyBag` are rejected. For worked style patterns, use
[Configure Pie Chart Styles](../recipes/configuration/configure-pie-chart-styles.md).

Use `cap reporting widgets xy-chart ... --json` when an agent needs the shared browser-aligned XY render contract with `categoryAxes`, `valueAxes`, `chartSeries[].renderConfig`, `chartSeries[].metricMetadata`, `styleMetadata`, `diagnostics`, `warnings`, and `noData`.

---

## Modification Commands

Commands for creating, updating, and deleting data.

### Create New Item

```bash
cap <domain> <entity> create --file input.json [--json]
# OR
echo '{...}' | cap <domain> <entity> create [--json]
```

**Entities supporting create:**
- `model inputs`, `model calculations`
- `model narratives`, `model narrative-attribute-types`, `model narrative-overrides`
- `masterdata org-nodes`, `masterdata disciplines`, `masterdata frameworks`
- `masterdata units`, `masterdata data-sources`
- `templates capture-templates`, `templates report-templates`
- `templates widget-templates`, `templates dashboard-templates`, `templates org-node-templates`
- `data input-values`, `data change-requests`

For capture and report templates, `showMetricsInColumns: true` requires at
least one grouping level (`dataGrouping` other than None; error code
`MetricsInColumnsRequiresGrouping`, "Metrics as columns needs at least one
grouping."). The CLI validates this before calling the API. Each grid row is
then one combination of the chain's values (for example site and discipline),
and a metric that does not apply to the row (outside its discipline or
framework node, or excluded at its site) comes back with
`isNotApplicable: true` on its cell. Without Org Node in the chain, every row
uses the one selected site. Table widget templates still need an Org Node-only
chain for metrics as columns. A metrics-as-columns page is limited to
2,000,000 cells (rows on the page × metrics × periods) and 80,000 columns:
over that, `reporting computed-values list`, the input-value grid commands
and template previews get a 400 with `code: "WideGridCellLimit"` and the
cell and column counts. A `take: 0` column-list request is refused only
when the columns alone are over the cap; a request without `take` is
counted at the 1,000-row paging maximum. Ask for fewer metrics, sites or periods, a coarser interval, or a
smaller `take`; report requests may also send `hiddenMetricIds` to leave
metrics out of the cells (rows and paging stay the same). Template lint warns
above 500,000 cells (`WideGridSlow`). Save rejects a new template, or an edit
that widens one, whose one-year, 100-row worst case is over 2,000,000 cells
or 80,000 columns (`WideGridCellLimit` on `showMetricsInColumns`); an edit
that does not widen an already over-limit template saves with that issue as a
warning. `upload-excel` applies the same rule: a new or widened over-limit
template is not saved and appears as an upload error. Optional Then By levels go in
`additionalDataGrouping`.

`showTimePeriodsInRows: true` lists time periods as rows (under the grouping
levels) and metrics as columns in the web viewers. It requires
`showMetricsInColumns: true` (error code
`TimePeriodsInRowsRequiresMetricsInColumns`, "Periods as rows needs metrics as
columns.") and is allowed with `dataGrouping` None, where the template becomes a
period-by-metric sheet for its one site. Otherwise it only changes the web
layout: the grid API returns the same rows as metrics as columns, one per
grouping combination with every period's cells. On a flat template that is a
single row for the selected site, so `reporting computed-values` returns one
row with a column per metric instead of one row per metric.
`allowMultiOrgNodeSelect` is coerced false when no grouping a viewer can use
has Org Node.

How viewers' pickers tick is set per template. Each setting has an `…Open`
flag: `true` lets the viewer change it, `false` fixes it and shows the viewer
the rule with a lock. Leaving them out keeps the defaults below.

| Field | Values | Default |
|---|---|---|
| `orgNodeCascade`, `orgNodeCascadeOpen` | ticking a site ticks the sites below it | `false`, `true` |
| `orgNodeLevelRule`, `orgNodeLevelRuleOpen` | `{ "id": 0, "name": "Any" }`, `{ "id": 1, "name": "Lowest" }` (only sites with no site under them in the viewer's picker), `{ "id": 2, "name": "One Level" }` (the first tick decides the level; needs `allowMultiOrgNodeSelect`) | Any, `false` |
| `disciplineCascade`, `disciplineCascadeOpen` | ticking a discipline ticks the disciplines below it | `false`, `true` |
| `frameworkCascade`, `frameworkCascadeOpen` | ticking a framework node ticks the nodes below it | `false`, `true` |

These change the pickers only, not the rows a filter returns. A CLI older than
these fields drops them on `save`, which resets them to the defaults.

To let viewers regroup the grid, set `showDataGroupingFilter: true` and list
the levels they may group by in `allowedDataGroupingLevels`. Viewers build their
own chain from those levels, in any order (one to three levels, each once):

```json
{
  "dataGrouping": { "id": 1, "name": "OrgNode" },
  "additionalDataGrouping": [{ "id": 2, "name": "Discipline" }],
  "showDataGroupingFilter": true,
  "allowedDataGroupingLevels": [{ "id": 1, "name": "OrgNode" }, { "id": 2, "name": "Discipline" }]
}
```

`{ "id": 0, "name": "None" }` lets viewers pick no grouping (a flat list).
The levels need at least two entries, None included (`DataGroupingLevelsRequired`),
and every level of the default grouping, or None for a flat default
(`DefaultDataGroupingOutsideLevels`). While None is allowed,
`showMetricsInColumns` needs `showTimePeriodsInRows`
(`MetricsInColumnsRequiresGrouping`). The API stores the levels in None,
OrgNode, Discipline, Framework order. With `showDataGroupingFilter: false` the levels are ignored and saved
empty. CLI and MCP data reads use the default grouping.
`columnDataGroupingLevels` places levels of the grouping chain on columns
(same `{ "id", "name" }` values, in chain order; empty or missing means every
level is on rows). With Site on columns the grid has a column block per site and
rows carry no site; with Discipline on columns a metric sits under its
discipline's block; with Framework on columns it repeats under each linked
framework node. Save and the CLI reject a level that is not in the grouping
(`ColumnDataGroupingLevelOutsideChain`), a level that is not Org Node,
Discipline or Framework or repeats (`ColumnDataGroupingLevelInvalid`), and a
layout with no rows (`LayoutRequiresRow`, for example every level on columns
while metrics are columns and periods are columns) or no columns
(`LayoutRequiresColumn`). `get` prints the placement as **Column Levels**.
In the template Excel download and upload the placement is the **Column Data
Grouping Levels** column. An upload with an invalid placement is not rejected:
it returns a warning and saves the template with every level on rows.
In grid reads, `inputColumns` / `metricColumns` list the leaf columns; each has
a `key` (the column-group ids, then the metric id, joined with `_`; just the
metric id when no level is on columns), a `groupIndex` and its `groups`. Cells
pair with these columns by position, not by id, because a metric repeats under
each site or framework block. With metrics as rows the columns are group-only:
their `id` is the empty GUID and each metric row has one cell per column.
`metricPropertyColumns` is the list of metric metadata columns next to the
metric name (enum `{ "id", "name" }` values). Spreadsheet templates default
to Unit of Measure; set `[]` to hide it. `metricAttributeTypeIds` adds one
column per metric attribute type. Grid layout hides those columns when
metrics are shown as columns; Excel download still writes them because Excel
is always metrics-as-rows.
Report templates may set `expandCalculations: true` with
`expandCalculationsMaxDepth` between 1 and 20 (default 10). Expansion is
rejected when `showMetricsInColumns` is true or any level is on columns
(`ExpandCalculationsRequiresRowGrouping`).
Report templates may also set `showEvidence: true` (default false): the
report's `download-excel` and `download-document` (PDF and Word) output then
add an Evidence column after each value column (`<period> Evidence`, or
`<metric> Evidence` when metrics are columns) with the number of files on
the value and the date the latest was added, for example
`2 files, 7 Oct 2026`. In Excel, values with files carry the same text as a
cell note whatever the setting.

Capture and report templates may set `showChangeRequests` and
`allowChangeRequestsToggle` (both default false). `showChangeRequests: true`
marks grid cells whose value has a pending change request the viewer may see.
`allowChangeRequestsToggle: true` gives viewers a Change Requests switch.
Opening a value names its pending request whatever the setting. Excel and
document downloads do not change. `get` prints both as **Show Change
Requests** and **Allow Change Requests Toggle**, and `save` keeps them. In the
template Excel workbook they are the `Show Change Requests` and
`Allow Change Requests Toggle` columns; a blank cell or a missing column means
false. A capture or validation completion report refuses both
(`MeasureExcludesChangeRequests`, see below).

Report templates may set `measure` (`{ "id": 0 }` Value, the default; `1`
Capture completion; `2` Validation completion). A completion template
counts, per cell, the values of its inputs and narratives that are expected
and due, and how many are captured (or validated) instead of showing values.
Nothing is stored; counts are worked out per request, over the data the
caller may view. Its layout is fixed: inputs and narratives as rows (no
calculations, so `includedDataTypes` must not be empty or include
calculations, and no `metricTypes` filter), periods as columns, every
grouping level on rows, the physical org structure, and none of
`showMetricsInColumns`, `showTimePeriodsInRows`, `expandCalculations`,
`showOnlyRowsWithValues`, `showRowsWithValuesFilter`, `showEvidence`,
`showChangeRequests`, `allowChangeRequestsToggle` or an
org node template setting. `save`, `create` and `upload-excel` refuse a
breach with the rule's message (for example "Capture and validation progress
show metrics as rows, not columns."), exit code 2; `lint` reports each breach
with a `Measure…` code (for example `MeasureRequiresMetricsInRows`). A selection of more than 65,000 values
(item × site × period) is refused with "Too many values to count. Narrow the
sites or the period."

A completion template may set `showWhoIsNext: true` (HB-618): each row then
names who acts next on its outstanding values (`whoIsNext` on each row and
`next` on each cell, with `users` and `validationLevels` naming the ids), by the
reminder rules: capturers of a blank, the open level's validators, or whoever
saved a rejected value. A Value template refuses it
(`MeasureRequiredForWhoIsNext`). In the template Excel workbook it is the
`Show Who Is Next` column; a missing column means false. Table widget
templates take the same `measure` and `showWhoIsNext` (Excel columns `Measure`
and `Show Who Is Next`); a progress table also refuses `valueSelectionConfig`
(`MeasureExcludesValueSelection`).

A completion template may set `completionBands`: bands as a metric's `bands`
(`lowerBoundValue`, `upperBoundValue`, `backgroundColor`, `foregroundColor`)
over the share done, 0 to 100, for example
`[{"backgroundColor":"danger-subtle"},{"lowerBoundValue":50,"backgroundColor":"success-subtle"}]`.
Omitted or null uses the defaults: red (`danger-subtle`) below 40, amber
(`warning-subtle`) from 40 and green (`success-subtle`) from 90; `[]` colours
nothing. The band rules apply, and an end must be a number: a
`lowerBoundMetric` or `upperBoundMetric`, `requireComment` or `autoValidate`
is refused. `save`, `create` and `upload-excel` refuse a breach with the
rule's message, exit code 2; `lint` names its code (for example
`CompletionBandsLimitMetric`). The bands only colour the browser, PDF and Word;
`computed-values list --json` returns them as `completionBands`.

On a completion template, `reporting computed-values list --json` adds
`measure`, `isScopeRestricted` (true when the counts cover only the data the
caller may view), `completion` on each cell and `completionTotal` on each row
(`expected`, `done`, `percent`, `state` when the cell holds one value,
`notDue` when nothing in it is due yet), and `groupCells`: every group row's
figures keyed by its path (levels joined with ` -> `; `""` is the total).
`metricValue` is null. States: 0 Not captured, 1 Captured, 2 Awaiting
validation, 3 In validation, 4 Validated, 5 Rejected. Use `--json`: the table
output shows no completion figures. `download-document` prints the figures
with a Total row and column, and `download-excel` writes the same layout as
one sheet. For a list of values to filter, use a template laid out down to the item. A value counts in the column that holds
its period's end (a quarterly input in the quarter-end month); a period is
due once its last day is before today's UTC date.
Each of the site, discipline and framework filters has two lists.
`orgNodes`, `disciplines` and `frameworks` (plus the matching
`*AttributeFilters`) are the **Limit to** lists: no viewer reaches data
outside them, whether or not the filter is open. Empty means everything the
viewer can access. `defaultOrgNodes`, `defaultDisciplines` and
`defaultFrameworks` are the **Default** picks the grid starts on. They must
sit inside the matching Limit to list when it has picks (error code
`DefaultOutsideLimit`). An empty Default starts on the Limit to picks.
A save replaces every list, so a JSON file without the `default*` keys (for
example one exported before these fields existed) clears the template's stored
defaults: export the template first and edit that file to keep them.
`show*Filter: false` fixes the filter on the Default picks (else the Limit to
picks). `show*Filter: true` lets the viewer pick anything inside the Limit to
list; picks outside it are dropped.

When rows are grouped without Org Node (for example `dataGrouping` Discipline),
or `showDataGroupingFilter` is true (a viewer can always group without Org Node),
a fixed site filter (`showOrgNodeFilter` false) must resolve to exactly one
org node: one `defaultOrgNodes` pick, or one `orgNodes` pick when there is no
default (error code `SingleOrgNodeRequired`). An open site filter may have at
most one `defaultOrgNodes` pick when the default grouping lacks Org Node
(error code `SingleDefaultOrgNodeRequired`). This includes flat templates
(`dataGrouping` None): without a Site level the grid shows one site.

`orgNodeTemplate` (`{ "id": ... }`, null = the physical structure) is the
template's org structure, and site picks in `orgNodes` and `defaultOrgNodes`
must belong to it (error code `SiteNotInStructure`). Report templates can let
viewers switch the org structure:

- `showOrgNodeTemplateFilter`: `true` lets viewers switch; `false` (default)
  fixes the report on `orgNodeTemplate`. Capture templates must leave it
  false (error code `StructureSwitchReportOnly`). Capture templates have no
  org structure: leave `orgNodeTemplate` empty (error code
  `CaptureRequiresCanonicalStructure`).
- `orgNodeTemplates`: the allowed org structures (`[{ "id": ... }]`). Empty
  means every org structure the viewer can access.
- `allowPhysicalStructure`: whether the physical structure is offered when
  switchable (default true).
- `orgNodeTemplate` is then the default structure and must be one of the
  allowed ones (error code `DefaultStructureOutsideLimit`).

Site picks belong to one structure, so while the structure is switchable:
`orgNodes` must be empty (`SiteLimitRequiresFixedStructure`); a fixed site
filter must have no `defaultOrgNodes` (`FixedSitesRequireFixedStructure`);
rows grouped without Org Node, or a grouping viewers can change, need
`showOrgNodeFilter: true` (`SwitchableStructureRequiresOpenSites`). An open site filter's
`defaultOrgNodes` apply to the default structure only; other structures start
at their root. Notifications always use the default structure. Dashboards have
their own org structure switch (see `reporting dashboards` above).

#### Lint a capture or report template

```bash
cap templates capture-templates lint --file template.json [--json]
cap templates report-templates lint --file template.json [--json]
```

`lint` checks a template JSON file (the same payload as `create` and `save`)
without saving it. A template that lists its metrics (`spreadsheetTemplateMetrics`) ignores the metric
attribute filters and the discipline and framework attribute filters, and also the disciplines and
frameworks (limits, defaults and viewer filters) unless they scope narratives that are shown and not
listed; one that lists narratives ignores the narrative attribute filters. `lint`, `save` and preview
clear what is ignored. **Errors** are exactly what `save` would reject; **warnings**
never block a save. Each issue has a stable `code`, a `message`, the top-level
`field` it belongs to (camelCase, empty when not tied to one field) and an
editor `stage` (`data`, `layout` or `look`). An issue whose
message carries values (a limit, a range, narrative names) also has `params`,
by camelCase name, which `--json` passes through; the text output keeps
printing `message`. The exit code
is `2` when there is at least one error and `0` otherwise. `--json` prints the
API result unchanged: `{ "errors": [...], "warnings": [...] }`.

| Warning code | Template | Meaning |
|---|---|---|
| `RowTypesExcludeBoundNarratives` | capture, report | Narratives are bound but Narrative is not in `includedDataTypes` (empty means every type). |
| `RowTypesExcludeBoundCalculations` | report | Calculation metrics are bound but Calculation is not in `includedDataTypes`. |
| `SiteDefaultMissing` | capture, report | The site filter is open and single-select, `orgNodes` limits it to more than one site, and `defaultOrgNodes` is empty: viewers start with no site. |
| `IntervalNotSet` | capture, report | `dataInterval` is not set, so the template shows days. |
| `NarrativesOutsideIntervalOrScope` | capture, report | Bound narratives no longer match `dataInterval`, the Limit to `orgNodes` or the Limit to `disciplines`. |

Other error codes include `SingleOrgNodeRequired`, `SingleDefaultOrgNodeRequired`,
`DefaultOutsideLimit`, `SiteNotInStructure`, `StructureSwitchReportOnly`,
`DefaultStructureOutsideLimit`, `SiteLimitRequiresFixedStructure`,
`FixedSitesRequireFixedStructure`, `SwitchableStructureRequiresOpenSites`,
`MetricsInColumnsRequiresGrouping`, `TimePeriodsInRowsRequiresMetricsInColumns`,
`ExpandCalculationsRequiresRows`, `DuplicateNarratives`, `UnknownNarratives`
(a bound narrative that does not exist in the tenant), `UnknownOrgNodeTemplate`
and `BandMetricReferenceInvalid` (a range compares with its own metric or one
that does not exist). A range whose limit metric's unit has no conversion to the
row metric's unit reports `BandMetricUnitNotConvertible`, as input and calculation
lint do; widget templates report both codes the same way. Save refuses exactly
what `lint` reports as an error.

A template range that breaks a range rule reports that rule's own code, the same
code a metric or a widget reports: `BandsTooMany`, `BandFirstHasLowerBound`,
`BandLowerBoundRequired`, `BandLowerBoundNotAscending`, `BandBoundsReversed`,
`BandBoundHasValueAndMetric`, `BandColourNotToken`, `BandColourContrastTooLow`,
`BandRequireCommentNotAllowed` or `BandAutoValidateNotAllowed`, with `metric`,
`band` (the range's number, from 1) and `path` params. These replace the single
`BandsInvalid` code earlier versions reported. Overlapping ranges (`BandOverlap`)
and a gap between ranges (`BandCoverageGap`) are notes: they never block a save
or an Excel upload. Excel row errors and save errors on ranges start with the
code, for example `[BandBoundsReversed] bands[1].lowerBoundValue must be less
than or equal to upperBoundValue.`

#### Resolve what a capture or report template contains

```bash
cap templates capture-templates resolve-content (--file template.json | --id <guid>) [--json]
cap templates report-templates resolve-content (--file template.json | --id <guid>) [--json]
```

`resolve-content` lists the metrics and narratives a template resolves to,
sorted by name: the same list the template editor shows behind "See list", and
exactly as many items as the `counts` that `lint` returns. Give exactly one of
`--file` (a template JSON file, the same payload as `lint`) or `--id` (a saved
template, read first and sent as stored). Each template kind needs the same permission as its `lint` command (`Model.Manage` for capture templates, `Templates.Manage` for report templates). The template's own scope applies (limit picks and
attribute filters); your data permissions and viewer filters do not. A metric
excluded at every site a viewer can pick is not listed; one excluded at only some
of them is. Those sites are the "Limit to" sites (the Default sites, else the
"Limit to" ones, when the site filter is fixed); without picked sites,
exclusions are not applied. Exclusions never cascade to child sites. A list is
empty when the row types exclude it.

Text output is `Metrics (n)` and one name per line, a blank line, then
`Narratives (n)` the same way. `--json` prints the API result unchanged:
`{ "metrics": [{ "id": "...", "name": "..." }], "narratives": [...] }`.

After a successful `create` or `save` of a capture or report template, the CLI
runs `lint` and prints each warning to stderr as
`Warning: [Code] field: message`. With `--json`, the result is one document:
`{ "success": true, "id": "...", "warnings": [...] }`. If the warning check
itself fails, the save still succeeds and the CLI says it could not check; a
server without the Lint endpoint (404) is skipped silently.

`upload-excel` for capture and report templates applies the same rules before
saving. A template that `save` would reject is skipped and reported as an upload
error; the other templates in the file still save. Warnings are reported and the
template saves. Both use row 0 with the template name in front, for example
`Site Monthly: Narratives are selected, but Narratives is not one of the row
types, so they will not show.` The checks use the narratives in the sheet, or
the template's stored narratives when the sheet has no Narrative Name column.

The capture and report template workbooks carry each template's narratives in
a **Narrative** column group (`Narrative Name`, `Narrative Sort Order`), one
narrative per row below the template row, beside the metric rows. Download
writes names in the request language (fallback default), ordered by sort order.
On upload:

- With the `Narrative Name` column, the sheet's rows replace the template's
  narratives; a template with no narrative rows has its narratives removed.
- Without the column (a workbook downloaded before it existed), the stored
  narratives are kept.
- An unknown or ambiguous narrative name, or a GUID, is a row error on that
  cell. That template is not saved; the other templates in the file still
  save. Upload narratives (`model narratives`) before the templates.

### Update Existing Item

```bash
cap <domain> <entity> save --file input.json [--json]
# OR
echo '{...}' | cap <domain> <entity> save [--json]
```

**Note:** Input JSON must include the `id` field for updates. For widget templates using auto-wrap, place the `id` in the inner object (alongside `name`, `widgetType`, etc.) - the CLI preserves it when wrapping.

### Create Widget Template

Widget templates support auto-wrapping - you can provide the inner object without the `widgetTemplate` wrapper:

```bash
# InfoCard (auto-wrapped - recommended)
echo '{
  "name": "Energy Usage Card",
  "title": "Energy Usage",
  "footnote": "#trend[<delta-calculation-metric-id>] since @[period:-1]",
  "widgetType": { "id": 0, "name": "InfoCard" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "styleConfiguration": {
    "panel": { "borderRadius": 8, "borderColor": "border", "borderWidth": 1, "backgroundColor": "surface", "fontFamily": "theme" },
    "title": { "foregroundColor": "text-primary", "fontWeight": "Bold", "fontSize": 14, "borderWidth": 0 },
    "value": { "foregroundColor": "text-primary", "fontWeight": "Bold", "fontSize": 32 },
    "trend": {
      "accentColor": "neutral",
      "fontWeight": "Normal",
      "showLabel": false,
      "up": { "color": "success", "icon": "arrow-up" },
      "down": { "color": "danger", "icon": "arrow-down" },
      "flat": { "color": "neutral", "icon": "equals" },
      "unknown": { "color": "neutral", "icon": "none" }
    }
  },
  "dataItems": [
    {
      "metric": { "id": "<metric-id>" },
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "sortOrder": 1
    },
    {
      "metric": { "id": "<delta-calculation-metric-id>", "name": "Delta Calculation metric" },
      "name": "Delta",
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "sortOrder": 2
    }
  ]
}' | cap templates widget-templates create --json

Info Card comparisons use ordinary selected metrics, Calculation metrics, and display-property tokens. Raw API/CLI JSON uses stored GUID-form tokens such as `[metric-guid]`, `[metric-guid]|-1|`, `#trend[metric-guid]`, `{narrative-guid}`, or `{narrative-guid}|-1|`; the widget editor and Widget Template Excel use metric names (`[Metric Name]`) and narrative names (`{Narrative Name}`) and transpose them to GUIDs internally. Narrative tokens are allowed only in TextBlock, InfoCard and Table title, subtitle (TextBlock), description, and footnote; find narrative ids with `cap model narratives list --json`. Widget Template Excel preserves the same bounded style contract in the `Info Card Style` JSON column; do not add target, benchmark, prior, delta, or delta-percent columns or CLI-only fields.

Info Card style JSON is schema-limited. `fontFamily` must be one of `theme`, `sans`, `serif`, `mono`, `nunito`, `roboto`, `poppins`, or `arial`; `fontSize`, `borderWidth`, and panel `accentWidth` are bounded integers. `accentSide` and `accentWidth` are panel-only fields. A border renders only with positive `borderWidth` plus safe `borderColor`; a panel accent edge renders only with `accentSide`, positive `accentWidth`, and safe `accentColor`.

Trend presentation is configured per Info Card through `styleConfiguration.trend`. Direction keys are `up`, `down`, `flat`, and `unknown`; each direction accepts a registry `color` token or hex value and a bounded `icon` token. Use `icon: "none"` for color-only trends. Do not use raw CSS classes, FontAwesome class names, HTML, scripts, `rgb(...)`, `var(...)`, or `url()` values.

# PieChart
echo '{
  "name": "Waste Breakdown",
  "widgetType": { "id": 1, "name": "PieChart" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 0, "name": "Dynamic" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "showLegend": true,
  "pieChartWidgetTemplateType": { "id": 1, "name": "Donut Chart" },
  "centerMode": { "id": 2, "name": "Metric Value" },
  "centerStaticText": null,
  "centerMetricId": "<total-waste-metric-id>",
  "centerMetricLabel": null,
  "centerNumberFormat": { "precision": 2, "magnitude": "Millions" },
  "legendValueMode": { "id": 1, "name": "Value" },
  "sliceLabelMode": { "id": 3, "name": "Label and Value" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "styleConfiguration": {
    "panel": { "backgroundColor": "surface", "borderColor": "border", "borderWidth": 1, "borderRadius": 8, "padding": 20 },
    "title": { "foregroundColor": "text-primary", "fontWeight": "Bold", "fontSize": 18 },
    "chartArea": { "backgroundColor": "surface", "padding": 8 },
    "slices": { "opacity": 0.9 },
    "sliceBorders": { "borderColor": "surface", "borderWidth": 2 },
    "ticks": { "borderColor": "border", "borderWidth": 1, "opacity": 0.75 },
    "legendValues": { "foregroundColor": "text-secondary", "fontWeight": "Bold", "fontSize": 13 },
    "donutCenter": { "foregroundColor": "text-primary", "fontWeight": "Bold", "fontSize": 28 },
    "stateMessages": { "foregroundColor": "warning" }
  },
  "dataItems": [
    {
      "metric": { "id": "<metric-id>" },
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "showInLegend": true,
      "showTooltip": true,
      "sortOrder": 1,
      "presentation": {
        "slice": { "backgroundColor": "#4a90d9" },
        "legendValue": { "fontWeight": "Bold", "foregroundColor": "text-secondary" },
        "tooltip": { "fontSize": 12, "foregroundColor": "text-primary" },
        "sliceLabels": { "fontSize": 12, "foregroundColor": "text-primary" }
      },
      "numberFormat": { "precision": 1, "magnitude": "Thousands" }
    }
  ]
}' | cap templates widget-templates create --json

# XYChart (time series) - axes required, each data item must reference an axis.
# None is intentional here because the chart keeps one point per period.
echo '{
  "name": "Monthly Trend",
  "widgetType": { "id": 2, "name": "XYChart" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 0, "name": "Dynamic" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "showLegend": true,
  "invertAxes": false,
  "styleConfiguration": {
    "categoryAxis": { "labelRotation": 45, "labelDensity": "Comfortable" },
    "series": { "strokeWidth": 2 }
  },
  "axes": [{ "name": "Value", "dynamicAxis": true }],
  "timePeriodAggregationMethod": { "id": 0, "name": "None" },
  "dataItems": [
    {
      "metric": { "id": "<metric-id>" },
      "xyChartWidgetTemplateDataItemType": { "id": 2, "name": "Line" },
      "xyChartWidgetTemplateAxis": { "name": "Value", "dynamicAxis": true },
      "presentation": {
        "series": { "color": "#1565C0" }
      },
      "timePeriodAggregationMethod": { "id": 0, "name": "None" },
      "showInLegend": true,
      "showTooltip": true,
      "sortOrder": 1
    }
  ]
}' | cap templates widget-templates create --json

# Table dashboard table with Dynamic Narrative Scope
echo '{
  "name": "Portfolio KPI Matrix",
  "title": "Portfolio KPI Matrix",
  "widgetType": { "id": 4, "name": "Table" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 3, "name": "Quarter" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "dataGrouping": { "id": 1, "name": "OrgNode" },
  "orgNodeRowSelectionMode": { "id": 0, "name": "Children" },
  "showOnlyRowsWithValues": true,
  "showMetricValue": true,
  "metricPropertyColumns": [{ "id": 0, "name": "Unit of Measure" }],
  "showMetricsInColumns": false,
  "missingValuePlaceholder": "-",
  "dataItems": [
    {
      "metric": { "id": "<revenue-metric-id>", "name": "Revenue" },
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "sortOrder": 0,
      "bands": [
        { "backgroundColor": "danger-subtle", "foregroundColor": "text-primary" },
        { "lowerBoundValue": 90, "backgroundColor": "warning-subtle", "foregroundColor": "text-primary" },
        { "lowerBoundValue": 95, "backgroundColor": "success-subtle", "foregroundColor": "text-primary" }
      ]
    }
  ],
  "metricAttributeTypeIds": ["<sector-attribute-type-id>"],
  "narrativeSelectionMode": { "id": 0, "name": "Dynamic" },
  "narrativeScopeConfigured": true,
  "narrativeDisciplineFilters": [
    { "id": "<discipline-id>", "name": "Finance" }
  ],
  "narrativeFrameworkFilters": [],
  "narrativeMetricAttributeFilters": [],
  "narrativeDisciplineNodeAttributeFilters": [],
  "narrativeFrameworkNodeAttributeFilters": [],
  "narrativeAttributeFilters": [],
  "narratives": [],
  "styleConfiguration": {
    "rowLabelHeader": "Company",
    "zebraStripeColor": "surface",
    "gridHeader": {
      "foregroundColor": "text-primary",
      "fontWeight": "Bold",
      "textTransform": "Uppercase",
      "letterSpacing": 1
    },
    "valueCells": {
      "textAlign": "End"
    }
  }
}' | cap templates widget-templates create --json
```

> **Note:** The CLI auto-wraps JSON in `{"widgetTemplate": {...}}` if the root object has `name` or `widgetType` properties. You can also use the explicit wrapper format if preferred.

### Widget Value Selection

Widget templates can carry advanced `valueSelectionConfig` in JSON create/save
payloads and widget-template Excel import/export. The
same validator runs for API, CLI JSON, and Excel upload, so unsupported matrix
cells and unknown nested fields fail consistently.

Use schema and sample commands to inspect the editable shape:

```bash
cap templates widget-templates schema --widget-type info --json
cap templates widget-templates schema --widget-type pie --json
cap templates widget-templates schema --widget-type xy --json
cap templates widget-templates schema --widget-type table --json
cap templates widget-templates schema --widget-type textblock --json
cap templates widget-templates sample --widget-type pie --json
```

#### Dynamic Metric Selection And Metric-Set Value Selection

`widgetTemplate.metricSelectionMode` controls whether a widget lists explicit
metrics or resolves metrics from scope filters:

| Mode | `metricSelectionMode` | Behavior |
|------|------------------------|----------|
| **Static** | `{ "id": 0, "name": "Static" }` | You list each metric explicitly in `widgetTemplate.dataItems[]`. |
| **Dynamic** | `{ "id": 1, "name": "Dynamic" }` | The metrics are resolved at render time from the metric-scope filters below. A dynamic Table template may keep `dataItems` as overrides of the matching metrics they name (label, `numberFormat`, `bands`); they do not limit or order the metrics. Info Card/Pie dynamic samples omit explicit `dataItems`. |

In Dynamic mode the metric set is scoped by these widget-level filter arrays.
Every array is optional; an **empty array selects all** along that axis, and you
add `{ "id", "name" }` entries to narrow it:

| Filter field | Narrows the set by |
|--------------|--------------------|
| `metricTypeFilters` | Metric type: `{ "id": 0, "name": "Input" }` and/or `{ "id": 1, "name": "Calculation" }` |
| `metricDisciplineFilters` | Discipline node(s) |
| `metricFrameworkFilters` | Framework node(s) |
| `metricAttributeFilters` | Metric attribute value(s) |
| `disciplineAttributeFilters` | Primary discipline attribute filter object(s), used by current Table metric scope |
| `frameworkAttributeFilters` | Primary framework attribute filter object(s), used by current Table metric scope |
| `disciplineNodeAttributeFilters` | Discipline org-structure attribute value(s) |
| `frameworkNodeAttributeFilters` | Framework org-structure attribute value(s) |

For Table templates in Dynamic metric mode, author at least one non-empty
metric-scope filter family: `metricDisciplineFilters`,
`metricFrameworkFilters`, `metricAttributeFilters`, `disciplineAttributeFilters`,
or `frameworkAttributeFilters`. A table ignores `metricTypeFilters`: its row
types (`includedDataTypes`) say which metric types it shows, and `save` folds
any metric types it is sent into them. Empty arrays still mean "all" for an authored
axis once Dynamic mode has a non-empty filter family.

A Table template has one period window in every `dataRangeMode`: with Static,
set the widget-level `dataPeriodStart` and `dataPeriodEnd` (lint error
`PeriodWindowRequired` when missing); with SelectedYear (id 2) the window is the
periods of the viewer's year up to their last selected period, and no offsets are
set. A table refuses SelectedYearToLatestData (id 3) with
`DataRangeModeNotSupported`. Table `dataItems[]` take no window.

To sort and Top/Bottom-rank either a dynamically-resolved Info Card/Pie metric
set or an explicit static chart/card metric set, attach a **widget-level**
`valueSelectionConfig` with `ownerScope: "WidgetDynamicMetricSet"` (see the
example further below). In Static mode, keep explicit `dataItems[]`; the engine
treats the candidates as a static metric set and updates the render order from
the selected values. This is distinct from partitioned per-data-item selection,
which uses `ownerScope: "DataItem"` on `dataItems[].valueSelectionConfig`. Do not
combine widget-level static metric-set selection with data-item-level selection
on the same static chart/card.

**Dynamic metric-set value selection** is limited to Info Card and Pie Chart.
XY Chart has no dynamic metric discovery (use explicit static XY data items).
Table widgets do support Dynamic metric selection through Metric Scope
(discipline/framework filters and attribute filters), but
Table ranking/filtering uses the separate `TableRows` owner scope over resolved
rows, not `WidgetDynamicMetricSet`. Run
`cap templates widget-templates schema --widget-type table --json` to inspect
the Table metric, narrative, custom-column, and style fields.

Supported owner scopes:

| Scope | JSON location | Supported widgets |
|-------|---------------|-------------------|
| `DataItem` | `widgetTemplate.dataItems[].valueSelectionConfig` | Info Card, Pie Chart, XY Chart partitioned data items |
| `WidgetDynamicMetricSet` | `widgetTemplate.valueSelectionConfig` | Static Info Card, Pie Chart, and XY metric sets; Dynamic Info Card and Pie Chart metric sets |
| `TableRows` | `widgetTemplate.valueSelectionConfig` | Table row filtering, ordering, and Top/Bottom rank |

XY Chart does not support Dynamic mode metric-set discovery; create explicit
static XY data items, then use widget-level `WidgetDynamicMetricSet` selection
to order or Top/Bottom-rank that configured set.

All enum values in `valueSelectionConfig` are serialized as **bare strings**
(e.g. `"rankMode": "Top"`), not `{ "id", "name" }` objects. The config object
rejects unknown members, so misspelled or stale fields fail validation.

**Config fields:**

| Field | Type / values | Rule |
|-------|---------------|------|
| `ownerScope` | `DataItem`, `WidgetDynamicMetricSet`, `TableRows` | Must match the JSON location (see table above) |
| `valueSelector` | object (see selector kinds below) | Required |
| `orderMode` | `None`, `ValueAscending`, `ValueDescending` | |
| `rankMode` | `None`, `Top`, `Bottom` | |
| `rankLimit` | integer `1`..`1000` (MaxPageSize) | Required when `rankMode` is `Top`/`Bottom`; must be omitted otherwise |
| `nullPlacement` | `Last`, `DefaultLast` | Only `Last`/`DefaultLast` are authorable; authoring `First` is rejected |
| `tableRowScope` | `All`, `LeafOrgNodes` | Optional, TableRows only. `LeafOrgNodes` filters OrgNode + metrics-as-columns tables to leaf row candidates before predicates, order, and Top/Bottom rank |
| `predicates` | array, max 5 | Filters applied before ranking (see below) |
| `diagnosticsLabel` | string (optional) | Free-text label echoed in selection diagnostics output |
| `boundsWarningMode` | `Default`, `Warn`, `Suppress` (optional) | Controls advisory warnings when the projected element count is large |

**Selector kinds** (`valueSelector.selectorKind`):

| Scope | Supported `selectorKind` | Required selector fields |
|-------|--------------------------|--------------------------|
| `DataItem`, `WidgetDynamicMetricSet` | `AggregatedRange` | `aggregationMethod` (`Sum`/`Average`/`LastValue`/`Min`/`Max`), `fallbackWhenMissing` (`None`/`TreatAsNull`/`Reject`) |
| `TableRows` | `TableMetricAggregate`, `TableRowAggregate` | `aggregationMethod`, `fallbackWhenMissing`; `TableMetricAggregate` also takes `tableColumnKey` (e.g. `"metric:<metric-guid-n>"`). `TableRowAggregate` must not carry `tableColumnKey`. |

**How a table ranks.** A table row ranks, and is filtered, on the value stored for its metric at its site for the
viewer's period: the latest period the viewer picks (with fixed periods, the period they run up to), at that period's
own type, whatever interval the columns show. A table with quarter columns and FY 26 picked ranks on each site's FY 26
value. A table that fixes a coarser interval (`showDataIntervalFilter: false`) ranks on that interval's period holding
the picked one: a yearly table with Mar 2026 picked ranks on the year holding March. The table never adds up or averages its columns to rank, so `aggregationMethod` is still required but a table
does not read it. A site with no stored value for that period ranks last; ties keep the site path order. Narrative
rows never take a rank slot: each kept site's narratives follow it. With sites on columns a row has no site, so it has
no value to rank or filter on: rank needs OrgNode on rows (`ValueSelectionNeedsSiteGrouping`).

**Predicates** (`predicates[]`, max 5, applied before ranking):

| Field | Type / values | Rule |
|-------|---------------|------|
| `predicateType` | `Value`, `Null` | |
| `operator` | Value: `GT`, `GTE`, `LT`, `LTE`, `EQ`, `NE`, `Between`. Null: `IsNull`, `IsNotNull` | Operator set depends on `predicateType` |
| `operandKind` | `Number`, `Metric` (Value predicates only) | Defaults to `Number` |
| `lowerValue` | number | `Number` operand: required for all Value operators |
| `upperValue` | number | `Number` operand: required only for `Between` |
| `metricId` / `metricStableKey` | GUID / stable key | `Metric` operand: exactly one required; cannot use `Between`; cannot carry numeric values |
| `tolerance` | number ≥ 0 (optional) | Negative tolerance is rejected |

`Null` predicates carry no operand fields (no `operandKind`, `lowerValue`,
`upperValue`, `metricId`, or `metricStableKey`). `Metric` operand predicates are
not supported in dynamic metric-set selection (`WidgetDynamicMetricSet`); use
`Number` operands there.

Example dynamic Pie Top 5 (with a Number-operand filter):

```json
{
  "name": "Top Inputs | Pie",
  "widgetType": { "id": 1, "name": "Pie Chart" },
  "metricSelectionMode": { "id": 1, "name": "Dynamic" },
  "valueSelectionConfig": {
    "ownerScope": "WidgetDynamicMetricSet",
    "valueSelector": {
      "selectorKind": "AggregatedRange",
      "aggregationMethod": "Sum",
      "fallbackWhenMissing": "TreatAsNull"
    },
    "orderMode": "ValueDescending",
    "rankMode": "Top",
    "rankLimit": 5,
    "nullPlacement": "Last",
    "tableRowScope": "LeafOrgNodes",
    "predicates": [
      {
        "predicateType": "Value",
        "operator": "GTE",
        "operandKind": "Number",
        "lowerValue": 0
      }
    ],
    "diagnosticsLabel": "Top 5 inputs by sum",
    "boundsWarningMode": "Warn"
  }
}
```

Example Table row selection (Top 5 rows by a metric column, with a Metric-operand filter):

```json
{
  "name": "Top Sites | Table",
  "widgetType": { "id": 4, "name": "Table" },
  "valueSelectionConfig": {
    "ownerScope": "TableRows",
    "valueSelector": {
      "selectorKind": "TableMetricAggregate",
      "aggregationMethod": "Sum",
      "tableColumnKey": "metric:<metric-guid-n>",
      "fallbackWhenMissing": "TreatAsNull"
    },
    "orderMode": "ValueDescending",
    "rankMode": "Top",
    "rankLimit": 5,
    "nullPlacement": "Last",
    "predicates": [
      {
        "predicateType": "Value",
        "operator": "GT",
        "operandKind": "Metric",
        "metricStableKey": "<threshold-metric-guid-n>"
      }
    ]
  }
}
```

> `tableColumnKey` uses the stored `metric:<guid>` form where `<guid>` is the
> metric ID without dashes (the `N` GUID format). The save-ready `table` sample
> uses `{ "predicateType": "Null", "operator": "IsNotNull" }`. Comparing a
> column with itself filters most rows.

Partitioned data items (Children/Descendants) support per-data-item selection at
`widgetTemplate.dataItems[].valueSelectionConfig` with `ownerScope: "DataItem"`;
it is rejected on unpartitioned items. Run
`cap templates widget-templates sample --widget-type info --json` for a
ready-to-edit example that carries the widget-level static metric-set
`valueSelectionConfig` instead.

Excel stores `valueSelectionConfig` as a single JSON-in-cell column. A blank cell
means no value-selection config, and upload rejects stale/removed fields such as
`changeFilter`, `moversMode`, and `includeOnlyChanged`, as well as removed
change-typed predicate types (`AbsoluteChange`, `PercentChange`, `SignChange`).

Pie and donut templates use the same shared WidgetTemplate API and CLI command surface as the existing widget types. Manage them with `cap templates widget-templates list|get|get-bulk|create|save|delete|download-excel|upload-excel`, and render live data with `cap reporting widgets pie-chart ...`.

**Pie/donut display fields:**

| JSON field | Excel column | Values / default | Rule |
|------------|--------------|------------------|------|
| `pieChartWidgetTemplateType` | `Pie Chart Type` | `0` Pie Chart, `1` Donut Chart | Required for pie widgets (`PieChartTypeRequired`) |
| `centerMode` | `Center Mode` | `0` None, `1` Static Text, `2` Metric Value; default `None` | Donut Chart only (`PieCenterNeedsDonut`) |
| `centerStaticText` | `Center Static Text` | Free text | Required when `centerMode.id` is `1`; disallowed for Metric Value |
| `centerMetricId` | `Center Metric` (by name) | Metric GUID in JSON; `""` or `null` is not picked | Required when `centerMode.id` is `2`; disallowed for Static Text |
| `centerMetricLabel` | `Center Metric Label` | Optional text | Optional override for Metric Value centers; if blank, render uses metric friendly name, then metric name |
| `legendValueMode` | `Legend Value Mode` | `0` Hidden, `1` Value; default `Value` | Stored enum behind the UI's **Show Legend Value** checkbox; controls numeric values in visible legend rows, not row visibility |
| `sliceLabelMode` | `Slice Label Mode` | `0` Hidden, `1` Label, `2` Value, `3` Label and Value; default `Hidden` | Controls labels rendered on slices, separate from tooltips and legend rows |

Widget-template Excel download/upload includes the display columns above. On upload, `Center Metric` names the center metric (an exact-name lookup). Legacy workbooks that omit the new columns continue to import with defaults.

The dashboard editor presents `centerMetricId` through the searchable metric grid picker. The CLI and Excel contracts still use the explicit metric ID/name fields so imports remain stable and diffable.

Table templates use the same shared WidgetTemplate API and CLI command surface as the existing widget types. Manage them with `cap templates widget-templates list|get|get-bulk|create|save|delete|download-excel|upload-excel`, and set the widget type to `{ "id": 4, "name": "Table" }` in JSON or the corresponding Table widget type in Excel. Table data comes from selected Input or Calculation metrics, their ComputedValues, and optional dynamic or static narrative rows; do not encode hidden widget-only calculations in the template.

TextBlock templates use the same shared WidgetTemplate API, CLI command surface, and widget-template Excel download/upload workflow. Manage them with `cap templates widget-templates list|get|get-bulk|create|save|delete|download-excel|upload-excel`, set the widget type to `{ "id": 6, "name": "TextBlock" }`, and render live data with `cap reporting widgets text-block ...`. TextBlock discovers metrics and narratives from explicit tokens in `title`, `subtitle`, `description`, and `footnote` (for example `"description": "{<narrative-id>}|-1|"`); do not create hidden `dataItems[]` rows for those references. Excel roundtrips `Subtitle` and `Text Block Style` columns alongside the existing title/description/footnote fields.

AI Summary templates use the same shared WidgetTemplate API and CLI command surface. Set the widget type to `{ "id": 3, "name": "AISummary" }`; `cap templates widget-templates sample --widget-type ai-summary --json` emits a save-ready one. `scope` is `Peers` (0, the widgets beside its placement), `Descendants` (1, its section and below) or `PickedMetrics` (2, the metrics in `dataItems`, each with the same `metricPartitioningMode`: `None`, `Children` or `Descendants`). A section scope keeps no `dataItems`. `leaveOutWidgetTypes` drops widget types from a section scope; `maxInsights` is 1 to 8 (`AISummaryMaxInsightsOutOfRange`); `insightTypes` needs at least one kind (`AISummaryInsightTypesRequired`); `insightOrder` is `AsWritten` or `SeverityFirst`. A dashboard placement points at the template like any other widget. Excel roundtrips the AI Summary columns (Scope, Leave Out, Max Insights, Insight Types, Order, AI Summary Style).

**Widget Types:**
| ID | Name | Use Case |
|----|------|----------|
| 0 | InfoCard | Single KPI value |
| 1 | PieChart | Part-to-whole breakdown |
| 2 | XYChart | Time series / trends |
| 3 | AISummary | Checked AI insights about a section's widgets or picked metrics |
| 4 | Table | Dashboard grid/table render |
| 6 | TextBlock | Metric-aware dashboard text |

**Table validation highlights:**
| Area | Rule |
|------|------|
| `dataGrouping` | Required; primary row grouping. `additionalDataGrouping` holds optional Then By levels (0–2). OrgNode anywhere in the chain requires `orgNodeRowSelectionMode`. Framework anywhere requires Dynamic metric selection. |
| Layout | `showMetricsInColumns` needs a grouping, or `showTimePeriodsInRows` (`MetricsInColumnsRequiresGrouping`). `showTimePeriodsInRows` needs `showMetricsInColumns` (`TimePeriodsInRowsRequiresMetricsInColumns`). `columnDataGroupingLevels` must be chain levels and leave a row and a column (`ColumnDataGroupingLevelOutsideChain`, `LayoutRequiresRow`, `LayoutRequiresColumn`). `showTotalRow` outside an OrgNode-only, metrics-as-columns layout is a warning (`TotalRowNeedsSiteOnly`). `valueSelectionConfig` (order and limit) needs OrgNode on rows: in the chain and not in `columnDataGroupingLevels` (`ValueSelectionNeedsSiteGrouping`; a warning on tables saved before the rule). `showDataIntervalFilter: false` (the table fixes its interval) needs a `dataInterval` (`IntervalFixedNeedsInterval`). `pageSize` (Rows per page) is 1 to 1000, default 50 (`PageSizeOutOfRange`). `heightMode` is `0` Fixed or `1` FitRows (`UnknownHeightMode`); `maxHeightPx` is 200 to 2000, default 600, checked only for Fixed (`MaxHeightOutOfRange`). `defaultExpandedLevels` (Open levels) is 0 to 10, default 10, every group open (`DefaultExpandedLevelsOutOfRange`). Left out, an update keeps the stored value; the CLI drops a JSON `null`, so send the default value (`50`, `0`, `600` and `10`) to reset one. |
| Tooltips | `metricTooltipProperties` and `metricTooltipAttributeTypeIds` choose the metric-name tooltip, with the same rules as `metricPropertyColumns` and `metricAttributeTypeIds`. |
| Metric selection | Metrics are listed or filtered, never both. Static tables use explicit `dataItems[]` and ignore the metric filters, which `save` clears (a Static table with filters and no `dataItems[]` saves as Dynamic); Static narratives likewise ignore the narrative scope. Dynamic tables use discipline, framework, metric-attribute, discipline-attribute, and framework-attribute filters and clear explicit data items on save. Metric types come from the row types (`includedDataTypes`). |
| Org-node scope | `orgNodeAttributeFilters` filters the resolved org-node scope before values are loaded; it is not the deferred `OrgNodeAttribute` row-grouping mode. |
| Custom columns | `showMetricValue`, `metricPropertyColumns` (`MetricProperty` EnumDTO list), and `metricAttributeTypeIds` control visible helper/value columns. When `showMetricValue` is false, at least one of `metricPropertyColumns` or `metricAttributeTypeIds` must be non-empty. |
| `narrativeSelectionMode` | `Dynamic` (`id: 0`) resolves narratives from Narrative Scope. `Static` (`id: 1`) renders explicit `narratives[]`. |
| `narrativeScopeConfigured` | Set `true` when Dynamic Narrative Scope is intentionally authored. Empty scope arrays then mean all permitted values; `false`/omitted preserves legacy metric-scope fallback. |
| Narrative Scope | Dynamic scope can filter by `narrativeDisciplineFilters`, `narrativeFrameworkFilters`, `narrativeMetricAttributeFilters`, `narrativeDisciplineNodeAttributeFilters`, `narrativeFrameworkNodeAttributeFilters`, and `narrativeAttributeFilters`. |
| Static narratives | `narratives[]` contains `{ narrative: { id, name }, sortOrder }` entries and is used only when `narrativeSelectionMode` is Static. |
| Org-node template | Do not author `orgNodeTemplateId` on new Table widgets; the table runs on the org structure the dashboard is showing. |
| Included data types | Do not use `includedDataTypes` as a new authoring control; it is an internal spreadsheet-adapter compatibility field. |
| Style | `styleConfiguration` uses the bounded Table style contract for slots, `rowLabelHeader`, `zebraStripeColor`, column overrides, row overrides, conditional formatting, and categorical color tags. |
| Deferred columns/grouping | Arbitrary static-text columns, org-node-attribute display columns, expression columns, and `OrgNodeAttribute` row grouping are not current authoring fields. |
| Render budget | The platform applies table render budgets server-side; budget values are not designer JSON inputs. |
| `--help` | Available on `cap templates widget-templates --help`, `cap reporting widgets --help`, and each widget subcommand |

See [Create Widget Template Recipe](../recipes/configuration/create-widget-template.md) for detailed examples.

### Delete Item

```bash
cap <domain> <entity> delete <id> [--json]
```

**Example:**
```bash
cap model inputs delete <id>
```

**Delete several inputs or calculations at once.** `cap model inputs delete` and `cap model calculations delete` also take `--ids` (`-i`): comma- or whitespace-separated GUIDs on one flag. A positional `<id>` is added to the list; duplicates are ignored, and an all-zero GUID counts as invalid. Repeated `--ids` flags last-win, so put every ID on one flag.

```bash
cap model calculations delete --ids <id1>,<id2>,<id3> --json
cap model inputs delete --ids "$(cat input-ids.txt)" --json     # one ID per line works too
```

- All IDs go into one delete operation: one preview, one start, one wait, and one model change. Calculations that use each other can be deleted together; one-at-a-time deletes are blocked unless the dependent is deleted first.
- The API takes at most 5000 IDs per delete and rejects more with `ExplicitIdsLimitExceeded`. A single `--ids` value is also limited by the shell: about 3,000 IDs on Linux and about 800 on Windows. Split longer lists into several commands.
- An ID that is not a GUID, or is all zeros, fails with exit code `2` before anything is sent.
- IDs that do not exist or are the wrong metric type (an input ID passed to `calculations delete`) are rejected by the preview; nothing is started for them. Calculations or inputs still used outside the set are kept, not deleted: a formula, band or unit conversion, a widget, capture or report template, a dashboard, or a narrative. Nothing is removed from those records. The text output prints `Kept, still in use:` with every use as `<record>: <kind>: <name>`; remove those uses first, then delete again.
- Exit codes: `0` when every ID was deleted; `6` (partial result) when some were deleted and others were rejected or blocked; `2` when nothing was deleted; `4` when the operation failed; `5` on timeout. In `--json` output, `success` is `true` when at least one ID was deleted, so a partial delete has `success: true` and exit code `6`.
- `--json` returns `entityIds` (the distinct IDs sent), `entityId` (only for a single ID), `operationId`, `status`, the `deletedCount`, `blockedCount`, `rejectedCount` and `warningCount` counts, and `resultSummary` with the first 10 reasons per group, plus `inUse` (every use of the kept records, as `item`, `kind`, `name`) and `inUseSummary`. The API reports counts and reasons, not a per-ID outcome list.
- The operation keeps running on the server after `--timeout-seconds` (default 600). Check it with:

```bash
cap model metrics delete-status <operation-id> [--json]
```

`delete-status` reports the operation's state only: exit `4` when it failed, otherwise `0`, including while it is still running or when some IDs were blocked. Read `status` and the counts to tell a full delete from a partial one.

**Metrics and narratives in use are kept.** A delete never removes a metric (input or calculation) or a narrative from the records that use it.
- A metric is kept while a calculation formula, another metric's band, an org-node override's band, a unit conversion factor, a widget or spreadsheet template (data item, row, band, pie centre or `[metric-guid]` text token), a dashboard header token or a narrative's metric link uses it.
- A narrative is kept while a table widget, a spreadsheet capture template, a `{narrative-guid}` text token, a dashboard header token or a narrative placed on a dashboard uses it.
- The delete preview lists the kept records and every use (API: `keptCount`, `deletableCount`, `inUse` and `inUseSummary` on the preview response; UI: a "kept" panel with "See every use" in the delete dialog). The delete removes only the others.
- The delete checks again just before it removes anything, so a use added after the preview keeps that record too. A kept record keeps its values.

### Delete Tenant

```bash
cap system tenants delete <tenant-id-or-exact-name> [--force] [--yes] [--timeout-seconds 300] [--json]
cap system tenants delete-status <operation-id> [--json]
cap system tenants delete-watch <operation-id> [--timeout-seconds 300] [--json]
```

`cap system tenants delete` accepts a tenant ID or exact tenant name. The
command is destructive and the server requires the all-tenants administrator
claim.

| Flag | Description |
|------|-------------|
| `--force` | Use the async force-delete operation flow. The CLI previews row counts, starts an operation with the preview confirmation phrase, then watches status. |
| `--yes` | Skip confirmation prompts for automation. The destructive preview is still printed before the operation starts. |
| `--timeout-seconds` | Maximum seconds to watch the async operation before returning timeout exit code `5`. The operation continues server-side; resume with `delete-watch`. |
| `--json` | Output JSON, including blocker details when the delete is blocked. |

Default flow:

```bash
cap system tenants delete "E2E Force Delete 20260612"
```

1. Prompts for confirmation unless `--yes` is supplied.
2. Sends `DELETE System/Tenants` with `{ "entityKeys": ["<id>"], "force": false }`.
3. On success, exits `0` and prints `Tenant deleted.` plus any warnings.
4. On `409 TENANT_DELETE_BLOCKED`, prints the blocker table and exits with `EXIT_BLOCKED` (`7`).

Force flow:

```bash
cap system tenants delete "E2E Force Delete 20260612" --force --timeout-seconds 600
```

1. Calls `POST System/Tenants/Delete/Preview` and prints the target, blocker, and row-count preview.
2. Prompts for the exact confirmation phrase unless `--yes` is supplied.
3. Calls `POST System/Tenants/Delete/Operations` with the expected total row count and confirmation phrase.
4. Polls `GET System/Tenants/Delete/Operations/{operationId}/Status` until terminal status or timeout.
5. On success, prints the operation ID and per-table deleted row counts. On timeout, exits `5` and prints a resume command.

Resume or inspect an operation:

```bash
cap system tenants delete-status <operation-id> --json
cap system tenants delete-watch <operation-id> --timeout-seconds 600
```

Exit codes:

| Scenario | Exit Code | Notes |
|----------|-----------|-------|
| Deleted or user declined a prompt | `0` | Decline prints `Aborted.` and sends no further destructive request. |
| Blocked by tenant data or residual guard | `7` (`EXIT_BLOCKED`) | JSON output includes `blockers`; text output prints a blocker table. |
| Async force delete watch timeout | `5` | Operation continues server-side. Use `delete-status` or `delete-watch <operation-id>` to resume. |
| Validation or non-interactive prompt without `--yes` | `2` | Use `--yes` for scripts and CI. |
| Auth, tenant, API, timeout, or transport errors | Existing shared CLI exit codes | See `cap schema --json` for the current exit-code contract. |

### Tenant AI configuration

```bash
cap system tenants ai-config get        [--tenant-id <id>] [--json]
cap system tenants ai-config set        [--tenant-id <id>] [--ai on|off] [--key-source nxgn|client]
                                        [--provider anthropic|deepseek] [--model <id>|default]
                                        [--api-key-stdin] [--acknowledge-data-location]
                                        [--all-languages on|off] [--json]
cap system tenants ai-config test       [--tenant-id <id>] [--provider anthropic|deepseek] [--model <id>]
                                        [--api-key-stdin] [--json]
cap system tenants ai-config remove-key [--tenant-id <id>] [--json]
cap system tenants ai-config models     [--provider anthropic|deepseek] [--json]
cap system tenants ai-config usage      [--tenant-id <id>] [--from yyyy-MM-dd] [--to yyyy-MM-dd]
                                        [--group-by feature|model|day] [--json]
```

- `--tenant-id` defaults to the current tenant. Changing settings needs System Manage; another tenant also needs Tenants Scope Unrestricted.
- `set` changes only the options given. `--model default` follows the provider's default model. `set` and `remove-key` save the tenant (`POST System/Tenants`) with only its AI settings changed, as the tenant page does.
- The client's key is never an option value: `--api-key-stdin` reads it from standard input, or a hidden prompt on a terminal. It is tested before it is stored and never printed; `get` shows only its last four characters.
- `--acknowledge-data-location` records that the tenant accepts DeepSeek's data location (People's Republic of China) and prints the notice to stderr first.
- `--all-languages on` also writes narrative AI summaries in every language a narrative has text in; `off` (the default) writes the default language only.
- `test` without a key tests the key in use; it exits 1 when the test fails. Key tests are limited to 10 a minute per person.
- `remove-key` forgets the stored client key and leaves AI as it is. It is refused while AI uses the client's key: turn AI off or switch to `--key-source nxgn` first. Revoke the key at the provider too.

`reporting dashboards get-insights` reports `ai_disabled` (409), `ai_unavailable` with a reason (409, not retryable) and `ai_busy` with a reason (503, retryable). See [Configure a Tenant's AI](../recipes/configuration/configure-ai.md).

### Bootstrap local tenant

```bash
cap system tenants bootstrap-local <name> --from-aspire [--force-new] [--json]
```

Reuses an exact tenant name when present, and leaves that tenant's configuration unchanged. When the reused tenant lacks any of the Day, Week, Month, Quarter or Year periods, the command warns and names the missing ones. A new tenant gets all five. `--force-new` appends `yyyyMMddHHmmss` and **creates a new Identity organisation** entitled to the current user. There is no teardown: leftover names such as `residual-h2 20260824122327` stay on the Identity picker until you `cap system tenants delete` them (switch off that tenant first). Prefer reuse for shared users such as testera. No in-repo test should pass `--force-new` against a shared human Identity user.

### Input value history

```bash
cap data input-values history --metric <id> --org-node <id> --data-interval month --start-date 2026-01-01 [--json]
```

Returns the capture-cell audit log, including after the value has been cleared.

### Save input values with comments

```bash
cap data input-values save --file values.json [--json]
```

Each item may carry `"comments"`. A value saved in a band marked `requireComment` without a comment is still saved; the command prints a `Comment required: ...` warning for it (also in the JSON `warnings`). Re-save the value with `"comments"` to explain it. Omitting `comments` keeps the stored comment; `""` clears it.

To clear a captured value, save its business key with `"value": null`; there is no separate delete command. A stored comment outlives the value: the first null save keeps the record with its comment, and a second one (or `"comments": ""` with the first) deletes it. A `"value": null` item with `"comments"` saves a comment without a value. See [Clear a Value](./model-building.md#clear-a-value).
An input excluded at an org node cannot be captured there (rule `OrgNodeExcluded`). If any item targets such an (input, org node) pair, the save fails with 400 and nothing in the request is saved; the error reads `InputValue for metric <id> at <start>: The input is excluded at this org node and cannot be captured there.` This applies to clearing too: an item with an empty `value` at an excluded org node is rejected. The exclusion names that org node only, so its child org nodes still capture. `data input-values upload-excel` reports a row with a number at an excluded org node as a row error (`'<input>' is excluded at '<org node>' and cannot be captured there.`), and an upload with any row error saves nothing. Blank cells at an excluded org node are ignored.

### Copy attachments between tenants

```bash
cap data input-values attachments download --template <id> --data-interval <day|week|month|quarter|year> \
    --periods "<names>" [--org-nodes <id>[,<id>]] --output <dir> [--overwrite] [--json]
cap data input-values attachments upload <dir> [--template <id>] [--dry-run] [--timeout-seconds 600] [--json]
```

Copies the evidence files on input and narrative values from one tenant to another, in the same environment or another one, through a local folder.

`download` reads the capture template's grid for one data interval and writes every file to `<dir>/files/<attachment id>/<file name>`, with a `manifest.json` that keys each file by cell type, input or narrative name (the name, not the friendly name), org node path and period start, under the manifest's one data interval. A folder that is not empty is refused unless `--overwrite` is given. Only cells the grid shows are covered: run it once per data interval, and note that files on values at excluded org nodes, on cells the template does not apply to, and on cells outside your view scope are not downloaded. A file you may not view (403) or that has gone (404) is reported and skipped.

`upload` acts on the active tenant. It finds the capture template with the manifest's template name (or `--template`), the periods by start date, and each value by cell type, name, org node path, interval and period start. It reads the template's grid for the org nodes named in the manifest, so files reach every site, not only the first. A template without an org node level shows one site at a time: its download holds that one site, and its upload matches only the first site of a manifest that names several. Each file is added to its value without changing the value: its number, unit, comment, status, approvals and other files stay as they are, and no recalculation or capture notification is triggered. The value's history gets "Attachment added: <file name>" under you. Every file ends with one outcome:

| Outcome | Meaning |
|---------|---------|
| `added` | the file was added (`would-add` with `--dry-run`) |
| `already-present` | the value already has a file with the same name and size; re-running a copy adds nothing twice |
| `no-target-value` | no value with that key exists in the target (the values are copied first, with `download-excel` / `upload-excel`), or the name or org node path is shared by several items in the target (message `ambiguous: N … named '…' in this tenant`); no file is added to one of them by guess |
| `locked` | the value or its period is locked: found from the grid before sending (also with `--dry-run`), or answered 409 |
| `forbidden` | you may not capture that value (403) |
| `invalid` | empty file, more than 50 MiB, a disallowed extension, a name over 250 characters, or more than 10 files on the value |
| `failed` | any other error; the API message is kept |

Exit code `0` when every file is `added` or `already-present`, `6` when some are not. `--dry-run` checks and matches everything and sends nothing. `--timeout-seconds` applies to the file uploads only. Files are added through `POST DataManagement/InputValues/Attachments` and `POST DataManagement/NarrativeValues/Attachments` (multipart: the value id and one file), which need Data Capture Manage and capture rights on the value.

`data input-values list` and `data narrative-values list` show each cell's file count in an `Attachments` column (`attachmentCount` with `--json`). `reporting computed-values list --json` carries `attachmentCount` and `latestAttachmentDate` (UTC) on each value cell and narrative cell with files; a longer period sums the files of the captured values it contains, and cells the viewer may not see carry none. `data input-values get --json` leaves out attachment fields that are null; a file's content is never in the response.

### Validate Data

```bash
cap data input-values validate <id> --result <approve|reject> [--comments "..."] [--json]
cap data input-values validate --ids <id>[,<id>] --result <approve|reject> [--comments "..."] [--json]
cap data change-requests validate <id> --result <approve|reject> [--comments "..."] [--json]
cap data narrative-values validate <id> --result <approve|reject> [--comments "..."] [--json]
```

| Result | Description |
|--------|-------------|
| approve | Approve the value (moves to next validation level or complete) |
| reject | Reject the value. A reject needs `--comments` explaining the reason; the CLI and the API refuse it without one, for `input-values`, `narrative-values` and `change-requests` alike |

An approval is recorded even when the value stays `Validation In Progress`, for example on a Consensus level that still needs another approver. Each item in `results` then carries `awaitingLevelName`, the level the value still waits for, and `repeatedOwnDecision`, which is `true` when the approval only repeated your own earlier one and so counted nothing new. The single-value text output says the same: `Input value <id> validation status: Validation In Progress. Waiting for the Manager level.` The `--ids` table adds `Awaiting Level` and `Repeat` columns.

`change-requests validate` decides each pending value of the request on its own: the values you can decide now at your level are approved or rejected, and the rest are skipped. A reject needs `--comments`; the CLI refuses it without one. The output summarises what the decision did, for example `Change request <id>: approved 4, moved on 0, rejected 0, skipped 2 (clash 2)`. When you approve at a Consensus level that still needs other approvers, the value stays at that level and the summary adds `recorded <n> (waiting on more approvals)`; its `outcome` is `Recorded`. A value is skipped as `clash` when another pending change request targets the same value, as `no permission` when you can't approve its discipline, and as `already decided` when you already decided it at every level you hold. With `--json`, `decisions` carries each value's `outcome`, `valueStatus` and `awaitingLevelName`, plus the `summary`. When no value could be decided, the command fails with the reason. A request whose values were decided both ways ends as `Partly Approved`.

`change-requests get` lists each value after the request's fields: item, period, value, `Status`, `Waiting On` (the level a pending value waits on) and `Last Decision` (outcome, level, decider, date and comment). With `--json`, each input and narrative value carries `validationStatus` (the status number), `awaitingLevelName` and `decisions`. `change-requests list --status` accepts `draft`, `validation-required`, `validation-in-progress`, `approved`, `rejected` and `partly-approved`.

`change-requests list --search <text>` finds requests by CR number (`157` or `CR-0157`), or by text matched, case-insensitively and in part, against input and narrative names and friendly names and the name of the person who raised the request. All-digit text is always a CR number, and a number search replaces `--number`. `--data-interval` is needed only with `--periods`, to match the period names; it is not an interval filter, and without `--periods` the list covers every period and interval. Besides number, status, reason, org node, interval and dates, each row shows `Items` (the inputs and narratives in the request), `Requested By`, `Raised`, `Waiting On` (each level that holds pending values, with how many, for example `Level 1 (2), Level 2 (1)`), `Clashes` (the numbers of other pending requests on the same value; `hidden` means a clashing request outside your access) and `Last Rejection` (level, decider, date and comment, with the item first when the request holds several values). With `--json`, each row also carries `requestedByName`, `raisedAt`, `itemNames`, `valueCount`, `levels`, `clashWith`, `hiddenClash` and `lastRejection`; a field with no value (`note`, `single`, `lastRejection`) is left out, while empty lists such as `levels: []` stay.

### Bulk Validation

Use `preview` and `validate` to approve or reject every input and narrative value in a
validation template's scope in one action, with one shared comment. Run
`preview` first. It counts what would be decided and what would be skipped,
without writing anything.

```bash
cap data bulk-validation preview \
  --template <spreadsheet-template-id> \
  --data-interval month \
  --periods "Jan 2026,Jun 2026" \
  --org-nodes <org-node-id>[,<org-node-id>] \
  --discipline-nodes <discipline-id> \
  --json

cap data bulk-validation validate \
  --template <spreadsheet-template-id> \
  --data-interval month \
  --periods "Jan 2026,Jun 2026" \
  --org-nodes <org-node-id> \
  --result approve \
  --comments "H1 sign-off" \
  --json
```

- **Scope:** the template bounds the scope.
  - Org node and discipline IDs are used exactly as given. Child nodes are not added.
  - If you leave one of the two flags out, every org node (or discipline) in the template is included.
  - You must pass at least one of `--org-nodes` and `--discipline-nodes`.
- **Interval:** `--data-interval` is an exact filter. A monthly scope never touches quarterly or annual values.
- **Overwrites:** any earlier decision you made on a value in scope is replaced.
- **Rejects:** `--result reject` needs `--comments` with the reason. The capturer sees it, and the CLI and the API refuse a reject without one.
- **Skipped values:** values you can't validate, and values that don't require validation, are skipped and counted.
- **Locked values:** these are still validated.
- **Files:** `preview` counts the values to be decided that carry supporting files (`withAttachments` in `--json`, the `With files` column in the table). A bulk decision opens none of them, so review those values first.
- **Limit:** one run decides at most 20,000 values. `preview` reports when a scope is over the limit.
- **Results:** `validate` prints the four counts per kind. It does not send cell ids, and `--json` includes an empty `results` list on each kind.

### Narrative Workflow

`cap model narratives get <id> --json` includes the narrative's data feeds
(`narrativeDataFeeds`) and the narrative its AI summary reads
(`summarisedNarrative`). Imported narrative text shows "Data feed" as the author
in `history`, and text an AI summary wrote shows "AI" (`changedByAi`).

```bash
cap model narratives lookup --capture-interval quarter --org-nodes <id> --discipline-nodes <id> --json
cap data narrative-values save --file narrative-value.json --json
cap data narrative-values submit <value-id> --json
cap data narrative-values validate <value-id> --result approve --comments "Approved" --json
cap data narrative-values get-by-context --narrative <id> --org-node <id> --period-type quarter --start-date 2026-01-01 --json
cap data narrative-values history <value-id> --json
cap data narrative-values use-ai-version <value-id> --json
```

On a summary narrative, `get` shows `isAiGenerated` (the AI wrote the text and
nobody has approved it yet) and `aiNewVersion` (the AI's newer text by language,
kept because a person changed the value). `use-ai-version` replaces the text
with that version in every language, as your capture save; it is refused when
there is no new version or the period is locked. `list` shows both in its `AI`
column.

For locked periods, apply the data lock first, then use the unified change
request workflow. Change requests can carry narrative edits in the same
contract surface as input edits:

```bash
cap data data-lock lock --data-interval quarter --periods "Q1 FY 25" --org-nodes <id> --discipline-nodes <id> --description "Close narrative capture" --json
cap data change-requests create --file change-request.json --json
cap data change-requests validate <request-id> --result approve --comments "Approved" --json
cap data data-lock unlock --data-interval quarter --periods "Q1 FY 25" --org-nodes <id> --discipline-nodes <id> --description "Reopen narrative capture" --json
```

Narrative names must be unique per tenant per language (trimmed,
case-insensitive, including auto-created translation stubs) and cannot contain
`{` or `}`, because widget text references narratives as `{Narrative Name}`.
A database unique index enforces the name rule, so concurrent saves of the
same name also fail cleanly. `model narratives create`/`save` return 400 with
field errors for a name another narrative has (`NameNotUnique`), a missing
field, braces, a discipline, framework node, attribute value or site the
workspace doesn't have, a data feed rule or an unusable metric link (`name`,
`discipline`, `frameworkNodes`, `attributeValues`, `captureInterval`,
`orgRollUpMethod`, `finerPeriodMethod`, `coarserPeriodMethod`,
`orgNodeExclusions`, `siteOverrides`, `narrativeDataFeeds`, `metricIds`). A 409
means another save got there first: `NarrativeNameConflict` (it took the name)
or `NarrativeConcurrentSave` (it changed the same narrative; save again).
`model narratives upload-excel` fails with the named conflict.
A save with an id edits that narrative, or creates it when no narrative has
the id, so a new narrative named like another is refused; only a request
without an id (`create`, Excel rows) updates the narrative of that name.
Saving a narrative with a different capture interval deletes its values at
every other interval, locked and approved values included.

A `save` file must carry `orgRollUpMethod` (Across Sites), `finerPeriodMethod`
(Finer Periods) and `coarserPeriodMethod` (Coarser Periods); `create` fills the
ones left out with None, Inherit and Latest. `list` and `get` show them, and
`distinct-values` takes `org-roll-up-method`, `finer-period-method` and
`coarser-period-method` next to `reference`, `capture-interval`,
`require-data-capture` and `require-validation`. See
[Create Narrative Definition](./model-building.md#across-sites-finer-periods-and-coarser-periods).

```bash
cap model narratives lint (--file narrative.json | --id <id> | --all [--details]) [--json]
```

`lint` checks a narrative without saving it, with the codes the web editor
uses: errors are exactly what `save` would reject (exit code 2). The file is
the same payload as `save` (or the output of `get --json`); `--id` lints a saved
narrative as it is stored. Warnings never block a save:
`CaptureIntervalChangeDeletesValues` says how many values (locked and approved
counted) a capture interval change deletes. `create` and `save` lint first and
print the same warnings. `--all` lints every narrative as it is stored; see
[Lint every metric](#lint-every-metric---all) for the summary.

Narrative JSON input uses the Capstone narrative request
contracts. Use `--file <path>` or pipe JSON on stdin; use `--json` so automation
receives full IDs and contract-shaped response envelopes.

**Narrative exclusions.** `model narrative-org-node-exclusions` hides a
narrative at one org node. It never deletes values: they stay stored and
return if the exclusion is removed. An exclusion applies to its own node only,
not to child sites.

| Command | Options |
|---------|---------|
| `list` | `--org-nodes` (exactly one, required), `--discipline-nodes`, `--grouping none\|discipline`, `--limit`, `--offset`, `--json`. Columns: ID, Name, Discipline, Included, Excluded |
| `save` | `--file` or stdin: `{"narrativeOrgNodeExclusions":[{"narrativeId","orgNodeId","isSelected"}]}`. `isSelected: true` means the narrative applies at the node (no exclusion) |
| `copy <source> <destinations>` | Gives each destination exactly the source node's exclusions: exclusions the source lacks are removed there |
| `download-excel` | `--output` |
| `upload-excel <file>` | Prints rejected rows |

### Data Locks

```bash
cap data data-lock lock \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --org-nodes <org-node-id> \
  --discipline-nodes <discipline-id> \
  --description "Close Q1 narrative capture" \
  --json

cap data data-lock unlock \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --org-nodes <org-node-id> \
  --discipline-nodes <discipline-id> \
  --description "Reopen Q1 narrative capture" \
  --json
```

Use `--periods "<start>,<end>"` for a continuous range, or use
`--start-period` and `--end-period`. At least one `--org-nodes` or
`--discipline-nodes` value is required; narrative lock tests must pass both.

### Notification Templates

```bash
cap notifications templates list [--limit N] [--offset N] [--json]
cap notifications templates get <id> [--json]
cap notifications templates tokens [-t|--type <type>] [-e|--event <event>] [--json]
cap notifications templates lint (-f|--file template.json | -i|--id <guid>) [--json]
```

Every template is **Written for** one thing: one event, or its notification
type's digest, which scheduled rules send. The values are:

| Notification type | Written for |
|-------------------|-------------|
| `DataCapture` | Pending capture digest, Input Value Captured, Data Lock Applied |
| `Validation` | Pending validation digest, Validation Completed |
| `ChangeRequest` | Pending change requests digest, Change Request Validated, Change Request Awaiting Decision |
| `DataFeedChanges` | Data feed changes digest |
| `AiProvider` | AI Key Failing, AI Key Recovered (no digest) |

In JSON a digest has a null `eventType`; an event template names its event,
for example `"eventType": {"id": 3, "name": "Input Value Captured"}`.

`list` shows the columns ID, Name, Type, **Written for**, **Used by** (`Not
used`, `1 rule` or `N rules`) and **Available to rules** (`yes`/`no`). Each
`--json` row carries `writtenFor` and `ruleCount`. `get` shows the same
fields, with the names of the rules under Used by. `get --json` carries
`rules` (`[{id, name}]`); `save` ignores it, so `get --json` output still
saves as is. `isActive` is **Available to rules**: it only decides whether
the rule picker offers the template, and rules that already use it keep
sending. Templates have no default any more: `isDefault` is gone from `get`
and `list`.

While any rule uses a template, what it is written for (its
`notificationType` and `eventType`) cannot change. `save` refuses it with the
code `ContentInUse` and names the rules. Point the rules at another template
first, or create a new template.

#### List the tokens

`tokens` lists each Written for value with a one-line hint and the tokens
every send of it fills: `{{Token}}`, a description, and `yes` under Body only
for a token that holds HTML (the digest tables and `{{ActionLink}}`), which
the subject cannot use. Filter with `-t`/`--type` (`DataCapture`,
`Validation`, `ChangeRequest`, `DataFeedChanges`, `AiProvider`) and
`-e`/`--event` (`InputValueCaptured`, `DataLockApplied`, `ValidationCompleted`,
`ChangeRequestValidated`, `ChangeRequestAwaitingDecision`, `AiKeyFailing`,
`AiKeyRecovered`). A type alone
returns its digest and its events; there is no digest-only filter. `--json`
prints the API result: `{ "contents": [{ notificationType, eventType,
writtenFor, hint, tokens: [{ key, group, description, sample, bodyOnly }] }] }`.
`group` is `RecipientAndWorkspace`, `ThisDigest`, `ThisEvent` or
`TablesAndLinks`. It needs `MasterData.View`.

Only tokens filled on every send are listed. A few more are accepted but not
listed, because they can be empty or are kept for older templates:
`{{ValidationResult}}`, `{{InputValueId}}` and `{{InputValueIds}}` on
Validation Completed, `{{ValidationResult}}` on Change Request Validated,
and `{{ActionUrl}}` and `{{ActionLink}}` on the Data feed
changes digest (empty for a recipient with no capture template that shows the
changes). A template that uses them saves and sends. `{{ScopeSummary}}` and
`{{NarrativeValueCount}}` on Validation Completed are filled only for a bulk
decision or a capturer's rejected values, so a single approval would fail;
`save` refuses them.

#### Lint a notification template

`lint` checks a template without saving it. Give exactly one of `--file`
(the same payload as `save`, or the output of `get --json`) or `--id` (a
saved template, read and checked as stored). **Errors** are exactly what
`save` would refuse; **warnings** never block a save. Each issue has a
`code`, a `message`, a `field` (`name`, `subject`, `body`, `eventType`,
`notificationType` or `translations`) and a `stage`, the editor section that
shows that field: `definition` for `name`, `description`, `notificationType`,
`eventType` and `isActive`, and `message` for `subject`, `body` and
`translations`. A token issue carries its position in `params`. Text output prints one line per issue as
`Error   [Code] field: message`, then a count line. The exit code is `2` when
there is at least one error and `0` otherwise. `--json` prints the API result
unchanged: `{ "errors": [...], "warnings": [...], "notes": [...] }`. It needs
`MasterData.Manage`.

| Code | Meaning |
|------|---------|
| `UnknownToken` | `{{X}}` is not a token; the message suggests the closest one when it can |
| `TokenNotFilled` | The token is real, but this Written for value does not fill it |
| `TokenNotAlwaysFilled` | `{{ScopeSummary}}` or `{{NarrativeValueCount}}` on Validation Completed |
| `HtmlTokenInSubject` | A body-only token in the subject |
| `DigestTableOnEvent` | A digest table in an event template |
| `WrongDigestTable` | Another digest's table |
| `ContentNotForType` | The event is not one this notification type offers |
| `AiNeedsEvent` | An AI Provider template without an event |
| `ContentInUse` | `notificationType` or `eventType` changed while rules use the template |
| `Required`, `TooLong` | Name, subject or body empty, or longer than 250, 500 or 8000 characters |
| `DuplicateName` | Another template has the name |

A token issue is an error only when the draft changes that text or what the
template is written for. Text that is the same as stored gets a warning, so a
mistake stored in another language never blocks a fix. So `lint --id`
reports stored token problems as warnings, and exits `0` unless a field is
missing or too long, or the name is taken.

`create` and `save` do not lint first; the server runs the same checks on
save. A refused save answers HTTP 400 with the field-keyed body
`{ "errors": { "eventType": ["..."], "body": ["..."] } }`, and the CLI lists
each field's messages under `errors`. Up to version 0.4.121 a refused template
save answered `{ "error": "..." }` instead. The Excel upload runs the same
checks per row: errors leave the row out, warnings are reported.

The web editor's live preview (`POST Notifications/Templates/Preview`)
renders a draft with sample values. The CLI has no preview command.

### Notification Rules

```bash
cap notifications rules lint (-f|--file rule.json | -i|--id <guid>) [--json]
```

`lint` checks a rule without saving it, as the web rule editor does. Give
exactly one of `--file` (the same payload as `save`, or the output of
`get --json`) or `--id` (a saved rule, read and checked as stored).
**Errors** are exactly what `save` and `upload-excel` refuse; **warnings**
(`RuleInactive`, `TemplateUnavailable`) never block a save. Each issue has a
`code`, a `message`, a `field` and the editor section it belongs to as
`stage` (`definition`, `when`, `message` or `recipients`). Text output
prints one line per issue as `Error   [Code] field: message`, then a count
line. The exit code is `2` when there is at least one error and `0`
otherwise. `--json` prints the API result unchanged, with the next five
sends under `counts.nextRuns` (`[{ fireAtUtc, periodName }]`; `periodName` is null except
for period-end rules). The next sends are empty for an event rule that sends each time, and
while the When section has an error; a summary rule lists its next sends. It needs `MasterData.View`.

| Code | Meaning |
|------|---------|
| `Required`, `TooLong`, `DuplicateName` | A name, trigger type, template, channel or Send (`priority`) is missing, a name is longer than 250 characters, or another rule has the name |
| `TriggerBothRetired` | Trigger type Both is retired; use Scheduled |
| `TemplateNotFound` | The template does not exist |
| `TemplateNeedsEvent`, `TemplateNeedsSummary`, `TemplateNeedsDigest` | An Event Driven rule that sends each time needs an event template, one that sends a summary needs that event's summary template; a Scheduled rule needs a digest |
| `SummaryNotOffered` | Send = Summary on an AI key event, which always sends each time (reported on `priority`) |
| `SummaryGap` | A summary must send at least every 120 days, the time events are kept |
| `DataFeedChangesNeedsSchedule` | A Data Feed Changes rule must be Scheduled |
| `AiEventScoped` | An AI key rule cannot have an org node or discipline (reported on `orgNode`, or on `disciplineNode` when only a discipline is set) |
| `ScheduleRequired`, `ScheduleCronAndOffsets` | A scheduled rule or a summary needs a `cronExpression` or period-end offsets, not both |
| `CronInvalid`, `CronNeverRuns` | The CRON expression does not parse, or never runs again |
| `OffsetsNeedSchedule`, `OffsetOutOfRange`, `OffsetRepeated` | Offsets only on a scheduled rule or a summary, between -31 and 31 days, no repeats |
| `PeriodTypeRequired`, `FireTimeRequired`, `FireTimeNeedsOffsets` | Offsets need Month, Quarter or Year and a `fireTime` (`HH:mm`); a `fireTime` without offsets is refused |
| `TimeZoneRequired`, `TimeZoneInvalid` | A scheduled rule or a summary needs a valid `timeZoneId` |
| `DataFeedChangesGap` | A Data Feed Changes rule must send at least every 120 days |

#### A rule's runs

```bash
cap notifications rules runs <rule-id> [--limit N] [--offset N] [--json]
```

Lists each time the rule was evaluated, newest first, as the web rule editor's
run history shows it: `evaluatedAt` (UTC), `evaluationSource` (Scheduled or
Event Driven), `result` (Dispatched, Skipped or Error), `skipReason`,
`evaluationKey` (the period or event the run was about), `recipientCount` and
`dispatchedCount`. For a scheduled reminder, `recipientCount` is the people
with pending work they can act on (when nobody has any, the run is Skipped
with `EMPTY_TABLE_RENDERINGS` and the count is everyone in scope: for Data
Capture everyone the rule reaches, for Validation and Change Request only the
people at the validation levels open at the time), and
`dispatchedCount` is the emails queued, leaving out anyone this rule already
emailed for the same send. A run keeps counts only: it does not record who
was left out or why. To see who a run emailed, read the delivery log for the
rule around the run's time. `--json`
returns `{ tenant, ruleId, data: [...], pagination }`. It needs
`MasterData.View`.

#### Who a rule reaches

```bash
cap notifications rules recipients <rule-id> [--json]
```

Reads the saved rule, then lists everyone its template's notification type,
org node and discipline reach, as the rule editor's recipient preview does.
Each person has `userId`, `name`, `email`, `timeZoneId` and `access`, which
says why they are included: an "Authorize all" scope such as `All data`,
`All capture` or `All validation` (with ` (role)` when it comes through a
role), or `Granted: <org node> · <discipline> · <level>` for the data
permission that covers the rule's org node and discipline. An AI Provider rule
is about the tenant itself, so it reaches the tenant's administrators (System
Manage) instead. This is who **can** receive the rule, not who a run emails:

- A scheduled Data Capture or Validation reminder mails only the people among
  them who have pending work they can act on.
- A Validation reminder also looks only at the validation levels that are open
  at the time, so the list can be longer than any one run's.
- An inactive rule mails nobody. The text output then says
  `Rule: <name> (inactive: it sends nothing)`, and the JSON has `isActive`.

With `MasterData.View` you get only the count. Names and emails need
`MasterData.Manage`; without it the text output prints
`The rule reaches N people. Names and emails need MasterData.Manage.` and the
JSON has `count` with an empty `recipients`. `--json` returns
`{ tenant, ruleId, ruleName, isActive, count, recipients: [...] }`.

### Notification Delivery Log

```bash
cap notifications delivery-log list [-r|--rule <guid>] [--since yyyy-MM-dd] [--until yyyy-MM-dd]
    [--status <status>] [--type <type>] [--recipient <text>] [--limit N] [--offset N] [--json]
```

Lists every notification the tenant has sent or tried to send, newest first, as
the web Delivery Log shows it: `createdAt` and `sentAt` (UTC), `ruleId` and
`ruleName` (null for a deleted rule), `recipientName`, `recipientEmail`,
`subject`, `status` (Pending, Processing, Sent or Failed), `notificationType`
(for example `Data Capture`), `channel` and `errorMessage`.

- `--rule` is filtered by the server.
- `--since` and `--until` are UTC dates, both inclusive.
- `--status` and `--type` match the whole value, ignoring case. `--status`
  must be Pending, Processing, Sent or Failed, and `--since` cannot be after
  `--until`; either mistake exits with code `2`.
- `--recipient` matches part of the name or email.

Those filters run in the CLI over every page of the log (stopping at the first
page older than `--since`), and `--limit`, `--offset` and `pagination.total`
then count the matches. Give `--since` or `--rule` to keep a large log quick.

It needs `MasterData.View`. Recipient names, emails and error messages need
`MasterData.Manage`; without it they are empty, the text output shows
`(hidden)` and `--recipient` matches nothing. The text output then ends with
`Recipients are hidden: names, emails and errors need MasterData.Manage.`, also
when no row matched. `--json` returns
`{ tenant, data: [...], pagination }`.

To check a scheduled rule's send without the browser:

```bash
cap notifications rules get <rule-id> --json          # lastFiredAt, lastEvaluationResult
cap notifications rules runs <rule-id> --limit 5      # people with pending work vs emails queued
cap notifications delivery-log list --rule <rule-id> --since 2026-11-14 --json   # who it emailed
cap notifications rules recipients <rule-id>          # who it could have emailed
cap security users get someone@example.com            # one person's grants and "Authorize all" scopes
```

### Users

```bash
cap security users get <user-id-or-email> [--json]
```

Shows one user in the current tenant as the web user editor does: roles, the
capture, report, dashboard and org node templates selected for them, the
number of their own data permissions in this tenant, and the nine **Authorize all** scopes. Each scope
has its `code` (for example `nxgn.capstone.templates.capture.unrestricted`),
whether the user holds it (`isSelected`), whether that comes through a role
(`isInheritedFromRole`) and whether the user also holds it as their own grant
(`isDirect`, kept when the role is removed). An email is looked up with one call to the users list,
ignoring case; like the web users list, that call needs Identity to be
reachable. `--json` returns `{ tenant, user }`, with the full user including
`dataPermissions`. It needs `Security.View`.

---

## Excel Commands

Commands for bulk import/export via Excel files.

### Download to Excel

```bash
cap <domain> <entity> download-excel -o file.xlsx
```

> **Note:** `-o` is the short flag for `--output`. If omitted, downloads to `{Entity}.xlsx` in current directory.

**Entities supporting Excel download:**
- `model inputs`, `model calculations`
- `model metric-attribute-types`, `model metric-framework-nodes`
- `model metric-org-node-exclusions`, `model narrative-org-node-exclusions`
- `masterdata units`, `masterdata data-sources`
- `masterdata disciplines`, `masterdata frameworks`, `masterdata org-nodes`
- `masterdata discipline-attribute-types`, `masterdata framework-attribute-types`, `masterdata org-node-attribute-types`
- `templates capture-templates`, `templates report-templates`
- `templates widget-templates`, `templates dashboard-templates`, `templates org-node-templates`
- `data input-values` (requires `--template <id>`)
- `reporting computed-values` (requires `--template`, `--data-interval`, `--periods`)

### Upload from Excel

```bash
cap <domain> <entity> upload-excel <file.xlsx> [--json]
cap <domain> <entity> upload-excel -f <file.xlsx> [--json]
```

Most upload commands take the file as the first argument. These take it with `-f` (`--file`) instead: `templates capture-templates`, `templates report-templates`, `templates widget-templates`, `templates dashboard-templates`, `templates org-node-templates`, `notifications templates`, `notifications rules`, `security users` and `data input-values`.

A workbook can be up to 100 MB. A larger file is refused before it is sent, with `File exceeds the maximum upload size (100 MB): <file>` and exit code `2`.

Seven upload commands use the async upload pipeline: `data input-values`, `model inputs`, `model calculations`, `model narratives`, `masterdata org-nodes`, `masterdata disciplines`, and `masterdata frameworks`. The command sends the file, receives an upload id, and polls the upload's status until `Succeeded` or `Failed`.

```bash
cap masterdata org-nodes upload-excel "artifacts/performance/excel/Org Structure.xlsx" \
  --timeout-seconds 3000 \
  --json
```

The default timeout is `3000` seconds (50 minutes) for the tree-node uploads and `600` seconds (10 minutes) for the others. Use `--timeout-seconds <n>` for larger files. In JSON mode the final status payload includes `uploadId`, `status`, `fileName`, `fileSizeBytes`, `totalRows`, `processedRows`, `createdDate`, `updatedDate` (last activity, when the server reports it), `completedDate`, and `errors`. Each error has `rowIndex`, `columnIndex`, `errorMessage`, `severity` (`Warning` for a warning) and, when a coded rule raised it, the lint `code` (for example `UnitOfMeasureUnknown` or `FormulaUnknownMetric`); the text table shows it in a `Code` column. `rowIndex` and `columnIndex` are 0-based; the text table shows the Excel row number and column letter instead (cell D5 is row `5`, column `D` in the table, `rowIndex` 4 and `columnIndex` 3 in JSON).

**Timeout and resume.** If the upload does not finish within the timeout, the command exits `5` and prints the upload id. The upload keeps running on the server. Check it later with the same entity's `upload-status` command, which the timeout message names:

```bash
cap model inputs upload-status <upload-id> [--json]
```

`upload-status` reads the status once and does not poll. In JSON mode it prints the same status payload as `upload-excel`.

| Upload status | Exit code |
|---|---|
| `Succeeded` with no errors (warnings allowed) | `0` |
| `Succeeded` with row errors | `6` |
| `Failed` | `4` |
| `Pending` or `Processing` (text output shows rows processed, how long ago the upload was created, and how long ago a `Processing` upload last made progress, or `not yet picked up` for a `Pending` upload) | `0` |
| Unknown upload id, or a finished upload that has been purged (by default 30 days after it completed; uploads still `Pending` or `Processing` are never purged) | `2` |

An upload that stays `Pending` because its processing message was lost is re-sent by the server about every 5 minutes, so it does not need re-uploading; a `Processing` upload that shows no progress for 30 minutes is re-sent the same way, until 2 hours after processing started, when it is marked `Failed` so it can be re-uploaded (or split into smaller files). Use the same entity that ran the upload: an input upload id is not found under `model calculations upload-status`. The three tree-node entities share one upload store, so any of them can read a tree-node upload id.

Non-async Excel upload commands return `ExcelUploadResult` in JSON mode:
`items`, row counts such as `savedRows`/`newRows`/`updatedRows`, `isValid`, and
`uploadErrors` with 0-based `rowIndex` and `columnIndex` (the text table shows
the Excel row number and column letter), and `errorMessage`. Keep these
row/cell diagnostics for spreadsheet workflows; for a single item kept as JSON,
use `create --file` or `save --file`.

**Entities supporting Excel upload:**
- `model inputs`, `model calculations`
- `model metric-attribute-types`, `model metric-framework-nodes`
- `model metric-org-node-exclusions`, `model narrative-org-node-exclusions`
- `masterdata units`, `masterdata data-sources`
- `masterdata disciplines`, `masterdata frameworks`, `masterdata org-nodes`
- `masterdata discipline-attribute-types`, `masterdata framework-attribute-types`, `masterdata org-node-attribute-types`
- `templates capture-templates`, `templates report-templates`
- `templates widget-templates`, `templates dashboard-templates`, `templates org-node-templates`
- `data input-values`

`model metric-org-node-exclusions` workbooks contain only `Org Node Path` and
`Metric` columns. Each data row means that the named metric is excluded from the
resolved org-node path. Upload is upsert-only: rows present in the workbook
create or keep exclusions, and rows omitted from the workbook do not delete or
include existing exclusions.

`model narrative-org-node-exclusions` workbooks have one sheet, "Narrative Org
Node Exclusions", with only `Org Node Path` and `Narrative` columns. Each row
excludes the named narrative at that one org node. Upload is upsert-only, the
same as metrics. Unknown or ambiguous paths, GUID-like values and unknown
narratives are rejected per row; valid rows still save.

---

## Narrative Roll-up Status (HB-652)

Rolled-up narrative text (parent sites, longer periods, summary narratives) is worked out in the background after a
narrative changes. These commands read whether it has caught up, the way `data recalculation` does for calculations.
They never wait on calculations, and `data recalculation wait` never waits on narratives.

```bash
cap data narrative-aggregation status --json        # narrativeVersion, caughtUpTo, waiting, failed, stuck, settled
cap data narrative-aggregation wait --json          # wait for the current narrative version (default target)
cap data narrative-aggregation wait 124 --timeout-seconds 120 --stall-seconds 60 --json
cap data narrative-aggregation log --failed --json  # failed roll-ups, newest attempt first
cap data narrative-aggregation log --narrative "Water" --limit 20
```

- `wait` is settled when `caughtUpTo >= target`. A summary narrative fed by a change carries the same number, so the
  wait ends only once the summaries are stored too.
- `wait` exits 5 (Timeout) on its timeout, or as `stalled` when `caughtUpTo` does not move for `--stall-seconds`. A
  stalled wait names the next step: open System > Narrative Aggregation Log and press Retry waiting.
- `log` pages, filters and sorts on the server; `--failed` and `--narrative` are server filters. A failure that repeats
  is one row with its attempts.
- Status and log need System view; Retry waiting (web only) needs System manage.

---

## People to Follow Up (HB-675)

Who is late with data capture or validation across the workspace, most late values first: the list the home page shows
administrators. Late means a value not captured or not validated for a month that ended within the last 12 months,
counted as the progress reports count (a locked late value still counts; a value counts for each person who must act).

```bash
cap reporting completion follow-ups                 # top 10: person, to capture, to validate per level, rejected, sites, late since
cap reporting completion follow-ups --limit 50
cap reporting completion follow-ups --json          # everyone: available, windowStart, windowEnd, people[]
```

- "Nobody assigned" is late work no capturer or validator can act on (no grant at that site).
- Needs Data Validation manage and access to every site and discipline; anyone else gets `available: false`.

---

## Status & Configuration Commands

```bash
cap status                          # Show current context (auth, tenant, config)
cap version --json                  # Show CLI version, runtime, install path, API endpoint, tenant
cap update check --json             # Check latest CLI release; does not install updates
cap config get [key]                # View one or all configuration settings
cap config set <key> <value>        # Set configuration value
cap config unset <key>              # Reset setting to default
```

---

## Authentication Commands

```bash
cap auth login                      # Identity OIDC login (PKCE, opens browser)
cap auth login --no-browser         # Print the authorize URL; keep the CLI running until callback
cap auth login --device             # Device-code flow (SSH / no loopback)
cap auth login --with-api-key       # API key auth (reads CAPSTONE_API_KEY env var)
cap auth login --with-api-key --persist  # Persist the key locally after validation
cap auth logout                     # Revoke Identity refresh token and clear local credentials
cap auth whoami                     # Show current user/tenant
cap auth tenants                    # List accessible tenants
cap auth switch-tenant <id>         # Change active tenant
```

**API Key Authentication (CI/automation):**
```bash
export CAPSTONE_API_KEY=your-api-key
cap auth login --with-api-key
```

Optional env vars: `CAPSTONE_IDENTITY_URL` (login-only issuer override), `CAPSTONE_CALLBACK_PORT` (pin PKCE loopback), `CAPSTONE_PKCE_TIMEOUT_SECONDS` (loopback wait, 30–1800, default 900), `CAPSTONE_CONFIG_DIR`. SSH sessions should use `--device`, not `--no-browser`. `--no-browser` keeps a loopback listener for 15 minutes by default — complete Microsoft SSO (SSO-only users: do not use the Identity password form) before it expires. If the listener dies, Chrome shows connection refused; re-run login instead of reusing the URL. Identity's own login session is separate from this wait.

Tenant switch or first-login that saves a tenant but Identity requires a new login (`RequiresReauth`) exits `6` (`PartialResult`). Auth-required failures exit `3`.

---

## Command Status Summary

The schema-derived command surface above is the exact implemented command list.
This table is a high-level availability summary only.

| Domain | Commands | Status |
|--------|----------|--------|
| `root` | schema, concepts, status, version | ✅ Available |
| `workflows` | list, show | ✅ Available |
| `update` | check | ✅ Available |
| `config` | set, get, list, show, unset | ✅ Available |
| `auth` | Login, logout, tenant selection, current-user context, API key management, available language list | ✅ Available |
| `meta` | cross-domain lookups | ✅ Available |
| `model` | metrics, inputs, calculations, narratives, model tree aliases, lookups | ✅ Available |
| `model` | metric/narrative attribute types, framework nodes, org-node exclusions, input/calculation/narrative overrides, formula validation | ✅ Available |
| `masterdata` | org-nodes, disciplines, frameworks, units, data-sources | ✅ Available |
| `masterdata` | Excel commands for above entities | ✅ Available |
| `masterdata` | discipline-attribute-types, framework-attribute-types, org-node-attribute-types | ✅ Available |
| `masterdata` | data-exports | ⏳ Planned |
| `templates` | All CRUD + Excel, schema/sample helpers, dashboard audit/rename | ✅ Available |
| `data` | availability, lookups, time-periods, recalculation, input-values, narrative-values, change-requests, data-lock, bulk-validation | ✅ Available |
| `reporting` | computed-values, dashboards, widgets | ✅ Available |
| `notifications` | notification templates (CRUD + Excel, tokens, lint), notification rules (CRUD + Excel, lint, runs, recipients), delivery log | ✅ Available |
| `perf` | memory-throughput diagnostics | ✅ Available |
| `security` | users Excel import/export, user get | ✅ Available |
| `security` | user CRUD, roles, version | ⏳ Planned |
| `system` | tenant CRUD, schema/sample, teardown, tenant fiscal config, local bootstrap; language list via `auth languages` | ✅ Available |
| `system` | language CRUD, translations | ⏳ Planned |

---

## Error Codes

| Code | Meaning | Resolution |
|------|---------|------------|
| `AUTH_REQUIRED` | Not logged in | Run `cap auth login` |
| `TOKEN_EXPIRED` | Session expired | Re-run command (auto-refresh) or `cap auth login` |
| `NOT_FOUND` | ID doesn't exist | Verify ID with `list` command |
| `VALIDATION_ERROR` | Invalid input | Check JSON structure matches DTO |
| `FORBIDDEN` | No permission | Check user roles/data permissions |

---

## See Also

- [SKILL.md](../SKILL.md) — Vocabulary translation and domain overview
- [Template Selection Guide](./template-selection.md) — Choose capture/report/widget/dashboard flows quickly
- [Glossary](./glossary.md) — Term definitions and mappings
- [Output Formats](./output-formats.md) — Table vs JSON output examples
- [Recipes](../recipes/README.md) — Step-by-step workflows
