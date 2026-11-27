# MCP Tool Reference

> Complete reference for all 58 Capstone MCP tools with parameters, response formats, and usage examples.

---

## Authentication (6 tools)

### `auth_login`

Login to Capstone using Identity OIDC (PKCE). Opens a browser. The authorize URL is also written to stderr so headless hosts can complete login without a local browser (`CAPSTONE_NO_BROWSER=1` skips opening one).

No parameters. Identity chooses the SSO provider (typically Microsoft).

**Returns:** Login status message with authenticated user identity and the authorize URL.

---

### `auth_logout`

Revokes the Identity refresh token (`POST {issuer}/connect/revoke`) and always clears local credentials, even if revoke fails.

No parameters.

**Returns:** Confirmation message.

---

### `auth_whoami`

Get current user identity, tenant context, roles, and accessible templates.

No parameters.

**Returns:** Text with the user's email, name and current tenant (from `GET Security/UserPreferences`). With an API key it adds that the key is scoped to one tenant and cannot switch.

---

### `auth_tenants_list`

List available tenants accessible to the current user.

No parameters.

**Returns:** Tenant names and IDs, with the current tenant marked (`POST System/Tenants/GetLookupList`).

---

### `auth_switchTenant`

Switch to a different tenant. Changes become your default for future sessions.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tenantId` | string (ID) | Yes | Tenant ID to switch to (from `auth_tenants_list`) |

**Returns:** Confirmation with new tenant name. Invalidates cached resources. An API key cannot switch: keys are scoped to a tenant when they are created.

---

### `auth_languages_list`

List the languages configured in the Capstone platform, with their codes and the platform language IDs used by localisation and preference APIs.

No parameters.

**Returns:** One line per language: name, code and platform ID (`POST System/Localisation/Languages/GetLookupList`).

---

## Model Read (12 tools)

### `model_metrics_list`

List metrics in the Capstone model. Returns metric hierarchy with names, types, units, and parent relationships.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Search by metric name (case-insensitive substring match) |
| `metricType` | string | No | Filter by type: `input`, `calculation`, or omit for all |
| `disciplineId` | string (ID) | No | Filter by discipline ID |
| `limit` | int | No | Maximum results (default: 100) |

**Returns:** Tree grid of metrics with name, type (Input/Calculation), discipline, unit of measure, data interval, and calculation details.

**Response structure:**
```json
{
  "gridRows": [
    {
      "id": "<id>",
      "name": "Electricity Consumption",
      "path": "Electricity Consumption",
      "metricType": "Input",
      "discipline": { "id": "...", "name": "Environmental", "path": "Environmental" },
      "unitOfMeasureName": "kWh",
      "dataIntervalName": "Month",
      "precision": 2,
      "formula": null,
      "calculationPhase": null,
      "orgStructureAggregationMethod": { "id": 0, "name": "Sum" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" }
    },
    {
      "id": "<id>",
      "name": "Total Energy",
      "path": "Energy > Total Energy",
      "metricType": "Calculation",
      "discipline": { "id": "...", "name": "Environmental", "path": "Environmental" },
      "unitOfMeasureName": "kWh",
      "dataIntervalName": "Month",
      "precision": 2,
      "formula": "SUM(Electricity, Gas, Diesel)",
      "calculationPhase": { "id": 1, "name": "Phase 2" },
      "orgStructureAggregationMethod": { "id": 0, "name": "Sum" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" }
    }
  ]
}
```

> **Tip:** Inputs and Calculations are **types of Metrics**, not separate entities. Use the `metricType` parameter to filter.

---

### `model_metrics_get`

Get full details for a single metric including formula, aggregation settings, and exclusions.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `metricId` | string (ID) | Yes | Metric ID to retrieve |

**Returns:** Formatted metric details including type, discipline, unit of measure, data interval, precision, and editability. Uses `POST Model/Metrics/GetTreeGridList` internally and filters client-side.

**Note:** Output is formatted text (not JSON). Example:
```
Metric: Electricity Consumption
==============================

ID: <id>
Type: Input

Classification:
  Discipline: Environmental
  Discipline Path: Environmental

Measurement:
  Unit of Measure: kWh
  Data Interval: Month
  Precision: 2 decimal places

Editable: Yes

Tree Path: Energy > Electricity Consumption
```

**For Calculation metrics, the formula field is populated:**
```json
{
  "formula": "IF [Total Denominator] <> 0 THEN [Total Numerator] / [Total Denominator] ELSE 0",
  "calculationPhase": { "id": 1, "name": "After Aggregations" }
}
```

---

### `model_inputOverrides_list`

List input overrides. One row is an input metric at an org node.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Metric name contains this text |
| `orgNodeId` | string (ID) | No | Only overrides for this org node |
| `disciplineId` | string (ID) | No | Only overrides in this discipline |
| `limit` | int | No | Maximum rows (default: 200) |

**Returns:** Text rows with id, site, discipline, unit, precision, validation, capture, and bands (`Metric bands`, `None (formatting off)`, or a count), and the total when more rows match. With no `orgNodeId`, every org node's overrides are listed. The search runs on the server, so a match past the first `limit` rows is still found. `GET` is not used; the call is `POST Model/InputOverrides/GetGridList`.

### `model_inputOverrides_get`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Input override ID |

**Returns:** Text for one override, including each band's bounds and colour. `GET Model/InputOverrides?id=`.

### `model_calculationOverrides_list`

List calculation overrides. Same shape as input overrides, without validation and capture flags. Calculation bands reject comment-required and auto-validate.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Metric name contains this text |
| `orgNodeId` | string (ID) | No | Only overrides for this org node |
| `disciplineId` | string (ID) | No | Only overrides in this discipline |
| `limit` | int | No | Maximum rows (default: 200) |

**Returns:** Text rows with the site, as for input overrides. `POST Model/CalculationOverrides/GetGridList`.

### `model_calculationOverrides_get`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Calculation override ID |

**Returns:** Text for one override. `GET Model/CalculationOverrides?id=`.

### `model_inputs_lint`

Check an input definition without saving it. Errors are exactly what saving would reject. Warnings and notes never block a save: a warning is a risk, such as `DataIntervalChangeDeletesValues` (changing the data interval deletes every stored value of the input, locked and approved ones included); a note is worth knowing but harmless, such as `NameSimilarToExisting` (a name close to an existing metric's).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `inputJson` | string (JSON) | One of the three | The input definition, as posted to save |
| `inputId` | string (ID) | One of the three | A saved input to lint as it is |
| `all` | bool | One of the three | Lint every saved input and return a summary |

**Returns:** One line per issue, errors first, then warnings, then notes: `Error   [Code] field: message`, `Warning …`, `Note    …`, then a count line; or "No issues found in the input." With `all`, the summary described under `model_calculations_lint`. `POST Model/Inputs/Lint`.

### `model_calculations_lint`

Check a calculation without saving it, with the codes the web editor, CLI and Excel upload use. Errors are exactly what saving would reject: the formula's own errors (`FormulaRequired`, `FormulaSyntax`, `FormulaUnknownMetric`, `FormulaUnknownFunction`, `FormulaFuturePeriod`, `FormulaCircular`, each placed on its line and column except an empty formula's `FormulaRequired`; a circular formula sits on the reference that closes the cycle), `CalculationPhaseRequired`, both aggregations when results add up (`TimeAggregationRequired`, `OrgAggregationRequired`) and the shared metric rules. Warnings never block a save: `PhaseDiffersFromSuggestion` (the formula's shape suggests the other phase, with the reason), `BeforeOperandBlank` (an operand is blank when results add up), `BeforeInputLongerThanInterval` (an input captured less often with Sum, Last value or None is split, put in the last period or blank before the formula runs; a repeated Average, Min or Max factor doesn't warn) and `BeforeRollDownNotReported` (Roll Down hides the results from reports).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `calculationJson` | string (JSON) | One of the three | The calculation definition, as posted to save; the formula may use metric names or ids |
| `calculationId` | string (ID) | One of the three | A saved calculation to lint as it is |
| `all` | bool | One of the three | Lint every saved calculation and return a summary (the phase audit) |

**Returns:** For one calculation, the same lines as `model_inputs_lint`. With `all`: `Linted N calculations: X with errors, Y with warnings.`, then one line per code (`Error   FormulaUnknownMetric (1): Diesel intensity`, `Warning PhaseDiffersFromSuggestion (2): …`), then `Failing: …` with the calculations saving would refuse. `GET Model/Calculations?id=` for each, then `POST Model/Calculations/Lint`.

---

### `model_orgNodes_list`

List organization structure (hierarchical tree of locations, regions, business units).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Filter by name (case-insensitive substring match) |
| `limit` | int | No | Maximum results to return (default: all) |

**Returns:** Tree grid of org nodes with name, level, tree path, and parent relationships.

**Response structure:**
```json
{
  "gridRows": [
    {
      "id": "<id>",
      "name": "Corporate",
      "level": 0,
      "treePath": "Corporate",
      "parentId": null
    },
    {
      "id": "<id>",
      "name": "Site A",
      "level": 1,
      "treePath": "Corporate > Site A",
      "parentId": "<id>"
    }
  ]
}
```

---

### `model_frameworks_list`

List reporting frameworks (GRI, SASB, CDP, TCFD, etc.) configured in the tenant.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Filter by name (case-insensitive substring match) |
| `limit` | int | No | Maximum results to return (default: 100) |

**Returns:** Tree grid of framework nodes with names and hierarchy.

---

### `model_metricFrameworkNodes_list`

List one page of metrics (inputs and calculations) with whether each is linked to a framework item, sorted by name,
with the total. The same list as the Metric Framework Assignments screen; it shows only the metrics you may view.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `frameworkNodeId` | string | No | Framework item ID from `model_frameworks_list`; omit to flag metrics linked to any framework item |
| `search` | string | No | Metric name contains (case-insensitive, on the server) |
| `metricType` | string | No | `input` or `calculation` |
| `disciplineId` | string | No | Discipline ID |
| `linkedOnly` | bool | No | Only the linked metrics (to `frameworkNodeId`, or to any framework item when it is omitted) |
| `limit` | int | No | Page size, capped at the platform maximum (default: 100) |
| `offset` | int | No | Metrics to skip, for the next page (default: 0) |

**Returns:** `Name | Linked | Discipline | ID` rows, a "Metrics 1–100 of N (L linked on this page)" header, a
"T metrics in all, K linked." line, and the offset for the next page when more match.

---

### `model_disciplines_list`

List disciplines (metric categories like Environmental, Social, Governance).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Filter by name (case-insensitive substring match) |
| `limit` | int | No | Maximum results to return (default: 100) |

**Returns:** Tree grid of discipline nodes with names and hierarchy.

---

## Data Operations (10 tools)

### `data_timePeriods_list`

List available time periods for a given data interval.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `dataInterval` | string | No | Period type: `day`, `week`, `month` (default), `quarter`, `year` |

**Returns:** List of time periods with names, start dates, and end dates.

**Response structure:**
```json
{
  "timePeriods": [
    { "id": "...", "name": "Jan 2025", "startDate": "2025-01-01T00:00:00Z", "endDate": "2025-01-31T23:59:59Z" },
    { "id": "...", "name": "Feb 2025", "startDate": "2025-02-01T00:00:00Z", "endDate": "2025-02-28T23:59:59Z" }
  ]
}
```

**Period name formats by interval:**

| Interval | Example Names |
|----------|---------------|
| `day` | `2026-01-15` |
| `week` | `(W5) Jan 26 2026` |
| `month` | `Jan 2026` |
| `quarter` | `Q1 FY 25` |
| `year` | `FY 2025` |

---

### `data_availability`

Check which time periods have data. Useful for finding periods to query before calling data tools.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `dataInterval` | int | No | Day=0, Week=1, Month=2 (default), Quarter=3, Year=4 |

> **Note:** This tool accepts `dataInterval` as an integer, unlike `data_timePeriods_list` which accepts a string. This is a known inconsistency.

**Returns:** List of periods with data availability indicators.

---

### `data_inputValues_list`

Query raw input data (manually entered or imported values) with validation status and lock state. Returns a compact pivot table: one row per metric×org, time periods as columns. Use a capture template ID (from `templates_captures_list`) OR pass `templateJson` for ad-hoc queries. For calculated/aggregated results, use `data_computedValues_list` instead. When the template places grouping levels on columns, each pivot row takes its org node, discipline and framework from the column's group, with Discipline or Framework columns added for those levels.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `templateId` | string (ID) | Conditional | Capture template ID (from `templates_captures_list`). Optional if `templateJson` is provided. |
| `templateJson` | string (JSON) | Conditional | Inline capture template JSON (alternative to `templateId`). Minimum: `{"name":"Ad-hoc","dataGrouping":{"id":1},"spreadsheetTemplateMetrics":[{"metric":{"id":"METRIC_GUID"},"sortOrder":0}]}`. `dataGrouping` id: 0=None, 1=OrgNode, 2=Discipline, 3=Framework. Optional `additionalDataGrouping` is an ordered list of `{id: 1|2|3}` Then By levels. Get metric IDs from `model_metrics_list`. |
| `timePeriodNames` | string | No | Comma-separated period names (e.g., `Jan 2025,Feb 2025`) |
| `periodType` | string | No | `month`, `quarter`, or `year` — alternative to `timePeriodNames` |
| `periodCount` | int | No | Number of recent periods (default: 4, used with `periodType`) |
| `dataInterval` | int | No | Day=0, Week=1, Month=2 (default), Quarter=3, Year=4 |
| `orgNodeIds` | string | No | Comma-separated org node IDs. Held inside the template's Limit to org nodes (inside its defaults when `showOrgNodeFilter` is false); omit to use the template defaults |
| `disciplineNodeIds` | string | No | Comma-separated discipline IDs, held inside the template's Limit to disciplines the same way |
| `frameworkNodeIds` | string | No | Comma-separated framework IDs, held inside the template's Limit to frameworks the same way |
| `metricNames` | string | No | Include only metrics matching these names (comma-separated, case-insensitive substring match, e.g. `Revenue,Profit`) |
| `excludeMetricNames` | string | No | Exclude metrics matching these names (comma-separated, case-insensitive substring match, e.g. `Change %`) |
| `limit` | int | No | Maximum pivot rows to return (default: 500) |
| `offset` | int | No | Number of pivot rows to skip for pagination (default: 0) |

**Returns:** Grid of input values with metric name, org node, period, value, validation status, and lock state.

**Response structure:**
```json
{
  "timePeriodColumns": [
    { "name": "Jan 2025" },
    { "name": "Feb 2025" }
  ],
  "gridRows": [
    {
      "id": "row-id",
      "name": "Electricity Consumption",
      "path": "Electricity Consumption->Corporate->Site A",
      "isDataRow": true,
      "unitOfMeasure": { "name": "kWh", "id": "uom-id" },
      "precision": 2,
      "groupingKey": "org-node-id",
      "timePeriodColumns": [
        {
          "inputValue": {
            "id": "value-id",
            "value": 15000.50,
            "validationStatus": 2,
            "isLocked": false,
            "userCanCapture": true
          }
        },
        {
          "inputValue": {
            "id": "<empty-id>",
            "validationStatus": 0,
            "isLocked": false,
            "userCanCapture": true
          }
        }
      ]
    }
  ]
}
```

**Key points:**
- `gridRows[].timePeriodColumns` aligns with the top-level `timePeriodColumns` by index
- A zero ID in `inputValue.id` means no value has been entered for that cell
- `value` field is absent (not null) when no data exists
- `path` uses `->` separators: `MetricName->OrgPath`
- `groupingKey` is the org node ID for that row

> **Period resolution:** Specify periods either by name (`timePeriodNames`) or by type+count (`periodType` + `periodCount`). Use `data_timePeriods_list` to see available period names.

---

### `data_inputValues_save`

Save input values (batch upsert). Uses business-key matching — zero IDs resolve to existing records automatically.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `inputValues` | JSON array | Yes | Array of input value objects (see format below) |

**Input value object:**
```json
{
  "id": "<empty-id>",
  "value": 1500,
  "metric": { "id": "<metric-id>" },
  "orgNode": { "id": "<org-node-id>" },
  "timePeriodType": { "id": 2, "name": "Month" },
  "startDate": "2025-03-01T00:00:00Z"
}
```

**Business key:** `metric` + `orgNode` + `timePeriodType` + `startDate` uniquely identifies each value. The `id` field can always be the zero ID — the API resolves existing records by business key. This makes saves idempotent.

**Units:** the value is read in the unit the capture grid shows for that org node — the node's own unit override when one exists, otherwise the metric unit. The tool resolves that unit for you, and the API converts it to the stored metric unit with the unit conversion (a fixed factor, or the factor metric's value for that node and period).

**TimePeriodType values:**

| ID | Name |
|----|------|
| 0 | Day |
| 1 | Week |
| 2 | Month |
| 3 | Quarter |
| 4 | Year |

**Comments:** add `"comments": "..."` to explain a value. Omitting it keeps the stored comment; `""` clears it.

**Excluded org nodes (`OrgNodeExcluded`):** an input excluded at an org node cannot be captured there; the capture grid shows the cell as n/a. If any value targets such an (input, org node) pair, the save fails with 400 and nothing in the request is saved; the error reads `The input is excluded at this org node and cannot be captured there.` Clearing a value (an empty `value`) at an excluded org node is rejected too. The exclusion covers that org node only, not its children.

**Returns:** Save result with success/failure status. Values saved in a range that needs a comment, without one, are listed by id: ask the user why each value is correct, then save it again with `comments`.

---

### `data_computedValues_list`

**Primary data analysis tool.** Returns a compact pivot table: one row per metric×org, time periods as columns. Use a report template ID (from `templates_reports_list`) OR pass `templateJson` for ad-hoc queries. Set `dataInterval` for granularity: 0=day, 1=week, 2=month, 3=quarter, 4=year. Use for monthly summaries, weekly trends, daily drill-downs, cross-site comparisons. Returns read-only calculated results. When the template places grouping levels on columns, each pivot row takes its org node, discipline and framework from the column's group, with Discipline or Framework columns added for those levels.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `templateId` | string (ID) | Conditional | Report template ID (from `templates_reports_list`). Optional if `templateJson` is provided. |
| `templateJson` | string (JSON) | Conditional | Inline report template JSON (alternative to `templateId`). Minimum: `{"name":"Ad-hoc","dataGrouping":{"id":1},"spreadsheetTemplateMetrics":[{"metric":{"id":"METRIC_GUID"},"sortOrder":0}]}`. `dataGrouping` id: 0=None, 1=OrgNode, 2=Discipline, 3=Framework. Optional `additionalDataGrouping` is an ordered list of `{id: 1|2|3}` Then By levels. Optional `expandCalculations` / `expandCalculationsMaxDepth` (1..20) nest formula dependencies under calculation rows. Get metric IDs from `model_metrics_list`. |
| `timePeriodNames` | string | No | Comma-separated period names (e.g., `Jan 2025,Feb 2025`) |
| `periodType` | string | No | `month`, `quarter`, or `year` — alternative to `timePeriodNames` |
| `periodCount` | int | No | Number of recent periods (default: 4, used with `periodType`) |
| `dataInterval` | int | No | Day=0, Week=1, Month=2 (default), Quarter=3, Year=4 |
| `orgNodeIds` | string | No | Comma-separated org node IDs. Held inside the template's Limit to org nodes (inside its defaults when `showOrgNodeFilter` is false); omit to use the template defaults |
| `orgNodeTemplateKey` | string | No | Org structure: an org node template ID or `physical`; omit for the template's default structure. Only templates with `showOrgNodeTemplateFilter: true` can switch, and the key must be one of their allowed structures (see `templates_reports_get`); any other key returns no rows. An inline `templateJson` needs `showOrgNodeTemplateFilter: true` to switch. `orgNodeIds` must belong to the chosen structure |
| `disciplineNodeIds` | string | No | Comma-separated discipline IDs, held inside the template's Limit to disciplines the same way |
| `frameworkNodeIds` | string | No | Comma-separated framework IDs, held inside the template's Limit to frameworks the same way |
| `metricNames` | string | No | Include only metrics matching these names (comma-separated, case-insensitive substring match) |
| `excludeMetricNames` | string | No | Exclude metrics matching these names (comma-separated, case-insensitive substring match, e.g. `Change %`) |
| `includeMetricDetails` | bool | No | Include metric formula, aggregation, and discipline info section (default: false) |
| `limit` | int | No | Maximum pivot rows to return (default: 500) |
| `offset` | int | No | Number of pivot rows to skip for pagination (default: 0) |

> **Time period precedence:** If both `timePeriodNames` and `periodType`/`periodCount` are provided, `timePeriodNames` takes priority. If neither is provided, defaults to the 4 most recent months.

**Returns:** Grid of computed values with metric name, period, and formatted value. A Capture or Validation completion template shows each cell's state or share done, with a Total column; with `showWhoIsNext` on it adds a last **Who is next** column naming who captures or validates next (roles separated by `; `).

**Response structure:**
```json
{
  "timePeriodColumns": [
    { "name": "Q1 FY 25" },
    { "name": "Q2 FY 25" }
  ],
  "gridRows": [
    {
      "id": "row-id",
      "name": "GHG Emissions",
      "path": "Environmental > Emissions > GHG Emissions",
      "isDataRow": true,
      "unitOfMeasure": { "name": "tonnes CO2e" },
      "precision": 2,
      "timePeriodColumns": [
        { "value": 1850.00 },
        { "value": 1720.50 }
      ]
    },
    {
      "id": "row-id",
      "name": "Energy Use",
      "path": "Environmental > Energy > Energy Use",
      "isDataRow": true,
      "unitOfMeasure": { "name": "kWh" },
      "precision": 0,
      "timePeriodColumns": [
        { "value": 485000 },
        { "value": 462000 }
      ]
    }
  ]
}
```

**Key points:**
- Rows where `isDataRow: true` contain actual values — other rows are grouping headers
- `timePeriodColumns` in each row aligns with the top-level `timePeriodColumns` by index
- Values are numeric (not formatted strings)
- Use the `precision` field to format decimal places for display

**Capture and validation progress.** On a report template whose `measure` is Capture (1) or Validation (2) completion, the output starts with the measure, "Counted over the data you may view." when the caller's data access is restricted, the total (for example `Captured: 46%, 6 of 13`) and the whole-period figure of each group in the first two levels. Each cell shows the state of a single value (`Captured`, `Not captured`, `Awaiting validation`, `In validation`, `Validated`, `Rejected`), or the share done of several (`46%`), `Not due` when nothing in it is due yet and `·` when nothing is expected; a Total column gives each row's whole-period figure. Counts cover expected values that are due: a value counts in the column that holds its period's end, and a period is due once its last day is before today's UTC date.

> **Note:** Computed values are read-only calculated results. Use `data_inputValues_save` to modify source data.

---

### `data_computedValues_query`

**Ad-hoc computed values query.** Query specific metrics by ID without needing a report template. Use this when you know which metrics you want — constructs an inline template and calls the same API endpoint as `data_computedValues_list`.

For template-based queries where a report template already defines the metric set, use `data_computedValues_list` instead.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `metricIds` | string | Yes | Comma-separated metric IDs to query |
| `orgNodeIds` | string | No | Comma-separated org node IDs. When multiple are provided, results are grouped by org node. |
| `orgNodeTemplateKey` | string | No | Org structure: an org node template ID or `physical`; omit for the physical structure. Org node IDs must belong to that structure. |
| `dataInterval` | int | No | Day=0, Week=1, Month=2 (default), Quarter=3, Year=4 |
| `timePeriodNames` | string | No | Comma-separated period names (e.g., `Jan 2025,Feb 2025`) |
| `periodType` | string | No | `month`, `quarter`, or `year` — alternative to `timePeriodNames` |
| `periodCount` | int | No | Number of recent periods (default: 4, used with `periodType`) |
| `excludeMetricNames` | string | No | Exclude metrics matching these names (comma-separated, case-insensitive substring match, e.g. `Change %`) |
| `includeMetricDetails` | bool | No | Include metric formula, aggregation, and discipline info section (default: false) |
| `limit` | int | No | Maximum pivot rows to return (default: 500) |
| `offset` | int | No | Number of pivot rows to skip for pagination (default: 0) |

> **Time period precedence:** Same as `data_computedValues_list` — `timePeriodNames` takes priority over `periodType`/`periodCount`.

**Returns:** Same format as `data_computedValues_list` — grid of computed values with metric name, period, and formatted value.

**Example workflow:**
```
1. model_metrics_list(search="Revenue")  → find metric IDs
2. model_orgNodes_list(search="Site 1")  → find org node ID
3. data_computedValues_query(
     metricIds="<revenue-id>,<cost-id>",
     orgNodeIds="<site-id>",
     dataInterval=2,
     periodType="month",
     periodCount=6
   )
```

This bypasses the template discovery workflow — no need to list templates, guess which one covers your metrics, then query.

---

### `data_narrative_aggregation_status`

**Narrative roll-up status.** Whether the workspace's rolled-up narrative text has caught up with narrative changes: the narrative version, how far the roll-ups have caught up, and how many narratives wait, failed or have waited past the stuck time. No parameters. Read-only; the log and Retry waiting are in the web app and the CLI (`cap data narrative-aggregation log`).

**Returns:** A short status: up to date at a version, or catching up with the waiting, failed and stuck counts.

**Needs:** any page's view permission, as the header (HB-678).

---

### `data_recalculation_status`

**Calculation status.** Whether calculated values have caught up with the model, in the header's words: not run yet,
running (calculated up to version N of M), failed (each failing metric and its message), or up to date at a model
version. No parameters. Read-only. Needs any page's view permission.

---

### `data_myWork`

**What waits for me.** Per data gallery (capture, validation, change request approval): the gallery totals and the
sheets with waiting work, busiest first; a sheet that holds the gallery's whole total comes last. Capture counts values
to capture and rejected values to fix, validation the values waiting for your level, approval the change request values
you can decide now. Counts only your own sites, disciplines and levels: the same numbers as your home page and the
galleries' My work tab. A gallery you cannot open shows "nothing waiting".

| Parameter | Type | Description |
|-----------|------|-------------|
| `gallery` | string | Optional: `capture`, `validation` or `approval`. Empty = all three. |

---

### `data_changeRequests_list`

**Change requests.** The tab counts (requests and values waiting on me, raised by me), then the requests in one view:
number, status, site, inputs and narratives, who raised it and when, values and how many are yours to decide. Read-only.

| Parameter | Type | Description |
|-----------|------|-------------|
| `view` | string | Optional: `waiting-on-me`, `raised-by-me`, `rejected` or `all`. Empty = the web's choice (Waiting on me for approvers, Raised by me for everyone else). |
| `limit` | int | Requests to show, 1–100 (default 20). |

## Reporting (3 tools)

### `reporting_completion_followUps`

**People to follow up.** Who is late with data capture or validation across the workspace, most late values first:
each person's values to capture, to validate per level and rejected values to fix, their sites and the oldest late
month. Late means not captured or not validated for a month that ended within the last 12 months, counted as the
progress reports count. "Nobody assigned" is late work nobody can act on. No parameters. Read-only.

**Needs:** Data Validation manage and access to every site and discipline; otherwise it says the list is not available.

**Returns:** How many people are late, then one line per person.

### `reporting_dashboards_getData`

Get all widget data from a dashboard as CSV. A widget that fixes its interval ignores the request interval; a following widget's interval is only the default. Pass `dataInterval` to set the interval for following widgets. Best for: full dashboard export, cross-widget correlation.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `dashboardTemplateId` | string (ID) | Yes | Dashboard template ID (from `templates_dashboards_list`) |
| `orgNodeId` | string (ID) | Yes | Organization node ID |
| `timePeriodNames` | string | No | Comma-separated period names |
| `periodType` | string | No | `month`, `quarter` (default), or `year` |
| `periodCount` | int | No | Number of recent periods (default: 4) |
| `scope` | string | No | `peers` or `descendants` (default) |
| `dataInterval` | int | No | `0=day`, `1=week`, `2=month`, `3=quarter`, `4=year`; ignored by widgets that fix their interval; a following widget's interval is only the default |
| `orgNodeTemplateKey` | string | No | Org structure for a dashboard that lets viewers switch it (`showOrgNodeTemplateFilter: true`): an org node template ID or `physical`; omit for the dashboard default. A key outside the dashboard's allowed structures is refused (403). `orgNodeId` must belong to that structure |

**Returns:** CSV data for all widgets on the dashboard. Use for cross-widget correlation analysis.

---

### `reporting_widgets_getData`

Get one widget's data as CSV. Use a widget template ID (from `templates_widgets_list`) OR pass `templateJson` for ad-hoc queries. A widget that fixes its interval ignores the request interval; a following widget's interval is only the default. Pass `dataInterval` to set the interval for following widgets.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `widgetTemplateId` | string (ID) | Conditional | Widget template ID (from `templates_widgets_list`). Optional if `templateJson` is provided. |
| `templateJson` | string (JSON) | Conditional | Inline widget template JSON (alternative to `widgetTemplateId`). Required fields: `name`, `widgetType` (`{"id":0}` InfoCard, 1 PieChart, 2 XYChart, 4 Table), `dataRangeMode` (`{"id":0}`), `metricSelectionMode` (`{"id":0}`), `dataItems` array. Table templates use `dataGrouping` (`0` None, `1` OrgNode, `2` Discipline, `3` Framework), optional `additionalDataGrouping`, `dataItems` (metrics) or the metric scope filters, optional `narratives`, `showMetricValue`, `showMetricsInColumns`, `metricPropertyColumns`, `metricAttributeTypeIds` and `missingValuePlaceholder`. Optional `discipline` (`{"id":"ID"}`) only files the widget in the widget list. Get metric/discipline IDs from `model_metrics_list` and `model_disciplines_list`. An inline template is capped at 1,000 rows (org nodes × metrics) and at the report grid's cell limit (rows × periods); a larger one is refused with a 400 naming the count. |
| `orgNodeId` | string (ID) | No | Organization node ID (from `model_orgNodes_list`) |
| `timePeriodNames` | string | No | Comma-separated period names (e.g., `Q1 FY 25,Q2 FY 25`) |
| `periodType` | string | No | `month`, `quarter`, or `year` — alternative to `timePeriodNames` |
| `periodCount` | int | No | Number of recent periods (default: 4, used with `periodType`) |
| `dataInterval` | int | No | `0=day`, `1=week`, `2=month`, `3=quarter`, `4=year`; ignored by widgets that fix their interval; a following widget's interval is only the default |

**Returns:** CSV data for the widget's metrics and time periods.

The table render endpoint behind `apps_widget_table` (`Reporting/Widgets/GetTableData`) no longer accepts an inline template; it renders saved templates only. Draft table templates render through `POST Templates/WidgetTemplates/Preview`, which pages like a dashboard table. A `GetTableData` request without `take` returns every row (the response's `pageSize` names the table's Rows per page, else 50, its `maxHeightPx` the grid's on-screen maximum height, `null` when the table fits its rows, and its `defaultExpandedLevels` how many levels of row groups a dashboard viewer starts with open, `null` for every group open); `apps_dashboard_render` and `apps_widget_table` page through every row 1000 at a time (`apps_widget_table` stops at `limit` when one is given), as `cap reporting widgets table` does without `--take`.

`apps_widget_table` renders a table with periods as rows one row per row and period, with a Period column before the metric columns, as the dashboard does. Column headers name their column groups first when a level is on columns.

---

## Notifications (1 tool)

### `notifications_deliveryLog_summary`

**Failed emails.** Notifications sent since a date and how many failed (the home page's count), then the latest 10
failures: when (UTC), which rule, and the recipient and error where you may see them. Read-only.

| Parameter | Type | Description |
|-----------|------|-------------|
| `days` | int | Days back, today included (UTC), 1–90 (default 7). |

**Needs:** Master data view; recipients and errors need Master data manage.

---

## Templates — Discovery (7 tools)

### `templates_dashboards_list`

List dashboard templates (name + ID). Use the ID with: `reporting_dashboards_getData` for CSV data export, `apps_dashboard_render` for visual HTML rendering. Dashboard names indicate their content (e.g. 'Executive P&L Overview', 'Production Performance').

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Filter by name on the server (case-insensitive substring match) |
| `limit` | int | No | Maximum results to return (default: 100); the header gives the total that match |

**Returns:** Grid of dashboard templates with name, description, and ID.

**Response structure:**
```json
{
  "gridRows": [
    { "id": "dash-id", "name": "ESG Executive Dashboard", "description": "Overview of ESG performance" },
    { "id": "dash-id", "name": "Operations Dashboard", "description": "Operational KPIs" }
  ]
}
```

---

### `templates_widgets_list`

List widget templates (name + ID + type). Widget names encode their content: 'Subject | Qualifier | Interval | Type'. Use the ID with: `reporting_widgets_getData` for CSV data, `apps_widget_infoCard`/`apps_widget_pieChart`/`apps_widget_xyChart`/`apps_widget_table` for rendered HTML widgets. Search names to find widgets matching a user's request.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Filter by name on the server (case-insensitive substring match) |
| `limit` | int | No | Maximum results to return (default: 100); the header gives the total that match |

**Returns:** Grid of widget templates with name, widget type (InfoCard/PieChart/XYChart/Table), and ID.

**Response structure:**
```json
{
  "gridRows": [
    { "id": "wt-id", "name": "Emissions Card", "widgetType": { "id": 0, "name": "InfoCard" } },
    { "id": "wt-id", "name": "Energy Mix", "widgetType": { "id": 1, "name": "PieChart" } },
    { "id": "wt-id", "name": "Emissions Trend", "widgetType": { "id": 2, "name": "XYChart" } },
    { "id": "wt-id", "name": "Metric Table", "widgetType": { "id": 4, "name": "Table" } }
  ]
}
```

**Widget Types:**

| ID | Name | Use Case |
|----|------|----------|
| 0 | InfoCard | Single KPI value with optional comparison |
| 1 | PieChart | Part-to-whole relationships, breakdowns |
| 2 | XYChart | Time series, trends, multi-metric comparison |
| 4 | Table | Tabular metric rows and formatted values |

---

### `templates_reports_list`

List available report templates (define which metrics appear in computed value queries).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Filter by name on the server (case-insensitive substring match) |
| `limit` | int | No | Maximum results to return (default: 100); the header gives the total that match |

**Returns:** Grid of report templates with name, description, and ID.

---

### `templates_captures_list`

List available capture templates (define which metrics appear in data entry forms).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Filter by name on the server (case-insensitive substring match) |
| `limit` | int | No | Maximum results to return (default: 100); the header gives the total that match |

**Returns:** Grid of capture templates with name, description, and ID.

---

### `templates_reports_get`

Get detailed configuration of a specific report template.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Report template ID |

**Returns:** Template configuration including `Filed Under` (the discipline the template is filed under in the template list, `-` when unfiled), `Card` (the gallery card look: icon, colour and image file name; `-` when none), filters (discipline, framework, metric type), the Limit to lists (`Org Nodes (Limit to)`, `Disciplines (Limit to)`, `Frameworks (Limit to)`) and the Default lists (`Default Org Nodes`, `Default Disciplines`, `Default Frameworks`), included metrics, plus `showOnlyRowsWithValues` and `showRowsWithValuesFilter` (viewers can switch between all rows and rows with values), and the allowed grouping levels with `showDataGroupingFilter` (viewers can regroup by those levels in any order; `data_computedValues_list` always uses the default grouping), and `Column Levels`: the grouping levels placed on columns (`-` when every level is on rows). It also prints the `Org Structure` (the default structure; physical when none is set) and `showOrgNodeTemplateFilter` (Fixed, or viewers can switch). When viewers can switch, it lists the allowed structures (`Org Structures (Limit to)`; none means every structure the viewer can access) and `Allow Physical Structure`, and the call example includes `orgNodeTemplateKey`. `Measure` gives what the cells show: Value, Capture or Validation completion (see `data_computedValues_list`); on a completion template it also gives its completion bands (for example "below 40%: danger-subtle; 40% to 90%: warning-subtle; 90% and up: success-subtle"), with "(defaults)" when the template stores none. `Show Change Requests` (the report grid marks cells with a pending change request) and `Allow Change Requests Toggle` (viewers get a Change Requests switch); both default to false.

---

### `templates_reports_resolve_content`

List the metrics and narratives a report template resolves to, sorted by name: the same list the template editor shows behind "See list". The template's own scope applies (limit picks and attribute filters); your data permissions and viewer filters do not. A metric excluded at every site a viewer can pick (the "Limit to" sites, or the Default sites when the site filter is fixed) is not listed; exclusions never cascade to child sites. Needs the template management permission.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `templateId` | string (ID) | Conditional | Report template ID (from `templates_reports_list`). Optional if `templateJson` is provided. |
| `templateJson` | string (JSON) | Conditional | Inline report template JSON for a draft (alternative to `templateId`). Minimum: `{"name":"Draft","dataGrouping":{"id":0}}`. Without `spreadsheetTemplateMetrics`, every metric the filters allow is listed. |

Provide exactly one of `templateId` and `templateJson`.

**Returns:** `Resolves to {m} metrics and {k} narratives.`, then a `Metrics:` and a `Narratives:` list of names. Each list stops at 100 names and ends with `…and {n} more` when cut; the counts in the first line are always complete. Only report templates have this tool; capture templates are listed with the CLI `resolve-content` command.

---

### `templates_captures_get`

Get detailed configuration of a specific capture template.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Capture template ID |

**Returns:** Template configuration including `Filed Under` (the discipline the template is filed under in the template list, `-` when unfiled), `Card` (the gallery card look: icon, colour and image file name; `-` when none), assigned metrics, the `Org Structure` (physical when none is set; capture templates can't switch it), the Limit to and Default lists for org nodes, disciplines and frameworks (as `templates_reports_get`), and data entry settings: `showRowsWithValuesFilter` (viewers can switch between all rows and rows with values), `showDataGroupingFilter` with the allowed grouping levels (viewers can regroup by those levels in any order), `Column Levels` (the grouping levels placed on columns), `showPendingFilter` (the capture and validation grids offer a "Show pending only" toggle) and `showOnlyPending` (they open showing only values still to capture, or waiting for the viewing user's validation). `data_inputValues_list` does not apply the pending filter. `Show Change Requests` (the grids mark values with a pending change request) and `Allow Change Requests Toggle` (viewers get a Change Requests switch); both default to false.

---

## MCP Apps — Visualization (7 tools)

> **Date parameter note:** App tools use `startDate`/`endDate` (ISO format dates), unlike data tools which use `timePeriodNames` or `periodType+periodCount`. This is because Apps render visual time ranges, while data tools operate on discrete named periods. **Exceptions:** `apps_chart_render` (ad-hoc visualization) uses neither — data is provided inline. `apps_widget_aiSummary` also uses neither — it takes dashboard and org node IDs only.

### `apps_dashboard_render`

Render a full dashboard as interactive HTML with Chart.js charts. Use when the user wants to SEE a dashboard visually. For raw data analysis, use `reporting_dashboards_getData` (CSV) or `data_computedValues_list` (flexible aggregation) instead. Uses `startDate`/`endDate` (ISO yyyy-MM-dd), not period names.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `dashboardTemplateId` | string (ID) | Yes | Dashboard template ID |
| `orgNodeId` | string (ID) | Yes | Organization node ID |
| `startDate` | string | Yes | Start date (ISO format: yyyy-MM-dd) |
| `endDate` | string | Yes | End date (ISO format: yyyy-MM-dd) |
| `timePeriodType` | string | No | `day`, `week`, `month` (default), `quarter`, `year` |
| `orgNodeTemplateKey` | string | No | Org structure for a dashboard that lets viewers switch it: an org node template ID or `physical`; omit for the dashboard default. Every widget, tables included, the header and AI summaries run on it; a caller without a grant on the default gets 403 and must name a structure. `orgNodeId` must belong to that structure |

**Returns:** HTML with Chart.js visualizations rendered in Claude's conversation. The dashboard header (eyebrow, title, subtitle, badge) is resolved for the org node and period at the template's default interval, so header tokens such as `@[periods]` show their real text; a dashboard without a header shows its name. A Pie or XY chart whose metrics are all excluded at the org node renders "n/a" and is not reported as a widget error.

**Example call:**
```
apps_dashboard_render({
  dashboardTemplateId: "dash-id",
  orgNodeId: "org-id",
  startDate: "2024-10-01",
  endDate: "2025-03-31",
  timePeriodType: "quarter"
})
```

---

### `apps_widget_infoCard`

Render a KPI info card as HTML showing metric values with labels, units, and trend indicators, plus narrative text resolved in its title, description, and footnote. Use when the user wants to SEE a metric card. For raw numbers, use `reporting_widgets_getData` or `data_computedValues_list` instead. Uses `startDate`/`endDate` (ISO yyyy-MM-dd).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `widgetTemplateId` | string (ID) | Yes | Widget template ID (InfoCard type) |
| `orgNodeIds` | string | Yes | Comma-separated org node IDs |
| `startDate` | string | Yes | Start date (ISO format) |
| `endDate` | string | Yes | End date (ISO format) |
| `timePeriodType` | string | No | Period type (default: `month`) |

**Returns:** HTML info card with metric values, labels, and units. Narrative tokens (`{narrative-guid}`) in the title, description, and footnote render as HTML-encoded plain text with line breaks kept; see [Widget text tokens](#widget-text-tokens). An item whose metric is excluded at the org node reports `NotApplicable` with value "n/a" and no warning.

---

### `apps_widget_pieChart`

Render a pie/donut chart as interactive HTML. Shows part-to-whole breakdowns
(e.g. cost structure, revenue mix). Use when the user wants to SEE a chart. For
raw numbers, use `data_computedValues_list` instead. Uses `startDate`/`endDate`
(ISO yyyy-MM-dd).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `widgetTemplateId` | string (ID) | Yes | Widget template ID (PieChart type) |
| `orgNodeIds` | string | Yes | Comma-separated org node IDs |
| `startDate` | string | Yes | Start date (ISO format) |
| `endDate` | string | Yes | End date (ISO format) |
| `timePeriodType` | string | No | Period type (default: `month`) |

**Returns:** HTML with the pie/donut chart and server-resolved render metadata.
For styled templates, the metadata includes the same contract used by browser
dashboards and CLI JSON: `styleConfiguration`, `centerNumberFormat`, `center`,
`labelDisplay`, `legend`, `dataItems[].sourceDataItemKey`,
`dataItems[].presentation`, `dataItems[].numberFormat`,
`dataItems[].legendValue`, `dataItems[].sliceLabel`, `pieShape`, `dataItems[].formattedShare`, style warnings, and
`selectionDiagnostics` when value selection is configured. A Half Circle shape draws the top half of the chart. Small slices
grouped into Other draw as one slice in the `otherSlice` colour (default grey `#8E96A8`); its tooltip lists the members and
"and N more", and the metadata lists the shares where it lists values. As elsewhere in the MCP chart, slice labels are not drawn on
the canvas and a donut centre (including Total and Percentage) is a line under the chart.

---

### `apps_widget_gauge`

Render a gauge as HTML: one metric's value at **one site** on an SVG dial of its ranges. Use when the user wants to SEE where a KPI sits against its ranges. For raw numbers, use `data_computedValues_list` instead. Uses `startDate`/`endDate` (ISO yyyy-MM-dd).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `widgetTemplateId` | string (ID) | Yes | Widget template ID (Gauge type) |
| `orgNodeIds` | string | Yes | The one org node the gauge reads at; the server refuses more than one ("A gauge shows one site") |
| `startDate` | string | Yes | Start date (ISO format) |
| `endDate` | string | Yes | End date (ISO format) |
| `timePeriodType` | string | No | Period type (default: `month`) |

**Returns:** HTML with the dial and, under it, the value, the name of the range it reads in and the distance to the nearest range edge ("0.03 above 2.00"), so the reading never rests on colour alone. The shape is a Half Dial (180°), an Arc Dial (270°) or Solid (270°, filled to the value, no needle). Each range is an arc in the colour the server sends: the gauge's own ranges as chosen, the metric's or site's ranges in their strong shade or as set (the style's `rangeColours`); with conditional formatting off the arcs are grey. A thin tick marks each range edge. The scale works as an XY chart's value axis: a set minimum or maximum holds, an automatic end takes the value and every range edge, Start at zero pulls a positive low end to 0, and an automatic end past an open outer range ("2.00 and above") leaves room (the style's `rangeRoomPercent`, 10 % by default). A value past a set end pins the needle there and says "Off the scale"; on a log scale range edges at or below 0 are skipped.

