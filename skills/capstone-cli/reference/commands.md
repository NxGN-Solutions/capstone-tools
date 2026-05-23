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

---

## Agent Discovery Commands

These commands are embedded in the binary so agents can inspect the CLI without reading repository docs.

```bash
cap schema --json                  # Command names, args, defaults, mutating marker, output models
cap concepts --json                # Compact Capstone domain ontology
cap workflows list --json          # Workflow index for common agent tasks
cap workflows show inspect-model --json
cap version --json                 # CLI/runtime/install context
cap update check --json            # Explicit update availability check; no install is performed
```

`cap update check --json` uses `CAPSTONE_CLI_RELEASE_MANIFEST_URL` or `--manifest-url` when provided, otherwise it checks GitHub Releases metadata.

---

## Discovery Commands

Commands for exploring and querying data.

### List All Items

```bash
cap <domain> <entity> list [--json]
```

| Entity | Command | Notes |
|--------|---------|-------|
| Metrics (all) | `cap model metrics list` | Includes inputs + calculations; use `--rich` for formulas and aggregation metadata |
| Inputs | `cap model inputs list` | Input metrics only |
| Calculations | `cap model calculations list` | Calculated metrics only |
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
| Change Requests | `cap data change-requests list` | Edit requests |
| Narrative Change Requests | `cap data narrative-change-requests list --data-interval quarter --periods "..."` | Locked narrative edit requests |
| Data Locks | `cap data data-lock lock --data-interval quarter --periods "Q1 FY 25" --org-nodes <id> --discipline-nodes <id>` | Lock/unlock scoped data periods |

### Get Single Item

```bash
cap <domain> <entity> get <id> [--json]
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
```

`model metrics get` and rich list output include `metricType`, `formula`, `calculationPhase`, `orgStructureAggregationMethod`, and `timePeriodAggregationMethod`. Formula graph commands resolve both ID tokens and display-name tokens such as `[Sample Metric]`.

### Get Multiple Items (Bulk)

```bash
cap templates widget-templates get-bulk <id1> <id2> ... [--json]
```

Retrieves multiple widget templates in a single request. Returns partial results — valid templates are returned alongside errors for invalid or inaccessible IDs.

**Supported entities:** `templates widget-templates`

### Discover Time Periods

```bash
cap data time-periods list <type> [--json]
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
cap <domain> lookups <name> [--json]
```

| Domain | Available Lookups |
|--------|-------------------|
| `templates` | `widget-sizes`, `widget-types`, `data-grouping-types`, `data-range-modes`, `metric-selection-modes` |
| `data` | `change-request-reasons`, `change-request-status-types`, `change-request-validation-levels` |

### Reporting Commands

Computed values, dashboards, and widgets. Most require `--data-interval` and `--periods` (exceptions: `widgets get-data` auto-detects the interval from the widget template; `computed-values query` can resolve recent periods with `--period-count`).

```bash
# Computed values (via report template filter lens)
cap reporting computed-values list --template <report-template-id> --data-interval day --periods "Jan 25" [--json]

# Computed values (ad-hoc metric query, no saved report template required)
cap reporting computed-values query --metrics <metric-id>[,<metric-id>] --data-interval month --period-count 3 --include-metric-details --json
```

| Flag | Description | Required |
|------|-------------|----------|
| `--template <id>` | Report template whose filters determine visible metrics | Yes |
| `--metrics "<ids>"` | Metric IDs for ad-hoc `computed-values query` | Yes for `query` |
| `--data-interval <interval>` | day, week, month, quarter, year | Yes |
| `--periods "<names>"` | Comma-separated period names | Yes for `list`; optional for `query` when `--period-count` is supplied |
| `--period-count <n>` | Resolve the most recent N periods for `computed-values query` | No |
| `--org-nodes "<id>"` | Filter to specific org node | No |
| `--metric-types "<types>"` | `input`, `calculation` (comma-separated) | No |

```bash
# Widget data (type-specific — returns typed JSON for rendering)
cap reporting widgets info-card <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" [--json]
cap reporting widgets pie-chart <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" [--json]
cap reporting widgets xy-chart <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" [--json]
cap reporting widgets table <widget-template-id> --org-nodes <id> --data-interval quarter --periods "Q1 FY 25" [--json]

# Widget data (type-agnostic, returns CSV/AI extraction data, not dashboard Table render JSON)
cap reporting widgets get-data <widget-template-id> --org-node <id> --periods "FY 24, FY 25" [--json]

# Dashboard data
cap reporting dashboards get-data <dashboard-template-id> --org-node <id> --data-interval month --periods "Jan 25" [--json]
cap reporting dashboards get-insights <dashboard-template-id> <layout-node-id> --org-node <id> --data-interval month --periods "Jan 25" [--json]
```

