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
| `config` | `get`, `list`, `set`, `show`, `unset` |
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
| `reporting widgets` | `get-data`, `info-card`, `pie-chart`, `table`, `text-block`, `textblock`, `xy-chart` |
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
cap templates widget-templates get-bulk -i <id1> -i <id2> -i <id3> [--json]
cap model inputs get-bulk -i <id1> -i <id2> --json
cap model calculations get-bulk -i <id1> -i <id2> --json
```

> IDs are passed as **repeated `-i`/`--id` flags**, one GUID per flag. Positional
> arguments (`get-bulk <id1> <id2>`) and a single comma-joined string
> (`get-bulk -i "id1,id2"`) are **not** accepted and return a validation error.

Retrieves multiple full entities in one CLI invocation. Returns partial results — valid entities are returned alongside errors for invalid or inaccessible IDs. Model `get-bulk` commands return the same editable DTOs as `get`, so automation can hydrate list results before analysis or save-round-tripping without spawning one process per entity.

By default `templates widget-templates get-bulk` returns compact widget summaries.
Add `--full` to return the full editable widget template DTOs — including nested
`valueSelectionConfig` — matching the `get` payload shape, so the output can be
edited and piped back to `save`/`import-json`:

```bash
cap templates widget-templates get-bulk -i <id1> -i <id2> --full --json
```

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
| `templates` | `widget-sizes`, `widget-types`, `data-grouping-types`, `data-range-modes`, `metric-selection-modes`, `metric-partitioning-modes`, `partitioning-rank-modes`, `org-node-row-selection-modes`, `org-node-template-types`, `org-node-template-visibility-modes`, `pie-chart-types`, `xy-chart-data-item-types`, `ai-summary-contexts` |
| `data` | `change-request-reasons`, `change-request-status-types`, `change-request-validation-levels` |

Use `meta lookups` when building scripts or agents that need one discovery
surface for all enum/reference values. The domain-specific commands remain
available for direct lookup calls.

Some widget-template enum sets are schema-owned rather than standalone lookup
endpoints. For Pie/Donut center, legend-value, and slice-label modes, read
`cap templates widget-templates schema --widget-type pie --json` and inspect
`.fieldLookups[]` for `widgetTemplate.centerMode`,
`widgetTemplate.legendValueMode`, and `widgetTemplate.sliceLabelMode`.

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
cap reporting widgets text-block <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" [--json]

# Widget data (legacy CSV compatibility payload, not dashboard Table render JSON)
cap reporting widgets get-data <widget-template-id> --org-node <id> --periods "FY 24, FY 25" [--json]

# Dashboard data
cap reporting dashboards get-data <dashboard-template-id> --org-node <id> --data-interval month --periods "Jan 25" [--json]
cap reporting dashboards get-insights <dashboard-template-id> <layout-node-id> --org-node <id> --data-interval month --periods "Jan 25" [--json]

# Dashboard template static audit before live data checks
cap templates dashboard-templates audit <dashboard-template-id> --strict --json
```

> **Note:** Type-specific widget commands (`info-card`, `pie-chart`, `xy-chart`, `table`, `text-block`) use `--org-nodes` (plural, comma-separated) and can infer `--data-interval`/static periods from the widget template when configured. The `get-data` command uses `--org-node` (singular ID), auto-detects the data interval from the widget template, requires `--periods`, and returns the legacy CSV compatibility envelope, not the Table dashboard render response. Prefer `reporting computed-values` or typed widget commands for automation-safe checks. Use `cap data time-periods list --data-interval <interval>` to discover available period names.

### Dashboard Template Layout Metadata

Dashboard template commands understand the same layout metadata contract across JSON, Excel, audit, schema, and sample surfaces:

```bash
cap templates dashboard-templates schema --json
cap templates dashboard-templates sample --json
cap templates dashboard-templates get <id> --json
cap templates dashboard-templates save --file dashboard.json --json
cap templates dashboard-templates import-json --file dashboard-templates.json --upsert --json
cap templates dashboard-templates audit <id> --strict --json
cap templates dashboard-templates download-excel -o dashboards.xlsx
cap templates dashboard-templates upload-excel -f dashboards.xlsx --json
```

The JSON contract adds `dashboardStyle` at the template root and the following optional groups on `treeItems`:

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
| Layout mode | Default, Stack, Grid, Inline |
| Alignment | Default, Start, Center, End, Stretch |
| Minimum height | None, S, M, L |
| Header behavior | Default, Visible, Hidden, Collapsible, CollapsedByDefault |
| Spacing token | None, XS, S, M, L, XL |
| Responsive breakpoint | Never, Sm, Md, Lg |
| Typography | Default, Compact, Standard, Emphasis |
| Width behavior | WidgetSizeDefault, Auto, Fill |
| Callout variant | None, Info, Success, Warning, Critical, Narrative |
| Callout severity | Default, Low, Medium, High, Critical |