---

### `apps_widget_xyChart`

Render a bar/line/area chart as interactive HTML (Chart.js). Shows trends, time series, multi-metric comparisons. Use when the user wants to SEE a chart. For raw numbers, use `data_computedValues_list` instead. Uses `startDate`/`endDate` (ISO yyyy-MM-dd).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `widgetTemplateId` | string (ID) | Yes | Widget template ID (XYChart type) |
| `orgNodeIds` | string | Yes | Comma-separated org node IDs |
| `startDate` | string | Yes | Start date (ISO format) |
| `endDate` | string | Yes | End date (ISO format) |
| `timePeriodType` | string | No | Period type (default: `month`) |

**Returns:** HTML with Chart.js bar/line/column chart. An axis with a major step labels every step, as the dashboard does, when that gives at most 50 steps; Chart.js draws no minor ticks between them. A bridge (`chartKind` Bridge) draws as floating bars, From bar to To bar, in the colours the server gave each bar; the tooltip shows a step's change, signed. `apps_dashboard_render` draws a bridge widget the same way.

---

### `apps_widget_table`

Render a table widget as HTML. Shows dashboard-shaped table rows, columns, formatted values, warnings, and truncation state. Use when the user wants to SEE a table widget. For CSV extraction, use `reporting_widgets_getData` instead. Uses `startDate`/`endDate` (ISO yyyy-MM-dd).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `widgetTemplateId` | string (ID) | Yes | Widget template ID (Table type) |
| `orgNodeIds` | string | Yes | Comma-separated org node IDs |
| `startDate` | string | Yes | Start date (ISO format) |
| `endDate` | string | Yes | End date (ISO format) |
| `timePeriodType` | string | No | Period type (default: `month`) |
| `limit` | int | No | Maximum rows to render (default: every row) |