> **Note:** Type-specific widget commands (`info-card`, `pie-chart`, `xy-chart`, `table`) use `--org-nodes` (plural, comma-separated) and require `--data-interval`. These commands return typed dashboard render JSON for their widget type. The `get-data` command uses `--org-node` (singular ID), auto-detects the data interval from the widget template, and returns the type-agnostic CSV/AI extraction payload, not the Table dashboard render response. Use `cap data time-periods list` to discover available period names.

**Table widget command discovery for agents:**

```bash
cap templates widget-templates --help
cap templates widget-templates list --help
cap templates widget-templates get --help
cap templates widget-templates get-bulk --help
cap templates widget-templates create --help
cap templates widget-templates save --help
cap templates widget-templates download-excel --help
cap templates widget-templates upload-excel --help
cap reporting widgets --help
cap reporting widgets table --help
cap reporting widgets get-data --help
```

Use `cap reporting widgets table ... --json` when an agent needs the shared dashboard render contract with `columns`, `rows`, `rowContextCells`, `cells`, warnings, and truncation metadata. Use `cap reporting widgets get-data ... --json` when an agent needs the existing CSV extraction envelope for analysis workflows.

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
- `data input-values`, `data change-requests`, `data narrative-change-requests`

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
  "dataItems": [
    { "metric": { "id": "<metric-id>" }, "sortOrder": 1 }
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
  "dataItems": []
}' | cap templates widget-templates create --json

# XYChart (time series) - axes required, each data item must reference an axis
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
  "dataItems": []
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

Table templates use the same shared WidgetTemplate API and CLI command surface as the existing widget types. Manage them with `cap templates widget-templates list|get|get-bulk|create|save|delete|download-excel|upload-excel`, and set the widget type to `{ "id": 4, "name": "Table" }` in JSON or the corresponding Table widget type in Excel. Table data comes from selected Input or Calculation metrics and their ComputedValues; do not encode hidden widget-only calculations in the template.

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

### Validate Data

```bash
cap data input-values validate <id> --result <approve|reject> [--comments "..."] [--json]
cap data change-requests validate <id> --result <approve|reject> [--comments "..."] [--json]
cap data narrative-values validate <id> --result <approve|reject> [--comments "..."] [--json]
cap data narrative-change-requests validate <id> --result <approve|reject> [--comments "..."] [--json]
```

| Result | Description |
|--------|-------------|
| approve | Approve the value (moves to next validation level or complete) |
| reject | Reject the value (requires --comments explaining reason) |

### Narrative Workflow

```bash
cap model narratives lookup --capture-interval quarter --org-nodes <id> --disciplines <id> --json
cap data narrative-values save --file narrative-value.json --json
cap data narrative-values submit <value-id> --json
cap data narrative-values validate <value-id> --result approve --comments "Approved" --json
cap data narrative-values approved-read --narrative <id> --org-node <id> --period-type quarter --start-date 2026-01-01 --json
cap data narrative-values history <value-id> --json
```

For locked periods, apply the data lock first, then use narrative change
requests:

```bash
cap data data-lock lock --data-interval quarter --periods "Q1 FY 25" --org-nodes <id> --discipline-nodes <id> --description "Close narrative capture" --json
cap data narrative-change-requests create --file change-request.json --json
cap data narrative-change-requests validate <request-id> --result approve --comments "Approved" --json
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
- `reporting computed-values` (requires `--template`, `--data-interval`)

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

## Command Status

| Domain | Commands | Status |
|--------|----------|--------|
| `root` | schema, concepts, version | ✅ Available |
| `workflows` | list, show | ✅ Available |
| `update` | check | ✅ Available |
| `auth` | All | ✅ Available |
| `model` | metrics, inputs, calculations | ✅ Available |
| `model` | metric-attribute-types | ✅ Available |
| `model` | metric-framework-nodes | ✅ Available |
| `model` | input-overrides, calculation-overrides, formula-validation | ✅ Available |
| `masterdata` | org-nodes, disciplines, frameworks, units, data-sources | ✅ Available |
| `masterdata` | Excel commands for above entities | ✅ Available |
| `masterdata` | discipline-attribute-types, framework-attribute-types, org-node-attribute-types | ✅ Available |
| `templates` | All CRUD + Excel | ✅ Available |
| `data` | time-periods, input-values, change-requests | ✅ Available |
| `reporting` | computed-values, dashboards, widgets | ✅ Available |
| `security` | users, roles, version | ⏳ Planned |
| `system` | languages, translations, tenants | ⏳ Planned |

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
