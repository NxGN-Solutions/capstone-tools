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

> **Tip:** Use `cap data time-periods list --data-interval <interval>` to discover available period names before running reporting commands.

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
| `auth apikey` | `create`, `list`, `prune`, `revoke` |
| `config` | `get`, `set`, `unset` |
| `data` | `availability` |
| `data change-requests` | `create`, `delete`, `get`, `list`, `save`, `validate` |
| `data data-lock` | `lock`, `unlock` |
| `data input-values` | `create`, `download-excel`, `get`, `list`, `save`, `upload-excel`, `validate` |
| `data lookups` | `get` |
| `data narrative-values` | `get`, `get-by-context`, `get-by-metric-context`, `history`, `list`, `save`, `submit`, `validate` |
| `data recalculation` | `status`, `wait` |
| `data time-periods` | `diagnose`, `list` |
| `masterdata data-sources` | `create`, `delete`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `masterdata discipline-attribute-types` | `create`, `delete`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `masterdata disciplines` | `create`, `delete`, `download-excel`, `get`, `import-json`, `list`, `save`, `upload-excel` |
| `masterdata framework-attribute-types` | `create`, `delete`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `masterdata frameworks` | `create`, `delete`, `download-excel`, `get`, `import-json`, `list`, `save`, `upload-excel` |
| `masterdata org-node-attribute-types` | `create`, `delete`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `masterdata org-nodes` | `create`, `delete`, `download-excel`, `get`, `import-json`, `list`, `save`, `upload-excel` |
| `masterdata units` | `create`, `delete`, `download-excel`, `get`, `import-json`, `list`, `save`, `upload-excel` |
| `meta lookups` | `get`, `list` |
| `model calculation-overrides` | `copy`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `model calculations` | `create`, `delete`, `download-excel`, `get`, `get-bulk`, `import-json`, `list`, `save`, `upload-excel`, `validate-batch` |
| `model disciplines` | `list` |
| `model formula-validation` | `validate` |
| `model frameworks` | `list` |
| `model input-overrides` | `copy`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `model inputs` | `create`, `delete`, `download-excel`, `get`, `get-bulk`, `import-json`, `list`, `save`, `upload-excel` |
| `model lookups` | `get`, `list` |
| `model metric-attribute-types` | `create`, `delete`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `model metric-framework-nodes` | `copy`, `download-excel`, `list`, `save`, `upload-excel` |
| `model metric-org-node-exclusions` | `copy`, `list`, `save` |
| `model metrics` | `delete-eval`, `explain`, `get`, `graph`, `list` |
| `model narrative-attribute-types` | `create`, `delete`, `download-excel`, `get`, `get-all`, `list`, `lookup`, `save`, `upload-excel` |
| `model narrative-overrides` | `copy`, `create`, `download-excel`, `get`, `list`, `save`, `upload-excel` |
| `model narratives` | `create`, `delete`, `delete-preview`, `delete-start`, `delete-status`, `distinct-values`, `download-excel`, `get`, `list`, `lookup`, `save`, `upload-excel` |
| `model org-nodes` | `list` |
| `notifications templates` | `create`, `delete`, `get`, `list`, `save` |
| `perf` | `memory-throughput` |
| `reporting computed-values` | `audit`, `download-excel`, `list`, `query` |
| `reporting dashboards` | `get-data`, `get-insights` |
| `reporting widgets` | `get-data`, `info-card`, `pie-chart`, `table`, `xy-chart` |
| `root` | `concepts`, `schema`, `status`, `version` |
| `security users` | `download-excel`, `upload-excel` |
| `system tenants` | `bootstrap-local`, `create`, `delete`, `delete-status`, `delete-watch`, `fiscal-config update`, `get`, `list`, `sample`, `save`, `schema`, `snapshot`, `teardown` |
| `templates capture-templates` | `create`, `delete`, `download-excel`, `get`, `list`, `sample`, `save`, `schema`, `upload-excel` |
| `templates dashboard-templates` | `audit`, `create`, `delete`, `download-excel`, `get`, `import-json`, `list`, `rename`, `sample`, `save`, `schema`, `upload-excel` |
| `templates lookups` | `get` |
| `templates org-node-templates` | `create`, `delete`, `download-excel`, `get`, `list`, `sample`, `save`, `schema`, `upload-excel` |
| `templates report-templates` | `create`, `delete`, `download-excel`, `get`, `list`, `sample`, `save`, `schema`, `upload-excel` |
| `templates widget-templates` | `create`, `delete`, `download-excel`, `get`, `get-bulk`, `import-json`, `list`, `sample`, `save`, `schema`, `upload-excel` |
| `update` | `check` |
| `workflows` | `list`, `show` |

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
| Metric Attribute Types | `cap model metric-attribute-types list` | Custom metric attributes |
| Metric Framework Nodes | `cap model metric-framework-nodes list` | Metric-to-framework associations |
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
| Change Requests | `cap data change-requests list --data-interval quarter --periods "..."` | Governed input and narrative edit requests |
| Data Locks | `cap data data-lock lock --data-interval quarter --periods "Q1 FY 25" --org-nodes <id> --discipline-nodes <id>` | Lock/unlock scoped data periods |