**Returns:** HTML table with row headers, value columns, warnings, and truncation state.

---

### Widget text tokens

InfoCard, TextBlock and Table text is resolved by the API; MCP renders the result (`apps_widget_infoCard`, `apps_widget_textBlock`, `apps_dashboard_render`, and `templates_widgets_preview` for drafts). Stored token syntax in widget template JSON (a config may also name a metric, `[Metric Name]`: lint, save and preview resolve it, and save stores the id). Each resolved value span carries the `periodName` it was read at:

| Token | Meaning |
|-------|---------|
| `[metric-guid]`, `[metric-guid]\|-1\|` | Metric value, optionally offset by widget periods |
| `#trend[metric-guid]` | Metric value with trend direction |
| `@[period]`, `@[period:-1]` | Period label |
| `{narrative-guid}`, `{narrative-guid}\|-1\|` | Narrative text for the selected org unit (TextBlock title/subtitle/description/footnote, and InfoCard and Table title/description/footnote only; no `#trend`) |

A narrative token resolves at the narrative's own capture interval, whatever the widget's data interval: a quarterly narrative shows the quarter containing a selected month, or the latest quarter with text in a selected year, and `|-1|` counts in capture periods. The span's `periodName` names the resolved period. "Not applicable" means the narrative is excluded for the selected org unit. Other states are "No data", "Restricted", "Unavailable", and "Select a single org unit".

