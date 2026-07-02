# Output Formats

> How the CLI formats responses in table and JSON modes.

---

## Output Modes

Every command supports two output modes:

| Mode | Flag | Stdout | Stderr | Use Case |
|------|------|--------|--------|----------|
| **Table** (default) | (none) | Formatted ASCII table | Errors, warnings | Human reading |
| **JSON** | `--json` | Structured JSON | JSON errors/warnings | AI/automation, piping |

**Tip:** Always use `--json` when the output will be processed programmatically.

---

## JSON Contract Versions

Default `--json` output remains the existing per-command shape. This keeps
current scripts working: list commands still return their list envelope, get
commands still return their entity-specific envelope, and create/save/delete
commands still return the legacy success object shown below.

For automation that wants a stable top-level contract, set
`CAPSTONE_OUTPUT_VERSION=2` before running commands. Commands that use the
shared formatter then wrap JSON output as:

```json
{
  "ok": true,
  "data": {},
  "error": null,
  "warnings": [],
  "meta": {
    "outputVersion": 2,
    "command": null
  }
}
```

Errors in this mode use the same top-level fields with `ok: false`, `data:
null`, and an error object containing `code`, `message`, `details`, and
`retriable`.

Some commands expose a command-specific `--output-version` option for legacy
detail envelopes. Prefer `cap schema --json` for command-specific arguments;
the environment variable controls only the shared formatter envelope.

---

## Exit Codes and Error Classification

Use the process exit code for automation decisions. Use `cap schema --json` for
the live contract; the stable categories are:

| Exit code | Meaning | Typical causes |
|-----------|---------|----------------|
| `0` | Success | Command completed normally; valid analysis requests with no data are still success with diagnostics or warnings. |
| `1` | General CLI failure | User cancellation, local process failure, or unexpected CLI exception. |
| `2` | Validation/client-actionable failure | Unknown command, invalid option, missing argument, local file/schema validation, not found, API `400`, `404`, `409`, or `422`. |
| `3` | Authentication/tenant failure | Missing/expired auth, missing tenant, API `401` or `403`. |
| `4` | Server/API/network failure | Network failure, API `429`, API `5xx`, or malformed API response. |
| `5` | Timeout | API `408` or CLI-controlled polling/command timeout. |
| `6` | Partial result | Imports/uploads/checks that completed with row/item errors or strict audit findings. |

Machine-readable errors are written to stderr in JSON mode. In output v2 they
use the same top-level envelope with `ok: false`, `data: null`, and
`error.code`, `error.message`, `error.details`, and `error.retriable`.

Invalid local JSON input files are validation errors (`2`). Malformed API
responses are server/API contract failures (`4`) and use the shared response
format error path. API rate limiting (`429`) is retriable and exits `4`, not
validation.

---

## List Output (Collections)

### Table Mode

```
ID         | Name                  | Discipline        | Unit
-----------|-----------------------|-------------------|--------
<id>  | Electricity Usage     | Energy            | kWh
<id>  | Water Consumption     | Water             | kL
<id>  | Safety Incidents      | Health & Safety   | count
```

- Column widths auto-size to fit the longest value
- IDs are truncated to 8 characters + `...` for readability
- Columns are separated by ` | ` with a divider row

### JSON Mode

```json
{
  "items": [
    {
      "id": "<id>",
      "name": "Electricity Usage",
      "discipline": "Energy",
      "unit": "kWh"
    },
    {
      "id": "<id>",
      "name": "Water Consumption",
      "discipline": "Water",
      "unit": "kL"
    }
  ]
}
```

- Full IDs (not truncated)
- Property names in `camelCase`
- Null values omitted
- Pretty-printed with indentation

---

## Tree List Output (Hierarchical Entities)

Entities like org nodes, disciplines, and frameworks display as trees.

### Table Mode

```
ID         | Name                  | Level | Path
-----------|-----------------------|-------|------------------------------
<id>  | Corporate             | 0     | Corporate
<id>  |   Region 1            | 1     | Corporate > Region 1
<id>  |     Site A            | 2     | Corporate > Region 1 > Site A
<id>  |     Site B            | 2     | Corporate > Region 1 > Site B
<id>  |   Region 2            | 1     | Corporate > Region 2
```