Use safe color values only: named design tokens, `#rgb`, `#rrggbb`, `#rrggbbaa`, bounded `rgb(...)`/`rgba(...)`, or `var(--token-name)`. Use safe icon tokens containing letters, numbers, underscores, or hyphens.

Structured lengths are JSON objects, not CSS strings. Use a single `value` or side-specific `top`, `right`, `bottom`, and `left` numbers with `unit: { "id": 0, "name": "Px" }` or `unit: { "id": 1, "name": "Rem" }`. Free-text CSS shorthands such as `"1rem 2rem"` are rejected.

Excel dashboard-template workbooks include `Dashboard Style JSON` on the dashboard row and row-level metadata columns named `nodeLayoutJson`, `nodeStyleJson`, `placementLayoutJson`, and `calloutJson`. These cells contain the same JSON objects used by `save` and `import-json`.

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
cap templates widget-templates import-json --help
cap templates widget-templates download-excel --help
cap templates widget-templates upload-excel --help
cap templates dashboard-templates import-json --help
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
`styleConfiguration` for dashboard-render parity. Use `cap reporting widgets
pie-chart ... --json` when an agent needs the shared pie/donut render contract
with returned `styleConfiguration`, `centerNumberFormat`, resolved `center`,
`labelDisplay`, `legend`, `dataItems[].presentation`,
`dataItems[].numberFormat`, `dataItems[].legendValue`, and
`dataItems[].sliceLabel` metadata. In non-JSON mode, the command prints
`Center: <label>: <value>` when configured and includes `Legend` and `Slice
Label` columns. Use `cap reporting widgets table ... --json` when an agent
needs the shared dashboard table contract with `timePeriodColumns`,
`metricColumns`, `metricNumberFormats`, `gridRows`, `metadataColumns`,
`selectionDiagnostics`, `styleConfiguration`, `warnings`, `totalCount`, and
paging support. Use `cap reporting widgets text-block ... --json` when an agent
needs the shared TextBlock render contract with resolved `titleContent`,
`subtitleContent`, `descriptionContent`, `footnoteContent`, token spans,
`styleConfiguration`, `warnings`, and `diagnostics`. Use `cap reporting widgets
get-data ... --json` when an agent needs the legacy CSV compatibility envelope
for analysis workflows.

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
| `cap reporting widgets pie-chart <id> ... --json` | Top-level render contract with `id`, `title`, `pieChartWidgetType`, `styleConfiguration`, `center`, `legend`, `labelDisplay`, and `dataItems[]`. |

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
    "title": { "foregroundColor": "text-primary", "fontWeight": "Semibold", "fontSize": 14, "borderWidth": 0 },
    "value": { "foregroundColor": "text-primary", "fontWeight": "Bold", "fontSize": 32 },
    "trend": {
      "accentColor": "neutral",
      "fontWeight": "Medium",
      "showLabel": false,
      "up": { "color": "success", "icon": "arrow-up" },
      "down": { "color": "danger", "icon": "arrow-down" },
      "flat": { "color": "neutral", "icon": "equals" },
      "unknown": { "color": "unknown", "icon": "none" }
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

Info Card comparisons use ordinary selected metrics, Calculation metrics, and display-property tokens. Raw API/CLI JSON uses stored GUID-form tokens such as `[metric-guid]`, `[metric-guid]|-1|`, or `#trend[metric-guid]`; the widget editor and Widget Template Excel use metric names and transpose them to GUIDs internally. Widget Template Excel preserves the same bounded style contract in the `Info Card Style` JSON column; do not add target, benchmark, prior, delta, or delta-percent columns or CLI-only fields.

Info Card style JSON is schema-limited. `fontFamily` must be one of `theme`, `sans`, `serif`, `mono`, `nunito`, `roboto`, `poppins`, or `arial`; `fontSize`, `borderWidth`, and panel `accentWidth` are bounded integers. `accentSide` and `accentWidth` are panel-only fields. A border renders only with positive `borderWidth` plus safe `borderColor`; a panel accent edge renders only with `accentSide`, positive `accentWidth`, and safe `accentColor`.

Trend presentation is configured per Info Card through `styleConfiguration.trend`. Direction keys are `up`, `down`, `flat`, and `unknown`; each direction accepts a safe `color` token or hex value and a bounded `icon` token. Use `icon: "none"` for color-only trends. Do not use raw CSS classes, FontAwesome class names, HTML, scripts, or `url()` values.

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
    "title": { "foregroundColor": "text-primary", "fontWeight": "Semibold", "fontSize": 18 },
    "chartArea": { "backgroundColor": "surface", "padding": 8 },
    "slices": { "opacity": 0.9 },
    "sliceBorders": { "borderColor": "surface", "borderWidth": 2 },
    "ticks": { "borderColor": "border", "borderWidth": 1, "opacity": 0.75 },
    "legendValues": { "foregroundColor": "text-secondary", "fontWeight": "Semibold", "fontSize": 13 },
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
        "legendValue": { "fontWeight": "Semibold", "foregroundColor": "text-secondary" },
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
  "showUnitOfMeasure": true,
  "showMetricsInColumns": false,
  "missingValuePlaceholder": "-",
  "dataItems": [
    {
      "metric": { "id": "<revenue-metric-id>", "name": "Revenue" },
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "sortOrder": 0
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
    },
    "conditionalFormatRules": [
      {
        "configuredColumnId": "metric:<metric-guid-n>",
        "operator": "Negative",
        "style": { "foregroundColor": "danger" }
      }
    ]
  }
}' | cap templates widget-templates create --json
```

> **Note:** The CLI auto-wraps JSON in `{"widgetTemplate": {...}}` if the root object has `name` or `widgetType` properties. You can also use the explicit wrapper format if preferred.

### Widget Value Selection

Widget templates can carry advanced `valueSelectionConfig` in JSON create/save
payloads, `import-json` batches, and widget-template Excel import/export. The
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
| **Dynamic** | `{ "id": 1, "name": "Dynamic" }` | The metrics are resolved at render time from the metric-scope filters below. Omit explicit `dataItems` for dynamic Table templates; Info Card/Pie dynamic samples also omit explicit `dataItems`. |

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
metric-scope filter family: `metricTypeFilters`, `metricDisciplineFilters`,
`metricFrameworkFilters`, `metricAttributeFilters`, `disciplineAttributeFilters`,
or `frameworkAttributeFilters`. Empty arrays still mean "all" for an authored
axis once Dynamic mode has a non-empty filter family.

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
(`metricTypeFilters`, discipline/framework filters, and attribute filters), but
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
| `predicates` | array, max 5 | Filters applied before ranking (see below) |
| `diagnosticsLabel` | string (optional) | Free-text label echoed in selection diagnostics output |
| `boundsWarningMode` | `Default`, `Warn`, `Suppress` (optional) | Controls advisory warnings when the projected element count is large |

**Selector kinds** (`valueSelector.selectorKind`):

| Scope | Supported `selectorKind` | Required selector fields |
|-------|--------------------------|--------------------------|
| `DataItem`, `WidgetDynamicMetricSet` | `AggregatedRange` | `aggregationMethod` (`Sum`/`Average`/`LastValue`/`Min`/`Max`), `fallbackWhenMissing` (`None`/`TreatAsNull`/`Reject`) |
| `TableRows` | `TableMetricAggregate`, `TableRowAggregate` | `aggregationMethod`, `fallbackWhenMissing`; `TableMetricAggregate` also takes `tableColumnKey` (e.g. `"metric:<metric-guid-n>"`). `TableRowAggregate` must not carry `tableColumnKey`. |

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
> metric ID without dashes (the `N` GUID format).

Info Card and the other widget types support per-data-item selection at
`widgetTemplate.dataItems[].valueSelectionConfig` with `ownerScope: "DataItem"`.
Run `cap templates widget-templates sample --widget-type info --json` for a
ready-to-edit example that includes a data-item `valueSelectionConfig`.

Excel stores `valueSelectionConfig` as a single JSON-in-cell column. A blank cell
means no value-selection config, and upload rejects stale/removed fields such as
`changeFilter`, `moversMode`, and `includeOnlyChanged`, as well as removed
change-typed predicate types (`AbsoluteChange`, `PercentChange`, `SignChange`).

Pie and donut templates use the same shared WidgetTemplate API and CLI command surface as the existing widget types. Manage them with `cap templates widget-templates list|get|get-bulk|create|save|import-json|delete|download-excel|upload-excel`, and render live data with `cap reporting widgets pie-chart ...`.

**Pie/donut display fields:**

| JSON field | Excel column | Values / default | Rule |
|------------|--------------|------------------|------|
| `pieChartWidgetTemplateType` | `Pie Chart Type` | `0` Pie Chart, `1` Donut Chart | Required for pie widgets |
| `centerMode` | `Center Mode` | `0` None, `1` Static Text, `2` Metric Value; default `None` | Donut Chart only |
| `centerStaticText` | `Center Static Text` | Free text | Required when `centerMode.id` is `1`; disallowed for Metric Value |
| `centerMetricId` | `Center Metric Id` | Metric GUID | Required when `centerMode.id` is `2`; disallowed for Static Text |
| `centerMetricLabel` | `Center Metric Label` | Optional text | Optional override for Metric Value centers; if blank, render uses metric friendly name, then metric name |
| `legendValueMode` | `Legend Value Mode` | `0` Hidden, `1` Value; default `Value` | Stored enum behind the UI's **Show Legend Value** checkbox; controls numeric values in visible legend rows, not row visibility |
| `sliceLabelMode` | `Slice Label Mode` | `0` Hidden, `1` Label, `2` Value, `3` Label and Value; default `Hidden` | Controls labels rendered on slices, separate from tooltips and legend rows |

Widget-template Excel download/upload includes the display columns above plus `Center Metric`. On upload, set `Center Metric Id` for a stable metric reference or `Center Metric` for an exact-name lookup; if both are present they must resolve to the same metric. Legacy workbooks that omit the new columns continue to import with defaults.

The dashboard editor presents `centerMetricId` through the searchable metric grid picker. The CLI and Excel contracts still use the explicit metric ID/name fields so imports remain stable and diffable.

Table templates use the same shared WidgetTemplate API and CLI command surface as the existing widget types. Manage them with `cap templates widget-templates list|get|get-bulk|create|save|import-json|delete|download-excel|upload-excel`, and set the widget type to `{ "id": 4, "name": "Table" }` in JSON or the corresponding Table widget type in Excel. Table data comes from selected Input or Calculation metrics, their ComputedValues, and optional dynamic or static narrative rows; do not encode hidden widget-only calculations in the template.

TextBlock templates use the same shared WidgetTemplate API, CLI command surface, and widget-template Excel download/upload workflow. Manage them with `cap templates widget-templates list|get|get-bulk|create|save|import-json|delete|download-excel|upload-excel`, set the widget type to `{ "id": 6, "name": "TextBlock" }`, and render live data with `cap reporting widgets text-block ...`. TextBlock discovers metrics from explicit tokens in `title`, `subtitle`, `description`, and `footnote`; do not create hidden `dataItems[]` rows for those references. Excel roundtrips `Subtitle` and `Text Block Style` columns alongside the existing title/description/footnote fields.

**Widget Types:**
| ID | Name | Use Case |
|----|------|----------|
| 0 | InfoCard | Single KPI value |
| 1 | PieChart | Part-to-whole breakdown |
| 2 | XYChart | Time series / trends |
| 4 | Table | Dashboard grid/table render |
| 6 | TextBlock | Metric-aware dashboard text |

**Table validation highlights:**
| Area | Rule |
|------|------|
| `dataGrouping` | Required; controls row grouping. `OrgNode` also requires `orgNodeRowSelectionMode`. `Framework` requires Dynamic metric selection. |
| Metric selection | Static tables use explicit `dataItems[]` unless metric filters provide the report-template fallback. Dynamic tables use metric type, discipline, framework, metric-attribute, discipline-attribute, and framework-attribute filters and clear explicit data items on save. |
| Org-node scope | `orgNodeAttributeFilters` filters the resolved org-node scope before values are loaded; it is not the deferred `OrgNodeAttribute` row-grouping mode. |
| Custom columns | `showMetricValue`, `showUnitOfMeasure`, and `metricAttributeTypeIds` control visible helper/value columns. |
| `narrativeSelectionMode` | `Dynamic` (`id: 0`) resolves narratives from Narrative Scope. `Static` (`id: 1`) renders explicit `narratives[]`. |
| `narrativeScopeConfigured` | Set `true` when Dynamic Narrative Scope is intentionally authored. Empty scope arrays then mean all permitted values; `false`/omitted preserves legacy metric-scope fallback. |
| Narrative Scope | Dynamic scope can filter by `narrativeDisciplineFilters`, `narrativeFrameworkFilters`, `narrativeMetricAttributeFilters`, `narrativeDisciplineNodeAttributeFilters`, `narrativeFrameworkNodeAttributeFilters`, and `narrativeAttributeFilters`. |
| Static narratives | `narratives[]` contains `{ narrative: { id, name }, sortOrder }` entries and is used only when `narrativeSelectionMode` is Static. |
| Org-node template | Do not author `orgNodeTemplateId` on new Table widgets; dashboard config determines the org-node-template lens at render time. |
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