---

### `apps_widget_aiSummary`

Render pre-generated AI insights (trends, anomalies, correlations) as HTML cards. These are server-computed insights from the Capstone engine, not your own analysis. Requires a dashboard template ID and a specific dashboard section node ID.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `dashboardTemplateId` | string (ID) | Yes | Dashboard template ID |
| `dashboardTemplateNodeId` | string (ID) | Yes | Dashboard template node ID — specific section to analyze |
| `orgNodeId` | string (ID) | Yes | Organization node ID |

**Returns:** HTML with AI-generated insights and trend analysis.

---

## MCP Apps — Ad-hoc Visualization (1 tool)

### `apps_chart_render`

Render a chart from **raw data** as interactive HTML. No template needed — provide data inline. Use this for ad-hoc visualizations after querying data with `data_computedValues_list` or `data_inputValues_list`. For template-based rendering, use `apps_widget_pieChart`, `apps_widget_xyChart`, `apps_widget_infoCard`, or `apps_widget_table` instead.

> **Convention exception:** Unlike other `apps_` tools, `apps_chart_render` does **not** use `startDate`/`endDate` parameters. Data is provided inline via the `data` parameter.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `chartType` | string | Yes | Chart type: `pie`, `donut`, `bar`, `line`, `area`, or `infoCard` |
| `title` | string | Yes | Chart title displayed above the visualization |
| `data` | string (JSON) | Yes | Data as JSON — format depends on `chartType` (see below) |
| `options` | string (JSON) | No | Display options (see options table below) |