- Indentation indicates hierarchy depth
- Path column shows the full breadcrumb

### JSON Mode

```json
{
  "items": [
    {
      "id": "<id>",
      "name": "Corporate",
      "level": 0,
      "path": "Corporate",
      "children": [
        {
          "id": "<id>",
          "name": "Region 1",
          "level": 1,
          "path": "Corporate > Region 1"
        }
      ]
    }
  ]
}
```

---

## Single Entity Output (Get)

### Table Mode

```
Field                    Value
------------------------ ------------------------------------------
ID                       <id>
Name                     Electricity Usage
Description              Monthly electricity consumption at site
Discipline               Environmental > Energy
Unit                     kWh
Data Interval            Month
Org Aggregation          Sum
Time Aggregation         Sum
```

- Key-value pairs with label padding for alignment

### JSON Mode

Returns the full entity DTO:

```json
{
  "tenant": "Example Tenant",
  "id": "<id>",
  "name": "Electricity Usage",
  "description": "Monthly electricity consumption at site",
  "discipline": {
    "id": "<id>",
    "name": "Energy",
    "path": "Environmental > Energy"
  },
  "unitOfMeasure": {
    "id": "<id>",
    "name": "kWh"
  },
  "dataInterval": { "id": 2, "name": "Month" },
  "orgStructureAggregationMethod": { "id": 0, "name": "Sum" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" }
}
```

---

## Create / Save / Delete Output

### Table Mode

```
Created Metric with ID: <id>
```

```
Deleted Metric with ID: <id>
```

### JSON Mode

**Create / Save:**
```json
{
  "success": true,
  "id": "<id>"
}
```

**Delete:**
```json
{
  "success": true,
  "deleted": "<id>"
}
```

---

## Narrative Output

Narrative commands follow the same table/JSON split, with API contract objects
inside CLI envelopes:

| Command | JSON Shape |
|---------|------------|
| `cap model narratives list --json` | `{ "tenant": "...", "data": [NarrativeTreeGridRow], "pagination": {...} }` |
| `cap model narratives get <id> --json` | `{ "tenant": "...", "narrative": NarrativeDTO }` |
| `cap data narrative-values list --json` | `{ "tenant": "...", "data": GetNarrativeValueTreeGridListResponse }` |
| `cap data narrative-values get <id> --json` | `{ "tenant": "...", "narrativeValue": NarrativeValueDTO }` |
| `cap data narrative-values history <id> --json` | `{ "tenant": "...", "narrativeValueId": "...", "changes": [NarrativeValueChangeDTO] }` |
| `cap data change-requests list --json` | `{ "tenant": "...", "data": [ChangeRequestGridRow], "pagination": {...} }` |
| `cap data data-lock lock|unlock --json` | `{ "tenant": "...", "success": true, "id": "...", "action": "lock|unlock", ... }` |

Use `--json` for every step in an automated narrative workflow. Table output is
for humans and may truncate IDs.

---

## Excel Upload Output

### Table Mode (Success)

```
Upload successful: 42 Metric(s) imported
```

### Table Mode (Partial Success)

```
Warning: Upload completed with errors: 40 imported, 2 error(s)
Row | Column | Error
----|--------|----------------------------------
3   | 2      | Invalid metric name format
5   | 4      | Duplicate entry
```

### JSON Mode

```json
{
  "items": [
    { "id": "...", "name": "Metric A" },
    { "id": "...", "name": "Metric B" }
  ],
  "uploadErrors": [
    {
      "rowIndex": 3,
      "columnIndex": 2,
      "errorMessage": "Invalid metric name format"
    }
  ],
  "isValid": false
}
```

- `isValid: true` means all rows imported successfully
- `isValid: false` means some rows had errors (partial success)
- `items` contains successfully imported records
- `uploadErrors` lists row/column/message for each failure

---

## Excel Download Output

### Table Mode

```
Downloaded to /path/to/metrics.xlsx (2,458,624 bytes)
```

### JSON Mode

```json
{
  "filePath": "/path/to/metrics.xlsx",
  "fileSize": 2458624
}
```