### Get Single Item

```bash
cap <domain> <entity> get <id> [--json]
```

`cap masterdata org-nodes get <id> --json` returns selected attribute values by default. Add `--include-values` only when you need the full available value lists for each attribute type.

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
```

`model metrics get` and rich list output include `friendlyName`, `metricType`,
`formula`, `calculationPhase`, `orgStructureAggregationMethod`, and
`timePeriodAggregationMethod`. Input and calculation `get`/`list` text output
shows a **Friendly Name** field when one is set; JSON payloads include
`friendlyName` for create/save round trips.
Formula graph commands resolve both ID tokens and display-name tokens such as
`[Sample Metric]`.

### Get Multiple Items (Bulk)

```bash
cap templates widget-templates get-bulk <id1> <id2> ... [--json]
cap templates widget-templates get-bulk "<id1>,<id2>" [--json]
cap model inputs get-bulk <id1> <id2> ... --json
cap model calculations get-bulk <id1> <id2> ... --json
```

Retrieves multiple full entities in one CLI invocation. Returns partial results — valid entities are returned alongside errors for invalid or inaccessible IDs. Model `get-bulk` commands return the same editable DTOs as `get`, so automation can hydrate list results before analysis or save-round-tripping without spawning one process per entity.

**Supported entities:** `templates widget-templates`, `model inputs`, `model calculations`

### Identity For Imports And Upserts

The CLI uses one canonical identity contract for JSON batch/upsert workflows and
Excel upload workflows:

| Entity shape | Match key | Rule |
|--------------|-----------|------|
| Flat entities | Exact full name | Names are unique within the entity type. Leading/trailing whitespace is ignored and matching is case-sensitive. A matching full name updates the existing entity; no match creates a new entity. |
| Tree entities | Exact full path | Paths are unique for tree entities. Leading/trailing whitespace is ignored and matching is case-sensitive. A matching full path updates the existing node; no match creates a new node at that path. |

Do not use fuzzy matching, aliases, partial names, or tree-node short names for
upsert identity. Agents can inspect the `upsertIdentity` section from
`cap schema --json` before planning imports. The same section includes
`surfaces` entries for `json-batch-upsert` and `excel-upload`, so automation can
choose rerunnable JSON imports or spreadsheet uploads intentionally.

Masterdata, model definition, and widget/dashboard template JSON imports post
save requests using the same API identity semantics. Use `--upsert` with
`--dry-run` to preview create/update actions before applying:

```bash
cap masterdata units import-json --file units.json --upsert --json
cap masterdata org-nodes import-json --file org-nodes.json --upsert --json
cap masterdata disciplines import-json --file disciplines.json --upsert --json
cap masterdata frameworks import-json --file frameworks.json --upsert --json
cap model inputs import-json --file inputs.json --upsert --json
cap model calculations validate-batch --file calculations.json --upsert --json
cap model calculations import-json --file calculations.json --upsert --json
cap templates widget-templates import-json --file widget-templates.json --upsert --json
cap templates dashboard-templates import-json --file dashboard-templates.json --upsert --json
```

`--dry-run --upsert` returns `would-create` and `would-update` plan actions
without posting changes. Successful upserts return `updated` counts and `idMap`
entries alongside created items.

### Tenant Snapshot Export

Use snapshot export when you need an auditable, import-ready copy of the current
tenant's model-building surface:

```bash
cap system tenants snapshot --output snapshot --json
```

The command reads from the API only and writes JSON files locally:

| File | Re-import command | Identity |
|------|-------------------|----------|
| `masterdata/units.json` | `cap masterdata units import-json --file masterdata/units.json --upsert --json` | Exact full name |
| `masterdata/org-nodes.json` | `cap masterdata org-nodes import-json --file masterdata/org-nodes.json --upsert --json` | Exact full path |
| `masterdata/disciplines.json` | `cap masterdata disciplines import-json --file masterdata/disciplines.json --upsert --json` | Exact full path |
| `masterdata/frameworks.json` | `cap masterdata frameworks import-json --file masterdata/frameworks.json --upsert --json` | Exact full path |
| `model/inputs.json` | `cap model inputs import-json --file model/inputs.json --upsert --json` | Exact full name |
| `model/calculations.json` | `cap model calculations import-json --file model/calculations.json --upsert --json` | Exact full name |
| `templates/widget-templates.json` | `cap templates widget-templates import-json --file templates/widget-templates.json --upsert --json` | Exact full name |
| `templates/dashboard-templates.json` | `cap templates dashboard-templates import-json --file templates/dashboard-templates.json --upsert --json` | Exact full name |

`manifest.json` records counts, paths, import commands, and identity rules.
Snapshot export is not restore: review the files, run `--dry-run --upsert`
imports, validate calculations, then apply the explicit upsert commands.

### Restore From Snapshot Workflow

There is no automatic `cap restore` command. Restore is an explicit reviewed
workflow over the snapshot files so users can inspect which entities would be
created or updated before mutating a tenant.

```bash
cap workflows show restore-from-snapshot --json
```

The workflow order is:

1. Confirm the active target tenant with `cap status --json` and
   `cap auth whoami --json`.
2. Dry-run masterdata imports from `snapshot/masterdata/*.json`, then apply
   units, org nodes, disciplines, and frameworks with `import-json --upsert`.
3. Dry-run model imports, run
   `cap model calculations validate-batch --file snapshot/model/calculations.json --upsert --json`,
   then apply inputs and calculations.
4. Dry-run template imports, then apply widget templates before dashboard
   templates.
5. Wait for recalculation settlement and audit computed values/dashboard
   templates before treating the target tenant as restored.

### Discover Time Periods

```bash
cap data time-periods list --data-interval <type> [--json]
cap data availability --data-interval month [--org-nodes "<id>"] [--json]
```

| Type | Command |
|------|---------|
| Monthly | `cap data time-periods list --data-interval month` |
| Quarterly | `cap data time-periods list --data-interval quarter` |
| Yearly | `cap data time-periods list --data-interval year` |
| Weekly | `cap data time-periods list --data-interval week` |
| Daily | `cap data time-periods list --data-interval day` |

### Lookup Enums

```bash
cap meta lookups list --json
cap meta lookups get <name> [--domain model|data|templates] --json
cap <domain> lookups get <name> [--json]
```

| Domain | Available Lookups |
|--------|-------------------|
| `model` | `metric-types`, `time-period-types`, `time-period-aggregation-methods`, `org-structure-aggregation-methods`, `calculation-phases` |
| `templates` | `widget-sizes`, `widget-types`, `data-grouping-types`, `data-range-modes`, `metric-selection-modes` |
| `data` | `change-request-reasons`, `change-request-status-types`, `change-request-validation-levels` |

Use `meta lookups` when building scripts or agents that need one discovery
surface for all enum/reference values. The domain-specific commands remain
available for direct lookup calls.

### Reporting Commands

Computed values, dashboards, and widgets. Most require `--data-interval` and `--periods` (exceptions: `widgets get-data` auto-detects the interval from the widget template but still requires `--periods`; `computed-values query` and `computed-values audit` can resolve recent periods with `--period-count`).

```bash
# Computed values (via report template filter lens)
cap reporting computed-values list --template <report-template-id> --data-interval day --periods "Jan 25" [--json]

# Framework-grouped report template, optionally narrowed to a framework node
cap reporting computed-values list --template <framework-report-template-id> --framework-nodes <framework-node-id> --data-interval year --periods "FY 2026" --json

# Computed values (ad-hoc metric query, no saved report template required)
cap reporting computed-values query --metrics <metric-id>[,<metric-id>] --data-interval month --period-count 3 --include-metric-details --json

# Computed values audit for post-build verification and CI smoke checks
cap reporting computed-values audit --metrics <metric-id>[,<metric-id>] --data-interval month --periods "Jan 25" --target-model-version <version> --strict --json
```

| Flag | Description | Required |
|------|-------------|----------|
| `--template <id>` | Report template whose filters determine visible metrics | Yes |
| `--metrics "<ids>"` | Metric IDs for ad-hoc `computed-values query` or `audit` | Yes for `query`, `audit` |
| `--data-interval <interval>` | day, week, month, quarter, year | Yes |
| `--periods "<names>"` | Comma-separated period names | Yes for `list`; optional for `query`/`audit` when `--period-count` is supplied |
| `--period-count <n>` | Resolve the most recent N periods for `computed-values query`/`audit` | No |
| `--org-nodes "<id>"` | Filter to specific org node | No |
| `--discipline-nodes "<id>"` | Narrow the report template discipline scope | No |
| `--framework-nodes "<id>"` | Narrow the report template framework scope; framework-grouped reports render metrics under framework paths | No |
| `--metric-types "<types>"` | `input`, `calculation` (comma-separated) | No |

`computed-values audit --json` returns `ComputedValueAuditResult` with
`passed`, `summary`, `modelState`, `findings`, and `missingMetricIds`. Findings
include stale model state, no data rows, all-null values, and requested metrics
missing from the response. Add `--strict` when CI should receive exit code `6`
for audit findings while still getting the JSON body.

```bash
# Widget data (type-specific — returns typed JSON for rendering)
cap reporting widgets info-card <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" [--json]
cap reporting widgets pie-chart <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" [--json]
cap reporting widgets xy-chart <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" [--json]
cap reporting widgets table <widget-template-id> --org-nodes <id> --data-interval quarter --periods "Q1 FY 25" [--json]

# Widget data (legacy CSV compatibility payload, not dashboard Table render JSON)
cap reporting widgets get-data <widget-template-id> --org-node <id> --periods "FY 24, FY 25" [--json]

# Dashboard data
cap reporting dashboards get-data <dashboard-template-id> --org-node <id> --data-interval month --periods "Jan 25" [--json]
cap reporting dashboards get-insights <dashboard-template-id> <layout-node-id> --org-node <id> --data-interval month --periods "Jan 25" [--json]

# Dashboard template static audit before live data checks
cap templates dashboard-templates audit <dashboard-template-id> --strict --json
```

> **Note:** Type-specific widget commands (`info-card`, `pie-chart`, `xy-chart`, `table`) use `--org-nodes` (plural, comma-separated) and can infer `--data-interval`/static periods from the widget template when configured. The `get-data` command uses `--org-node` (singular ID), auto-detects the data interval from the widget template, requires `--periods`, and returns the legacy CSV compatibility envelope, not the Table dashboard render response. Prefer `reporting computed-values` or typed widget commands for automation-safe checks. Use `cap data time-periods list --data-interval <interval>` to discover available period names.

**Table widget command discovery for agents:**

```bash
cap templates widget-templates --help
cap templates widget-templates list --help
cap templates widget-templates get --help
cap templates widget-templates get-bulk --help
cap templates widget-templates create --help
cap templates widget-templates save --help
cap templates widget-templates import-json --help
cap templates widget-templates download-excel --help
cap templates widget-templates upload-excel --help
cap templates dashboard-templates import-json --help
cap templates dashboard-templates audit --help
cap reporting widgets --help
cap reporting widgets table --help
cap reporting widgets get-data --help
```

Use `cap reporting widgets table ... --json` when an agent needs the shared dashboard table contract with `timePeriodColumns`, `metricColumns`, `gridRows`, `metadataColumns`, `totalCount`, and paging support. Use `cap reporting widgets get-data ... --json` when an agent needs the legacy CSV compatibility envelope for analysis workflows.

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
  "widgetType": { "id": 0, "name": "InfoCard" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "dataItems": [
    {
      "metric": { "id": "<metric-id>" },
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "sortOrder": 1
    }
  ]
}' | cap templates widget-templates create --json

# PieChart
echo '{
  "name": "Waste Breakdown",
  "widgetType": { "id": 1, "name": "PieChart" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 0, "name": "Dynamic" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "showLegend": true,
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "dataItems": [
    {
      "metric": { "id": "<metric-id>" },
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "showInLegend": true,
      "showTooltip": true,
      "sortOrder": 1
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
  "axes": [{ "name": "Value", "dynamicAxis": true }],
  "timePeriodAggregationMethod": { "id": 0, "name": "None" },
  "dataItems": [
    {
      "metric": { "id": "<metric-id>" },
      "xyChartWidgetTemplateDataItemType": { "id": 2, "name": "Line" },
      "xyChartWidgetTemplateAxis": { "name": "Value", "dynamicAxis": true },
      "timePeriodAggregationMethod": { "id": 0, "name": "None" },
      "showInLegend": true,
      "showTooltip": true,
      "sortOrder": 1
    }
  ]
}' | cap templates widget-templates create --json

# Table (dimensional dashboard table)
echo '{
  "name": "Portfolio KPI Matrix",
  "title": "Portfolio KPI Matrix",
  "widgetType": { "id": 4, "name": "Table" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 3, "name": "Quarter" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "tableVersion": 1,
  "missingValuePlaceholder": "-",
  "groupingMode": { "id": 1, "name": "Org Node" },
  "metricRowSource": {
    "metrics": [],
    "metricTypeFilters": [],
    "disciplineFilters": [],
    "frameworkFilters": [],
    "metricAttributeFilters": [],
    "orgNodePartitioningMode": { "id": 0, "name": "None" }
  },
  "orgNodeRowSource": {
    "rowSelectionMode": { "id": 0, "name": "Children" },
    "orgNodeAttributeFilters": []
  },
  "includeOrgNodeName": true,
  "includeOrgNodePath": false,
  "includeOrgNodeAttributes": [
    { "id": "<sector-attribute-type-id>", "name": "Sector" }
  ],
  "includeMetricName": false,
  "includeMetricReference": false,
  "includeMetricAttributes": [],
  "includeDiscipline": false,
  "includeFramework": false,
  "columnLayout": { "id": 3, "name": "Metric Then Period" },
  "metrics": [
    {
      "metric": { "id": "<revenue-metric-id>", "name": "Revenue" },
      "sortOrder": 0
    },
    {
      "metric": { "id": "<ebitda-metric-id>", "name": "EBITDA" },
      "sortOrder": 1
    }
  ],
  "periodSet": [
    { "id": "<q1-period-id>", "name": "Q1 FY 25" },
    { "id": "<q2-period-id>", "name": "Q2 FY 25" }
  ],
  "extraColumns": [
    {
      "sortOrder": 0,
      "headerTranslations": [{ "value": "YoY %", "language": { "code": "en" } }],
      "calculationMetricSelection": {
        "metric": { "id": "<yoy-calculation-metric-id>", "name": "Revenue YoY %" }
      },
      "formatter": { "id": 2, "name": "Percent" },
      "precision": 1,
      "alignment": { "id": 2, "name": "Right" }
    }
  ],
  "dimensionalSorts": [
    {
      "targetDimension": { "id": 1, "name": "Row context" },
      "targetAttributeType": { "id": "<sector-attribute-type-id>", "name": "Sector" },
      "direction": { "id": 0, "name": "Ascending" },
      "nullPlacement": { "id": 0, "name": "Last" },
      "sortOrder": 0
    }
  ],
  "formatter": { "id": 0, "name": "Number" },
  "precision": 2,
  "unitMode": { "id": 3, "name": "Cell" },
  "alignment": { "id": 2, "name": "Right" },
  "widthMode": { "id": 0, "name": "Auto" }
}' | cap templates widget-templates create --json
```

> **Note:** The CLI auto-wraps JSON in `{"widgetTemplate": {...}}` if the root object has `name` or `widgetType` properties. You can also use the explicit wrapper format if preferred.

Table templates use the same shared WidgetTemplate API and CLI command surface as the existing widget types. Manage them with `cap templates widget-templates list|get|get-bulk|create|save|import-json|delete|download-excel|upload-excel`, and set the widget type to `{ "id": 4, "name": "Table" }` in JSON or the corresponding Table widget type in Excel. Table data comes from selected Input or Calculation metrics and their ComputedValues; do not encode hidden widget-only calculations in the template.

**Widget Types:**
| ID | Name | Use Case |
|----|------|----------|
| 0 | InfoCard | Single KPI value |
| 1 | PieChart | Part-to-whole breakdown |
| 2 | XYChart | Time series / trends |
| 4 | Table | Dashboard grid/table render |

**Table validation highlights:**
| Area | Rule |
|------|------|
| `tableVersion` | Must be `1` |
| `groupingMode` | Required; use `Metric` rows or `Org Node` rows |
| Row source | `GroupingMode=Metric` needs at least one metric selector; `GroupingMode=Org Node` needs `orgNodeRowSource.rowSelectionMode` |
| Row context | At least one include toggle or attribute list must be configured |
| `columnLayout` | Required; `Periods`, `Metrics`, `Metric Then Period`, and `Period Then Metric` derive generated value columns from `metrics` and `periodSet` |
| `extraColumns` | Optional per-row Calculation metric columns for patterns such as YoY |
| Render budget | The platform defaults to 2,000 rendered cells and has a 10,000-cell hard limit; these are not designer JSON inputs |
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

### Validate Data

```bash
cap data input-values validate <id> --result <approve|reject> [--comments "..."] [--json]
cap data change-requests validate <id> --result <approve|reject> [--comments "..."] [--json]
cap data narrative-values validate <id> --result <approve|reject> [--comments "..."] [--json]
```

| Result | Description |
|--------|-------------|
| approve | Approve the value (moves to next validation level or complete) |
| reject | Reject the value (requires --comments explaining reason) |

### Narrative Workflow

```bash
cap model narratives lookup --capture-interval quarter --org-nodes <id> --discipline-nodes <id> --json
cap data narrative-values save --file narrative-value.json --json
cap data narrative-values submit <value-id> --json
cap data narrative-values validate <value-id> --result approve --comments "Approved" --json
cap data narrative-values get-by-context --narrative <id> --org-node <id> --period-type quarter --start-date 2026-01-01 --json
cap data narrative-values history <value-id> --json
```

For locked periods, apply the data lock first, then use the unified change
request workflow. Change requests can carry narrative edits in the same
contract surface as input edits:

```bash
cap data data-lock lock --data-interval quarter --periods "Q1 FY 25" --org-nodes <id> --discipline-nodes <id> --description "Close narrative capture" --json
cap data change-requests create --file change-request.json --json
cap data change-requests validate <request-id> --result approve --comments "Approved" --json
cap data data-lock unlock --data-interval quarter --periods "Q1 FY 25" --org-nodes <id> --discipline-nodes <id> --description "Reopen narrative capture" --json
```

Narrative JSON input uses the Capstone narrative request
contracts. Use `--file <path>` or pipe JSON on stdin; use `--json` so automation
receives full IDs and contract-shaped response envelopes.

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
```

Tree-node uploads for `masterdata org-nodes`, `masterdata disciplines`, and `masterdata frameworks` use the async upload pipeline. The command sends the file, receives an upload id, and polls `MasterData/TreeNodes/Uploads/{uploadId}/Status` until `Succeeded` or `Failed`.

```bash
cap masterdata org-nodes upload-excel "artifacts/performance/excel/Org Structure.xlsx" \
  --timeout-seconds 3000 \
  --json
```

The default timeout is `3000` seconds (50 minutes). Use `--timeout-seconds <n>` for larger files. In JSON mode the final status payload includes `uploadId`, `status`, `fileName`, `fileSizeBytes`, `totalRows`, `processedRows`, `createdDate`, `completedDate`, and `errors`.

Non-async Excel upload commands return `ExcelUploadResult` in JSON mode:
`items`, row counts such as `savedRows`/`newRows`/`updatedRows`, `isValid`, and
`uploadErrors` with `rowIndex`, `columnIndex`, and `errorMessage`. Keep these
row/cell diagnostics for spreadsheet workflows; use `import-json --upsert` when
the source of truth is JSON and `orderedPlan`/`idMap` diagnostics are more
useful.

**Entities supporting Excel upload:**
- `model inputs`, `model calculations`
- `model metric-attribute-types`, `model metric-framework-nodes`
- `masterdata units`, `masterdata data-sources`
- `masterdata disciplines`, `masterdata frameworks`, `masterdata org-nodes`
- `masterdata discipline-attribute-types`, `masterdata framework-attribute-types`, `masterdata org-node-attribute-types`
- `templates capture-templates`, `templates report-templates`
- `templates widget-templates`, `templates dashboard-templates`, `templates org-node-templates`
- `data input-values`

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
cap auth login                      # OAuth login (interactive)
cap auth login --with-api-key       # API key auth (reads CAPSTONE_API_KEY env var)
cap auth logout                     # Clear stored credentials
cap auth whoami                     # Show current user/tenant
cap auth tenants                    # List accessible tenants
cap auth switch-tenant <id>         # Change active tenant
```

**API Key Authentication (CI/automation):**
```bash
export CAPSTONE_API_KEY=your-api-key
cap auth login --with-api-key
```

---

## Command Status Summary

The schema-derived command surface above is the exact implemented command list.
This table is a high-level availability summary only.

| Domain | Commands | Status |
|--------|----------|--------|
| `root` | schema, concepts, status, version | ✅ Available |
| `workflows` | list, show | ✅ Available |
| `update` | check | ✅ Available |
| `config` | set, get, unset | ✅ Available |
| `auth` | Login, logout, tenant selection, current-user context, API key management, available language list | ✅ Available |
| `meta` | cross-domain lookups | ✅ Available |
| `model` | metrics, inputs, calculations, narratives, model tree aliases, lookups | ✅ Available |
| `model` | metric/narrative attribute types, framework nodes, org-node exclusions, input/calculation/narrative overrides, formula validation | ✅ Available |
| `masterdata` | org-nodes, disciplines, frameworks, units, data-sources | ✅ Available |
| `masterdata` | Excel commands for above entities | ✅ Available |
| `masterdata` | discipline-attribute-types, framework-attribute-types, org-node-attribute-types | ✅ Available |
| `masterdata` | data-exports | ⏳ Planned |
| `templates` | All CRUD + Excel, schema/sample helpers, dashboard audit/rename | ✅ Available |
| `data` | availability, lookups, time-periods, recalculation, input-values, narrative-values, change-requests, data-lock | ✅ Available |
| `reporting` | computed-values, dashboards, widgets | ✅ Available |
| `notifications` | notification templates | ✅ Available |
| `perf` | memory-throughput diagnostics | ✅ Available |
| `security` | users Excel import/export | ✅ Available |
| `security` | user CRUD, roles, version | ⏳ Planned |
| `system` | tenant CRUD, schema/sample, teardown, tenant fiscal config, local bootstrap, snapshot; language list via `auth languages` | ✅ Available |
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