**Data format by chart type:**

**Pie / Donut:**
```json
[
  {"label": "Revenue", "value": 100, "color": "#ff0000", "precision": 2, "symbol": "$", "symbolIsPrefix": true},
  {"label": "Costs", "value": 60, "color": "#00ff00", "precision": 2, "symbol": "$", "symbolIsPrefix": true}
]
```

**Bar / Line / Area:**
```json
{
  "labels": ["Jan", "Feb", "Mar"],
  "series": [
    {"name": "Sales", "values": [100, 200, 150], "type": "bar", "color": "#0000ff", "precision": 0, "symbol": "units"},
    {"name": "Target", "values": [120, 120, 120], "type": "line", "color": "#ff0000", "precision": 0, "symbol": "units"}
  ]
}
```

- Each series can have its own `type` (`bar`, `line`, `area`) for mixed charts
- `color` is optional — defaults are provided

**InfoCard:**
```json
{
  "value": "1,234",
  "unit": "kWh",
  "label": "Energy Consumption",
  "comparison": {"value": "+12%", "label": "vs last month"}
}
```

- `unit`, `label`, and `comparison` are optional

**Per-item formatting fields (all chart types):**

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `precision` | number | 2 (charts), 0 (infoCard) | Decimal places for display |
| `symbol` | string | — | Unit symbol for tooltips (e.g., `"$"`, `"kWh"`, `"%"`) |
| `symbolIsPrefix` | boolean | `false` | Place symbol before value (`true` for `$`, `€`; `false` for `kWh`, `%`) |

**Options:**

| Key | Type | Description |
|-----|------|-------------|
| `subtitle` | string | Subtitle displayed below the title |
| `legendPosition` | string | `top`, `bottom`, `left`, `right`, or `none` |
| `stacked` | boolean | Stack bar/area series |
| `horizontalBars` | boolean | Horizontal bar orientation |
| `valueAxisLabel` | string | Y-axis label |
| `categoryAxisLabel` | string | X-axis label |
| `rotateCategoryLabels` | boolean | Rotate X-axis labels 45 degrees |

Unknown options keys are silently ignored.

**Returns:** Interactive HTML with Chart.js visualization.

---

## Templates — CRUD (13 tools)