---

## Widget Data Output

Widget commands have two output families:

| Command | Output Shape | Use Case |
|---------|--------------|----------|
| `cap reporting widgets get-data` | Legacy CSV compatibility object with `format`, `periods`, `sourceMetadata`, and `csvCompatibility.rawCsv` | Backward-compatible widget extraction when typed render commands are not available |
| `cap reporting widgets info-card` | Typed info-card DTO | Dashboard card render validation |
| `cap reporting widgets pie-chart` | Typed pie-chart DTO | Slice/percentage validation |
| `cap reporting widgets xy-chart` | Typed XY-chart render DTO, JSON-first | Browser-aligned series, axis, value, render metadata, and metric metadata validation |
| `cap reporting widgets table` | Typed Table render DTO | Dashboard grid/table render validation |

### XY Chart JSON Mode

`cap reporting widgets xy-chart <id> --json` returns the same server-owned XY render contract consumed by the browser renderer and MCP dashboard renderer. Use it for automation that needs to inspect how an XY chart will render, not just whether metric values exist.

Top-level fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | GUID | Widget/template identifier from the render response |
| `title` | string/null | Widget title |
| `categoryAxes` | array | Category axis definitions and rendered category labels |
| `valueAxes` | array | Value axis definitions, titles, and dynamic-axis flags |
| `chartSeries` | array | Rendered series; each item can include legacy fields plus `renderConfig` and `metricMetadata` |
| `styleMetadata` | array | Schema-limited style metadata for chart-internal elements such as series and axes |
| `invertAxes` | boolean | Legacy/programmatic orientation flag; prefer series `renderConfig.orientation` for render semantics |
| `showLegend` | boolean | Widget legend setting |
| `warnings` | array | Human-readable warnings, including no-data guidance |
| `noData` | object/null | Structured no-data diagnostic when selected periods return no values |
| `diagnostics` | array | Structured render diagnostics for automation and audit |

Each `chartSeries[].renderConfig` is the authoritative render contract for that series:

| Field | Description |
|-------|-------------|
| `seriesKey` | Stable response-local identifier for diagnostics and style metadata |
| `sourceDataItemId`, `sourceMetricId`, `sourcePartitionKey`, `sourcePartitionLabel` | Source widget data item, metric, and optional partition identity |
| `displayLabel` | Label used by the browser/CLI/MCP renderers |
| `configuredSeriesType`, `renderSeriesType` | Authored type and final rendered type (`Column`, `Bar`, `Line`, `Area`) |
| `orientation` | `vertical` or `horizontal` |
| `categoryAxisKey`, `valueAxisKey` | Axis assignment used for rendering |
| `sortOrder`, `renderLayer` | Ordering metadata for legend/data alignment and draw layering |
| `isStacked`, `showInLegend`, `showInTooltip`, `showBullets` | Per-series render behavior |
| `strokeColour`, `fillColour`, `strokeWidth`, `strokeDashArray`, `bulletRadius`, `styleKey` | Per-series style metadata |
| `singlePeriodCategoryKey`, `singlePeriodCategoryLabel` | Category labels used when each series renders as a single aggregate point |
| `comparisonRoleKey`, `comparisonRoleLabel`, `comparisonRoleOrder`, `comparisonRoleSource` | Optional model-derived comparison semantics; labels are not used as chart logic |
| `values` | Values aligned to the rendered categories; can contain `null` gaps |
| `diagnostics` | Series-specific render diagnostics |

Each `chartSeries[].metricMetadata` is emitted only when the metric is available in the caller's authorized reporting scope. It contains model-owned metric semantics and formatting such as `metricType`, `disciplinePath`, `frameworkPaths`, `attributeValues`, unit symbol, precision, aggregation methods, calculation phase, and formula flags. Missing or redacted metadata is reported in `diagnostics`.

Minimal shape:

```json
{
  "title": "Production Trend",
  "categoryAxes": [
    { "name": "Period", "categoryPropertyName": "Period", "categoryValues": ["Jan 26", "Feb 26"] }
  ],
  "valueAxes": [
    { "name": "Volume", "title": "Tonnes", "dynamicAxis": true }
  ],
  "chartSeries": [
    {
      "title": "Actual tonnes",
      "values": [55.0, 61.0],
      "renderConfig": {
        "seriesKey": "xy:actual",
        "sourceMetricId": "<metric-id>",
        "displayLabel": "Actual tonnes",
        "renderSeriesType": { "id": 0, "name": "Column" },
        "orientation": "vertical",
        "categoryAxisKey": "Period",
        "valueAxisKey": "Volume",
        "sortOrder": 0,
        "renderLayer": 10,
        "showInLegend": true,
        "showInTooltip": true,
        "strokeColour": "#1e88e5",
        "values": [55.0, 61.0],
        "comparisonRoleSource": "none",
        "diagnostics": []
      },
      "metricMetadata": {
        "metricId": "<metric-id>",
        "metricName": "Actual tonnes",
        "metricType": "Input",
        "unitOfMeasureSymbol": "t",
        "precision": 1,
        "attributeValues": {}
      }
    }
  ],
  "styleMetadata": [],
  "diagnostics": [],
  "warnings": [],
  "noData": null
}
```

### Pie Chart Widget JSON Mode

`cap reporting widgets pie-chart <id> --json` returns the shared pie/donut
render contract at the top level. It is not wrapped under `template`,
`widgetTemplate`, or `data`. Donut center content is resolved under `center`:
static text uses `center.text`, while Metric Value centers include
`center.label`, `center.value`, and `center.formattedValue`. When
`centerMetricLabel` is blank, the server resolves `center.label` from the metric
friendly name, then the metric name.

Top-level fields:

| Field | Meaning |
|---|---|
| `id` | Widget/template identifier from the render response |
| `title`, `description`, `footnote` | Display text after template resolution |
| `pieChartWidgetType` | Rendered Pie/Donut type |
| `showLegend` | Whether legend rendering is enabled |
| `styleConfiguration` | Server-resolved chart-wide bounded style slots |
| `centerNumberFormat` | Donut Metric Value center display format, when configured |
| `center` | Resolved donut center metadata, when configured |
| `labelDisplay` | Slice-label and tick visibility derived from `sliceLabelMode` |
| `legend` | Resolved legend visibility, value mode, and item labels/values |
| `dataItems[]` | Rendered slices with values, labels, colour, source identity, presentation, and number format |
| `styleWarnings`, `warnings`, `noData` | Non-fatal diagnostics and empty-data state |

Legend values and slice labels are resolved per data item. `dataItems[].legendValue` is populated only when the visible legend row should include the formatted value. `dataItems[].sliceLabel` follows `sliceLabelMode`. The aggregate legend block also exposes `legend.valueMode` and `legend.items[]` for dashboard render parity.

Styled Pie/Donut output also includes the server-resolved styling and automation
metadata used by browser dashboards and MCP apps:

| Field | Meaning |
|---|---|
| `styleConfiguration` | Template-level bounded style slots such as `panel`, `chartArea`, `slices`, `sliceLabels`, `ticks`, `legendValues`, `donutCenter`, and `stateMessages` |
| `centerNumberFormat` | Donut Metric Value center display format, when configured |
| `dataItems[].presentation` | Source data-item non-fill presentation for slice labels, tooltip, legend marker, legend label, and legend value |
| `dataItems[].numberFormat` | Source data-item value display format |
| `dataItems[].sourceDataItemKey` | Stable source identity used to connect runtime slices back to configured data items where available |
| `styleWarnings` / `dataItems[].styleWarnings` | Non-fatal inherited/default style warnings, such as contrast warnings that cannot be resolved at save time |
| `selectionDiagnostics` | Value-selection counts, warnings, and rank/page metadata when widget-level or data-item value selection is configured |

### Table Widget JSON Mode

`cap reporting widgets table <id> --json` returns the shared table/grid render contract:

```json
{
  "title": "Ambient Noise by Site",
  "timePeriodColumns": [
    { "name": "Q1 FY 26", "timePeriodType": 3, "startDate": "2026-01-01T00:00:00", "endDate": "2026-03-31T00:00:00" }
  ],
  "metricColumns": [],
  "metadataColumns": [
    { "field": "unitOfMeasure", "headerName": "Unit" }
  ],
  "gridRows": [
    {
      "id": "<metric-id>",
      "name": "Ambient Noise",
      "path": "Environment -> Ambient Noise",
      "isDataRow": true,
      "unitOfMeasure": { "id": "<unit-id>", "name": "dB(A)" },
      "precision": 2,
      "timePeriodColumns": [
        { "metricValue": 61, "precision": 2 }
      ],
      "metadataValues": { "unitOfMeasure": "dB(A)" }
    }
  ],
  "totalCount": 1,
  "orgNodeId": "<org-node-id>",
  "missingValuePlaceholder": "-"
}
```

### Table Widget Table Mode

Without `--json`, the CLI prints a readable table using `gridRows`,
`metadataColumns`, `timePeriodColumns`, and `metricColumns`, followed by row
counts.

```text
Ambient Noise by Site

Name          | Unit  | Q1 FY 26
--------------+-------+---------
Ambient Noise | dB(A) | 61

1 row(s) shown
```

`widgets table` requires `--data-interval` and plural `--org-nodes`. `widgets get-data` uses singular `--org-node` and auto-detects the data interval from the widget template.

---

## Error Output

Errors are written to **stderr** (not stdout), so they don't interfere with piped JSON output.

### Table Mode

```
Error: Metric not found
  id: <id>
  resourceType: Metric
```

Auth errors include a hint:
```
Error: Authentication required
  Hint: Run 'cap auth login' to authenticate
```

### JSON Mode

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Metric not found"
  }
}
```

### JSON Mode With `CAPSTONE_OUTPUT_VERSION=2`

```json
{
  "ok": false,
  "data": null,
  "error": {
    "code": "NOT_FOUND",
    "message": "Metric not found",
    "details": null,
    "retriable": false
  },
  "warnings": [],
  "meta": {
    "outputVersion": 2,
    "command": null
  }
}
```

### Error Codes

| Code | Meaning | Resolution |
|------|---------|------------|
| `AUTH_REQUIRED` | Not logged in | Run `cap auth login` |
| `AUTH_EXPIRED` | Token expired | Re-run command (auto-refresh) or `cap auth login` |
| `TENANT_REQUIRED` | No tenant selected | Run `cap auth switch-tenant <id>` |
| `NOT_FOUND` | Entity doesn't exist | Verify ID with `list` command |
| `VALIDATION_ERROR` | Invalid input data | Check JSON structure matches DTO |
| `API_ERROR` | Server-side error | Check API status, retry |
| `NETWORK_ERROR` | Connection failed | Check network, verify API URL in config |
| `FORBIDDEN` | No permission | Check user roles and data permissions |

---

## Warning Output

Warnings are written to **stderr** in both modes.

### Table Mode

```
Warning: Upload completed with errors: 40 imported, 2 error(s)
```

### JSON Mode

```json
{
  "warning": "Upload completed with errors: 40 imported, 2 error(s)"
}
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Legacy or uncategorized error |
| `2` | Validation/client input error |
| `3` | Authentication, authorization, or tenant-selection error |
| `4` | Network, API, or server-side error |
| `5` | Timeout waiting for a terminal state |
| `6` | Partial result: useful output was returned with row/item warnings or recoverable errors |

Use the exit code together with stderr JSON or the v2 envelope error object.
For example, `cap data recalculation wait ... --json` returns exit code `5`
when the model state does not settle before the timeout, while stdout still
contains the latest model-state payload for analysis.

---

## Piping and Composition

JSON mode is designed for piping:

```bash
# Get metric IDs and pipe to another tool
cap model metrics list --json | jq '.items[].id'

# Chain commands
METRIC_ID=$(cap model inputs create --file input.json --json | jq -r '.id')
cap model inputs get $METRIC_ID --json
```

Table mode is designed for human reading:

```bash
# Quick scan
cap model metrics list

# Filtered view (combine with grep)
cap model metrics list | grep "Energy"
```

---

## See Also

- [Commands Reference](./commands.md) — Full command listing
- [Glossary](./glossary.md) — Term definitions
- [SKILL.md](../SKILL.md) — CLI overview