> Template CRUD tools create, update, and delete widget, report, and dashboard templates. For listing/discovery of existing templates, see the [Templates](#templates--discovery-7-tools) section above.
>
> A refused create or save (invalid JSON or ID, a missing name, a name already taken, a failed local check, a template that is not there, or the server rejecting it) comes back as a tool error (`isError: true`), its text prefixed `An error occurred invoking '<tool>':`. `templates_widgets_lint` does the same when it cannot run the lint (invalid input or a failed local check); the issues a lint finds are its normal result, not an error.

### Widget Templates (6 tools)

#### `templates_widgets_get`

Get a widget template by ID as its full JSON config, for inspection or to change it before `templates_widgets_save`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Widget template ID (from `templates_widgets_list`) |

**Returns:** The widget template JSON (`GET Templates/WidgetTemplates?id=`), or "Widget template not found".

---

#### `templates_widgets_create`

Create a new widget template from JSON config. Checks for name collisions — if a widget with the same name already exists, the call **fails and returns the existing ID** so you can use `templates_widgets_save` instead.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `config` | string (JSON) | Yes | Widget template config (see required fields below) |

**Required fields:**

| Field | Type | Notes |
|-------|------|-------|
| `name` | string | Template name |
| `widgetType` | `{id: int}` | 0=InfoCard, 1=PieChart, 2=XYChart, 4=Table, 6=TextBlock |
| `dataRangeMode` | `{id: int}` | Data range mode |
| `metricSelectionMode` | `{id: int}` | Metric selection mode |

Optional `discipline` (`{id: "id"}`) files the widget under a discipline in the widget list (the web editor's
"Filed under"); without one it is listed under "No discipline". It doesn't limit the widget's metrics. Save replaces
it, so an update that leaves it out clears it: send back the `discipline` that `templates_widgets_get` returned.

**Additional requirements by widget type:**
- **PieChart** (`widgetType.id: 1`): Also requires `pieChartWidgetTemplateType` with `id`.
  Pie and Donut templates may also use bounded styling fields:
  `styleConfiguration`, `centerNumberFormat`, `dataItems[].presentation`, and
  `dataItems[].numberFormat`. Display fields: `centerMode` (`0` None, `1` Static Text, `2` Metric Value, `3` Total, `5` Percentage),
  `legendValueMode` (`0` Hidden, `1` Value, `2` Percent, `3` Value and Percent), `sliceLabelMode` (`0` Hidden, `1` Label, `2` Value,
  `3` Label and Value, `4` Percent, `5` Label and Percent), `pieShape` (`0` Full Circle, `1` Half Circle), `otherSliceMode`
  (`0` None, `1` Top N, `2` Below Percent) with `otherSliceThreshold` (Top N 2 to 20, Below Percent 1 to 25). Keep visual fields separate from
  `valueSelectionConfig`, which controls filtering/ranking/diagnostics.
- **XYChart** (`widgetType.id: 2`): Data items require `xyChartWidgetTemplateDataItemType` with `id`
- **InfoCard** (`widgetType.id: 0`): Optional `showDataIntervalFilter`, as on a Table: `true` follows the dashboard's interval with `dataInterval` as the default, `false` fixes `dataInterval` (lint error `IntervalFixedNeedsInterval` without one); omitted on create, the card follows exactly when it has no `dataInterval`. Row `dataPeriodStart` and text-token offsets count in the interval the card reads at, unconverted
- **Table** (`widgetType.id: 4`): Also requires `dataGrouping` (`0` None, `1` OrgNode, `2` Discipline, `3` Framework). Rows come from `dataItems` (Static metric selection) or the metric scope filters (Dynamic); optional `additionalDataGrouping` (0–2 Then By levels), `narratives`, `showMetricValue`, `showMetricsInColumns` (needs a grouping, or `showTimePeriodsInRows`), `showTimePeriodsInRows` (one row per period; needs `showMetricsInColumns`), `columnDataGroupingLevels` (grouping levels shown on columns; the layout must keep a row and a column), `metricPropertyColumns`, `metricAttributeTypeIds`, `metricTooltipProperties` and `metricTooltipAttributeTypeIds` (the metric-name tooltip), `missingValuePlaceholder`, `styleConfiguration` and `showDataIntervalFilter` (true follows the dashboard's interval with `dataInterval` as the default; false fixes `dataInterval` and needs one, `IntervalFixedNeedsInterval`; omitted on create follows exactly when `dataInterval` is null). Order and limit (`valueSelectionConfig`) needs OrgNode in the grouping. An update that leaves out the four layout and tooltip fields or `showDataIntervalFilter` keeps their stored values
- **TextBlock** (`widgetType.id: 6`): Requires at least one non-blank `title`, `subtitle`, `description`, or `footnote` (`TextBlockTextRequired`); omit `dataItems` (metrics and narratives are discovered from text tokens). Optional `showDataIntervalFilter`, as on an InfoCard: `true` follows the dashboard's interval with `dataInterval` as the default, `false` fixes `dataInterval`; omitted on create, the text block follows exactly when it has no `dataInterval`. Token errors use the `WIDGET_TOKEN_*` codes, with the slot as `field` and the token as `params.key`
- **All data items** need `metric` (with `id`) and `metricPartitioningMode` (with `id`)

Text fields use stored tokens: `[metric-guid]`, `[metric-guid]|-1|`, `#trend[metric-guid]`, `@[period:n]`, `{narrative-guid}`, and `{narrative-guid}|-1|`; a metric may also be named, `[Metric Name]`, and save stores its id. Narrative tokens are allowed only in TextBlock, InfoCard and Table title, description and footnote (and a TextBlock's subtitle), and resolve at the narrative's own capture interval. See [Widget text tokens](#widget-text-tokens).

**Pie/Donut styling fields:**

| Field | Notes |
|-------|-------|
| `styleConfiguration` | Template-level slots: `panel`, `title`, `description`, `footnote`, `chartArea`, `slices`, `sliceBorders`, `sliceLabels`, `ticks`, `tooltip`, `legend`, `legendMarkers`, `legendLabels`, `legendValues`, `donutCenter`, `otherSlice`, `stateMessages` |
| `centerNumberFormat` | `{ "precision": 0..6, "magnitude": "None\|Thousands\|Millions\|Billions\|Auto" }`; valid for Metric Value and Total donut centers |
| `dataItems[].presentation` | Source data-item non-fill styling slots: `slice`, `sliceBorders`, `sliceLabels`, `tooltip`, `legendMarker`, `legendLabel`, `legendValue`; `dataItems[].colour` remains slice/legend-marker fill authority |
| `dataItems[].numberFormat` | Same number-format shape as `centerNumberFormat`; applies to slice values, legend values, tooltips, MCP metadata, CLI JSON, and accessible text |
| `valueSelectionConfig` | Filtering, ordering, Top/Bottom ranking, and diagnostics only. Style payloads inside `valueSelectionConfig` are rejected. |

Unsupported style payloads such as raw CSS, HTML, callbacks, unsafe URLs,
arbitrary chart config, generic `style` bags, `styleBag`, and `propertyBag` are
rejected by API/MCP validation.

**Returns:** Success message with the new template ID and name, followed by the template's lint warnings when there are any (see [`templates_widgets_lint`](#templates_widgets_lint)). Warnings never block the save.

---

#### `templates_widgets_save`

Update an existing widget template by ID. Same required fields as `templates_widgets_create`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Template ID to update |
| `config` | string (JSON) | Yes | Widget template config with updated fields |

If the config contains an `id` field, it must match the `id` parameter or be the empty ID.

**Returns:** Success message with the updated template ID and name, followed by the template's lint warnings when there are any. Warnings never block the save.

---

#### `templates_widgets_lint`

Check a widget template without saving it. Errors are exactly what saving would reject; warnings and notes never block a save. Each issue carries a code, for example `NameRequired`, `NameDuplicate`, `MetricsRequired`, `NarrativesRequired`, `FrameworkGroupingNeedsMatching`, `OrgNodeAttributeFilterIncomplete` or `UnknownMetrics`. A table saved before these checks that already breaks `MetricsRequired` or `NarrativesRequired` gets a warning instead, so it still saves; a new break, and any other rule, is an error. An info card reports `DataItemDuplicate` (the same metric twice with the same period, sites and combining; a warning when the saved card already repeats it), `UnknownMetrics` and `MetricNotViewable` on a row, `StyleSlotValueInvalid` and `StyleSlotPropertyUnsupported` (with `path` and `allowed`), the warnings `DataItemLabelDuplicate`, `DataItemWindowSinglePeriod` and `TrendIconWithoutLabel`, and the notes `OverrideFieldsCleared` (a filtered card's row keeps only its label, number format and bands) and `TrendRangesGiveNoVerdict` (a `#trend` coloured by direction because its metric's ranges give no verdict; an info card or text block). A pie reports `PieChartTypeRequired`, the centre codes (`PieCenterNeedsDonut`, `PieCenterModeReserved`, `PieCenterFieldsNeedMode`, `PieCenterTextRequired`, `PieCenterFieldNotForMode`, `PieCenterMetricRequired`, `PieCenterMetricNotInPie`, `PieCenterShareAlwaysFull`), `PieOtherThresholdInvalid`, `EnumValueInvalid`, `TimePeriodAggregationRequired` (a warning on a pie already saved without a method), `UnknownMetrics` and `MetricNotViewable` on a slice or `centerMetricId`, and the notes `PieCenterNumberFormatFolded` and `PieSliceBandsFolded` (what save removes). Each code has one English message, the same on every channel.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `config` | string (JSON) | One of the two | The widget template config, as posted to `templates_widgets_save`. Checked locally first, as save does |
| `templateId` | string (ID) | One of the two | A saved widget template to lint as it is stored |

**Returns:** One line per issue, errors first, then warnings, then notes: `Error   [Code] field: message`, `Warning …`, `Note    …`, then a count line; or "No issues found in the widget template." `POST Templates/WidgetTemplates/Lint` (with `templateId`: `GET Templates/WidgetTemplates?id=` first).

---

#### `templates_widgets_preview`

Render an info card, text block, pie chart, XY chart or gauge widget template without saving it, as HTML, the way the dashboard would show it at the given sites and period, with your data permissions: the same figures, units, labels and band colours as `apps_widget_infoCard` shows once it is saved. A draft row that is not on the saved card yet renders too, and a draft's own bands colour the value. A text block renders its resolved title, subtitle, description and footnote as `apps_widget_textBlock` shows them once saved, so you can check commentary before saving; metric names in its text resolve as save stores them. A pie renders as `apps_widget_pieChart` shows it once saved: its slices (a negative slice is left out of the chart, as the dashboard leaves it out of the ring) and its data warnings (`PieNegativeValuesLeftOut`, `PieManySlices`, `PieNoValues`, `PieMixedUnits`), with its Other slice and half-circle shape. An XY chart renders as `apps_widget_xyChart` shows it once saved: its series on their axes, including axes and series the config adds before they have ids, and the render's notes (`XyNoValues` when no series has a value, `LowContrast` for a series colour that is hard to read on white, `ExcessiveSeriesExpansion` over 40 series). An AI Summary on Metrics I pick (`scope` 2) renders its resolved title, how many metrics it reads, and the insights already generated for those facts. The tool never calls the model, so a draft not generated yet in the widget editor says "Not generated". A section-scope AI Summary needs a dashboard section and previews in the widget editor. A gauge renders as `apps_widget_gauge` shows it once saved: its dial, value, range name, distance and off-scale note, at one site (two or more org nodes get the server's 400, "A gauge shows one site"). Only info cards, text blocks, pies, XY charts, AI Summaries and gauges preview here; a table config is refused before the server is asked.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `config` | string (JSON) | Yes | The info card, pie chart, XY chart, AI Summary, text block or gauge config, as posted to `templates_widgets_save` (`widgetType` `{"id":0}`, `{"id":1}`, `{"id":2}`, `{"id":3}`, `{"id":6}` or `{"id":7}`) |
| `orgNodeIds` | string | Yes | Comma-separated org node IDs (from `model_orgNodes_list`) |
| `startDate` | string | Yes | Start of the period (ISO `yyyy-MM-dd`) |
| `endDate` | string | Yes | End of the period (ISO `yyyy-MM-dd`) |
| `timePeriodType` | string | No | `day`, `week`, `month`, `quarter` or `year` (default `month`), the period's type |
| `dataInterval` | string | No | `day`, `week`, `month`, `quarter` or `year`: the dashboard interval. Left out (as `apps_widget_infoCard` sends none), the widget reads at its own interval; a widget that fixes its interval ignores it |

**Returns:** The card, text block, pie, XY chart or gauge as HTML, or the server's refusal: a 403 without Templates.Manage and Reporting.View, a 400 naming each field when the config has an error that stops it rendering (a missing name or period window, or a text block with no text, for example), or a filtered pie whose filters match no metric ("No metrics match this widget's filters."). `POST Templates/WidgetTemplates/Preview`.

---

#### `templates_widgets_delete`

Delete a widget template by ID. Permanently removes the template — dashboards referencing it will lose this widget.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Template ID to delete |

**Returns:** Confirmation message.

---

### Report Templates (3 tools)

#### `templates_reports_create`

Create a new report template from JSON config. Checks for name collisions — if a report with the same name already exists, the call **fails and returns the existing ID** so you can use `templates_reports_save` instead.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `config` | string (JSON) | Yes | Report template config (see required fields below) |

**Required fields:**

| Field | Type | Notes |
|-------|------|-------|
| `name` | string | Template name |
| `dataGrouping` | `{id: "id"}` | Data grouping to use |

Optional: `showDataGroupingFilter` (bool) and `allowedDataGroupingLevels` (list of `{id}`, 0=None (no grouping, a flat list), 1=OrgNode, 2=Discipline, 3=Framework; at least two, including the default's levels; viewers group by them in any order; ignored when the flag is false; allowing 0 needs `showTimePeriodsInRows` for `showMetricsInColumns`), `columnDataGroupingLevels` (list of `{id}`, levels of the grouping placed on columns in chain order: 1=OrgNode for a column block per site, 2=Discipline, 3=Framework; empty means every level is on rows; a level outside the grouping is rejected), `expandCalculations` (bool) and `expandCalculationsMaxDepth` (1..20, default 10). Expansion is rejected when `showMetricsInColumns` is true or any level is on columns. `showMetricsInColumns` needs at least one grouping level (`MetricsInColumnsRequiresGrouping`); each row is then one combination of the grouping values, and metrics that do not apply to a row are marked not applicable. `showTimePeriodsInRows` (bool) lists time periods as rows and metrics as columns in the web viewers; it requires `showMetricsInColumns` (`TimePeriodsInRowsRequiresMetricsInColumns`) and also works with no grouping (`dataGrouping` id 0), which makes a period-by-metric sheet for one site.

`measure` (`{id}`: 0 = Value, the default; 1 = Capture completion; 2 = Validation completion) makes the report count expected values that are captured or validated instead of showing values. A completion template needs inputs and narratives only in `includedDataTypes` (not empty, no calculations), no `metricTypes` filter, periods as columns, every grouping level on rows, the physical org structure, and none of `showMetricsInColumns`, `showTimePeriodsInRows`, `expandCalculations`, `showOnlyRowsWithValues`, `showRowsWithValuesFilter`, `showEvidence`, `showChangeRequests`, `allowChangeRequestsToggle` or an org node template setting; `templates_reports_create` refuses a breach with the rule's message (for example "Capture and validation progress show metrics as rows, not columns."). `completionBands` (optional) colours the cells by share done, 0 to 100: bands as a metric's `bands`, with number ends only (no `lowerBoundMetric`/`upperBoundMetric`, `requireComment` or `autoValidate`), for example `[{"backgroundColor":"danger-subtle"},{"lowerBoundValue":50,"backgroundColor":"success-subtle"}]`. None uses the defaults (red below 40, amber from 40, green from 90); `[]` colours nothing. A breach is refused with the rule's message.

`orgNodes`, `disciplines` and `frameworks` are the Limit to lists (no viewer reaches outside them); `defaultOrgNodes`, `defaultDisciplines` and `defaultFrameworks` are the starting picks and must sit inside the matching Limit to list (`DefaultOutsideLimit`). When rows are grouped without Org Node, or viewers can change the grouping, a fixed site filter (`showOrgNodeFilter` false) must resolve to exactly one org node, the single default or else the single Limit to pick (`SingleOrgNodeRequired`); an open one may have at most one default org node (`SingleDefaultOrgNodeRequired`).

`orgNodeTemplate` (`{id}`, null = physical) is the default org structure; site picks must belong to it (`SiteNotInStructure`). Set `showOrgNodeTemplateFilter: true` to let viewers switch between `orgNodeTemplates` (`[{id}]`; empty = every structure the viewer can access) plus the physical structure when `allowPhysicalStructure` is true (default). The default must be one of them (`DefaultStructureOutsideLimit`). While switchable, `orgNodes` must be empty (`SiteLimitRequiresFixedStructure`), a fixed site filter can't have `defaultOrgNodes` (`FixedSitesRequireFixedStructure`), and rows grouped without Org Node, or a grouping viewers can change, need `showOrgNodeFilter: true` (`SwitchableStructureRequiresOpenSites`). Open-site `defaultOrgNodes` apply to the default structure only; other structures start at their root.

**Returns:** Success message with the new template ID and name, followed by the template's lint warnings when there are any (see [Lint warnings](#report-template-lint-warnings)).

---

#### `templates_reports_save`

Update an existing report template by ID. Same required fields as `templates_reports_create`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Template ID to update |
| `config` | string (JSON) | Yes | Report template config with updated fields |

If the config contains an `id` field, it must match the `id` parameter or be the empty ID.

**Returns:** Success message with the updated template ID and name, followed by the template's lint warnings when there are any.

<a id="report-template-lint-warnings"></a>
**Lint warnings.** After a successful create or save, the tool calls `POST Templates/SpreadsheetReportTemplates/Lint` and appends each warning as `- [Code] field: message` under `Warnings (saved anyway; fix these so the rows show):`. Warnings never block the save. Codes:

| Code | Meaning |
|------|---------|
| `RowTypesExcludeBoundNarratives` | Narratives are bound but Narrative is not in `includedDataTypes` (empty means every type). |
| `RowTypesExcludeBoundCalculations` | Calculation metrics are bound but Calculation is not in `includedDataTypes`. |
| `SiteDefaultMissing` | The site filter is open and single-select with several sites in its Limit to and no default, so viewers start with no site. |
| `IntervalNotSet` | `dataInterval` is empty, so the template shows days. |
| `NarrativesOutsideIntervalOrScope` | Bound narratives no longer match `dataInterval`, the Limit to `orgNodes` or the Limit to `disciplines`. |

If the warning check fails, the result says so and the save still stands. There is no capture template save tool.

---

#### `templates_reports_delete`

Delete a report template by ID. Permanently removes the template.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Template ID to delete |

**Returns:** Confirmation message.

---

### Dashboard Templates (4 tools)

#### `templates_dashboards_get`

Get a dashboard template's full structure including all tree items with their IDs. **Required before modifying a dashboard** — the save operation uses full-replacement semantics.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Dashboard template ID |

**Returns:** JSON with complete dashboard structure:

```json
{
  "id": "dash-id",
  "name": "ESG Executive Dashboard",
  "filedUnder": {"id": "...", "name": "Energy", "path": "Environmental->Energy"},
  "icon": "fa-solid fa-chart-line",
  "backgroundColor": "primary",
  "image": {"type": "image/png", "fileName": "board.png", "data": "<base64, 24 KB or less>"},
  "dashboardStyle": {
    "backgroundColor": "surface-canvas",
    "contentWidth": {"id": 3, "name": "Wide"}
  },
  "treeItems": [
    {
      "id": "item-id",
      "path": "Overview",
      "isWidget": false,
      "nodeLayout": {
        "layoutMode": {"id": 2, "name": "Grid"}
      },
      "nodeStyle": {
        "backgroundColor": "surface-muted"
      }
    },
    {
      "id": "item-id",
      "path": "Overview -> Energy Card",
      "isWidget": true,
      "widgetTemplate": {"id": "wt-id", "name": "Energy Card"},
      "widgetType": {"id": 0, "name": "InfoCard"},
      "sortOrder": 1,
      "widgetSize": {"id": 1},
      "placementLayout": {
        "widthBehavior": {"id": 0, "name": "WidgetSizeDefault"}
      },
      "placementStyle": {
        "shadow": {"id": 2, "name": "Sm"}
      }
    }
  ]
}
```

---

#### `templates_dashboards_create`

Create a new dashboard template from JSON config. Checks for name collisions — if a dashboard with the same name already exists, the call **fails and returns the existing ID** so you can use `templates_dashboards_save` instead.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `config` | string (JSON) | Yes | Dashboard template config (see required fields below) |

**Required fields:**

| Field | Type | Notes |
|-------|------|-------|
| `name` | string | Dashboard name |
| `treeItems` | array | Array of tab and widget items |
| `dashboardStyle` | object | Optional dashboard canvas style |

**Tree item fields:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `path` | string | Yes | Use `->` for nesting (e.g. `"Tab -> Widget"`) |
| `isWidget` | boolean | Yes | `false` for tabs, `true` for widgets |
| `widgetTemplate` | `{id: "id"}` | Widget only | Widget template ID (from `templates_widgets_list`) |
| `widgetType` | `{id: int}` | Widget only | 0=InfoCard, 1=PieChart, 2=XYChart, 3=AISummary, 4=Table, 6=TextBlock. An AI Summary placement points at an AI Summary widget template, at most one per section |
| `sortOrder` | int | No | Display order within tab |
| `widgetSize` | `{id: int}` | No | Widget size |
| `nodeLayout` | object | Structural node only | Layout mode, alignment, spacing, header, and responsive metadata |
| `nodeStyle` | object | Structural node only | Background, foreground, title, header/content, typography, border, and radius metadata |
| `placementLayout` | object | Widget/narrative only | Width behavior, alignment, minimum height, spacing, and responsive metadata |
| `placementStyle` | object | Widget/narrative only | Background, foreground, border, radius, and shadow metadata |
| `callout` | object | Structural node or Narrative placement only | Variant, severity, icon, collapsible state, title/body translations |

Dashboard layout metadata uses the same JSON contract as the CLI and API. `dashboardStyle` styles the dashboard canvas/page area. `nodeLayout` and `nodeStyle` apply to structural nodes. `placementLayout` and `placementStyle` apply to widget or narrative wrappers. `callout` is valid on structural nodes and Narrative placements only.

Safe colors are named tokens, `#rgb`, `#rrggbb`, `#rrggbbaa`, bounded `rgb(...)`/`rgba(...)`, or `var(--token-name)`. Structured lengths are JSON objects with numeric `value` or side values and `unit: {"id": 0, "name": "Px"}` or `unit: {"id": 1, "name": "Rem"}`.

**Returns:** Success message with the new dashboard ID and name.

---

#### `templates_dashboards_save`

Update an existing dashboard template by ID.

> **WARNING — Full-Replacement Semantics:** The save replaces the **entire tree**. Any items omitted from `treeItems` will be **permanently deleted**. Always follow the get-merge-save workflow:
> 1. Call `templates_dashboards_get` to get the current tree
> 2. Merge your changes into the full tree
> 3. Save the **complete** tree with ALL items (including unchanged ones)
>
> **Never call this without first calling `templates_dashboards_get`.**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Dashboard template ID to update |
| `config` | string (JSON) | Yes | **Complete** dashboard template config — must include ALL tree items |

If the config contains an `id` field, it must match the `id` parameter or be the empty ID.

**Returns:** Success message with the updated dashboard ID and name.

---

#### `templates_dashboards_delete`

Delete a dashboard template by ID. Permanently removes the dashboard template and all its tree items.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Dashboard template ID to delete |

**Returns:** Confirmation message.

---

#### `templates_dashboards_lint`

Check a dashboard template without saving it. Errors are exactly what saving would reject, each with a code and the path it is on (for example `treeItems[3].nodeStyle.fontSize.value`). Codes include `NameRequired`, `NameNotUnique` (a new template only), `UnsafeText` (header or callout text holding `<`, `>`, `{`, `}`, `;`, `javascript:`, `expression(` or `url(`; ordinary words such as "description" are fine), `OutOfBounds` (for example a font size outside 8–96px or 0.5–6rem), `InvalidColor`, `UnknownField`, `WidgetTemplateRequired`, the viewer filter codes (`HiddenFilterRequiresDefaultOrgNode`, `DefaultStructureOutsideLimit`, `SwitchableStructureRequiresSitesFilter`, `UnsupportedDefaultDataInterval`, and for the `orgNodes` site limit `SiteLimitRequiresFixedStructure` and `DefaultOutsideLimit`) and the checks against the tenant's data (`WidgetTemplateNotFound`, `UnknownOrgStructure`, `UnknownAllowedStructure`, `DefaultOrgNodeOutsideStructure`, `SiteNotInStructure`). Only the JSON is checked locally, so every code comes from the server.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `config` | string (JSON) | One of the two | The dashboard template config, as posted to `templates_dashboards_save` |
| `templateId` | string (ID) | One of the two | A saved dashboard template to lint as it is stored |

**Returns:** One line per issue, errors first, then warnings, then notes: `Error   [Code] field: message`, then a count line; or "No issues found in the dashboard template." `POST Templates/DashboardTemplates/Lint` (with `templateId`: `GET Templates/DashboardTemplates?id=` first).

---

## Status (2 tools)

### `status`

Check Capstone MCP server status including authentication state and configuration.

No parameters.

**Returns:** Server version, API URL and authentication state (signed-in user, or API key mode).

---

### `version`

Return the Capstone MCP server version.

No parameters.

**Returns:** The version string.

---

## System (4 tools)

> Tenant AI settings. Changing them needs the System.Manage permission. These tools never show or set an API key, and never acknowledge DeepSeek's data location; use the tenant's AI settings page or the CLI for those.

### `system_tenantAiConfig_get`

Show a tenant's AI settings: on or off, whose key calls use (the NxGN default, which is the provider, model and key this deployment sets, or the client's own key), provider, model and key status. The key itself is never shown, only its last four characters.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tenantId` | string (ID) | No | Tenant ID. Defaults to the current tenant |

**Returns:** The AI settings as text (`GET System/Tenants?Id=`, the tenant's `ai`).

---

### `system_tenantAiConfig_set`

Change a tenant's AI settings: turn AI on or off, choose whose key calls use, the client's provider and the model. Settings you leave out keep their current values. The NxGN default always uses the provider this deployment sets, so `provider` is ignored for it. When a change needs a client key or the DeepSeek acknowledgement, the tool returns guidance to use the tenant's AI settings page or the CLI.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tenantId` | string (ID) | No | Tenant ID. Defaults to the current tenant |
| `enabled` | boolean | No | Turn AI on (`true`) or off (`false`) |
| `keySource` | string | No | `nxgn` (the NxGN default) or `client` |
| `provider` | string | No | The client's provider: `anthropic` or `deepseek`. Changing it removes the stored key |
| `model` | string | No | Model ID of the provider in use, or `default` to follow the default |
| `summariseInAllLanguages` | boolean | No | Write narrative AI summaries in every language a narrative has text in (`true`), or the default language only (`false`) |

Pass at least one of `enabled`, `keySource`, `provider`, `model` or `summariseInAllLanguages`.

**Returns:** The saved settings (`POST System/Tenants`, the tenant as read with its `aiSettings`), or the guidance above.

---

### `system_tenantAiUsage_get`

Show a tenant's AI usage: requests, tokens and failures, grouped by day, feature or model.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tenantId` | string (ID) | No | Tenant ID. Defaults to the current tenant |
| `from` | string | No | First day (ISO `yyyy-MM-dd`) |
| `to` | string | No | Last day (ISO `yyyy-MM-dd`), included |
| `groupBy` | string | No | `day` (default), `feature` or `model` |

**Returns:** Totals, then one line per group (`POST System/Tenants/AiConfiguration/Usage`).

### `system_configuration_history_list`

List configuration changes, newest first: who changed which input, calculation, site, discipline, framework node or
template, when, how (Web, Excel upload, CLI, API key, Integration), and the value before and after. Read-only; needs
the Configuration History view permission.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `itemType` | string | No | `Input`, `Calculation`, `OrgNode`, `DisciplineNode`, `FrameworkNode`, `SpreadsheetCaptureTemplate`, `SpreadsheetReportTemplate`, `WidgetTemplate` or `DashboardTemplate` |
| `itemId` | string (ID) | No | Only this item's changes |
| `since` | string | No | First day (ISO `yyyy-MM-dd`, UTC) |
| `until` | string | No | Last day (ISO `yyyy-MM-dd`, UTC), included |
| `limit` | number | No | Rows to return (default 100) |
| `offset` | number | No | Rows to skip (default 0) |

**Returns:** The total, then one line per change: when, who, type, item (with id), action, field, before, after, how
(`POST System/ConfigurationHistory/GetGridList`).

---

## Workflow (1 tool)

### `analysis_workflow`

Get the recommended workflow for Capstone data analysis and presentation. Returns the step-by-step approach for turning raw data into visual, insight-driven output.

No parameters.

**Returns:** Markdown guide with 6-step workflow: Orient → Dashboard Overview → Pull Data → Find Insights → Visualize Each Insight → Narrate.

**Triggering:** The server's `instructions` field directs Claude to call this tool automatically before analytical tasks. No user action needed.

**Steps covered:**
1. **Orient** — Confirm data availability, identify scope
2. **Dashboard Overview** — Render pre-built dashboards as visual starting point
3. **Pull Detailed Data** — Get numbers at multiple granularities
4. **Identify Findings** — Quantify what, why, what-to-do, and where
5. **Visualize Each Finding** — Create focused charts with `apps_chart_render` (critical step)
6. **Narrate** — Lead with dollar impact, reference charts

---

## Common Parameter Patterns

### Time Period Resolution

Data tools support two ways to specify time periods:

| Approach | Parameters | When to Use |
|----------|-----------|-------------|
| **By name** | `timePeriodNames="Jan 2025,Feb 2025"` | When you know exact period names |
| **By type+count** | `periodType="quarter"`, `periodCount=4` | When you want N most recent periods that have started |

Use `data_timePeriods_list` to discover available period names.

**App tools use a different approach:**

| Approach | Parameters | When to Use |
|----------|-----------|-------------|
| **Date range** | `startDate="2024-10-01"`, `endDate="2025-03-31"` | All app/visualization tools |

### Data Interval

The `dataInterval` parameter controls the time granularity. Be aware of the type difference:

| Tool | Type | Format |
|------|------|--------|
| `data_timePeriods_list` | string | `day`, `week`, `month`, `quarter`, `year` |
| `data_availability` | int | Day=0, Week=1, Month=2, Quarter=3, Year=4 |
| `data_inputValues_list` | int | Day=0, Week=1, Month=2, Quarter=3, Year=4 |
| `data_computedValues_list` | int | Day=0, Week=1, Month=2, Quarter=3, Year=4 |
| `data_computedValues_query` | int | Day=0, Week=1, Month=2, Quarter=3, Year=4 |

### ID Parameters

All ID parameters expect standard ID format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

Invalid IDs return an error with a hint to the discovery tool.

---

## See Also

- [SKILL.md](../SKILL.md) — Quick overview and usage patterns
- [Data Model](./data-model.md) — Enum values and aggregation patterns
- [Glossary](./glossary.md) — Term definitions with MCP tools
- [Resources](./resources.md) — Auto-loaded context data
