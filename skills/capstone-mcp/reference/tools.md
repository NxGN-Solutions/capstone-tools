# MCP Tool Reference

> Complete reference for all Capstone MCP tools with parameters, response formats, and usage examples.

---

## Authentication (5 tools)

### `Login`

Login to Capstone using OAuth. Opens browser for Microsoft/Google SSO authentication.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `provider` | string | No | OAuth provider: `microsoft` (default) or `google` |

**Returns:** Login status message with authenticated user identity.

---

### `Logout`

Logout from Capstone. Clears stored authentication tokens.

No parameters.

**Returns:** Confirmation message.

---

### `Whoami`

Get current user identity, tenant context, roles, and accessible templates.

No parameters.

**Returns:** User identity, tenant name, roles, and configured templates.

**Response example:**
```json
{
  "userName": "user@example.com",
  "tenantName": "Example Organization",
  "tenantId": "<id>",
  "roles": ["Admin", "DataManager"],
  "language": "en"
}
```

---

### `ListTenants`

List available tenants accessible to the current user.

No parameters.

**Returns:** List of tenant names and IDs, with current tenant marked.

**Response example:**
```json
{
  "tenants": [
    { "id": "<id>", "name": "Example Organization", "isCurrent": true },
    { "id": "<id>", "name": "Example Test Tenant", "isCurrent": false }
  ]
}
```

---

### `SwitchTenant`

Switch to a different tenant. Changes become your default for future sessions.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tenantId` | string (ID) | Yes | Tenant ID to switch to |

**Returns:** Confirmation with new tenant name. Invalidates cached resources.

---

## Model Read (5 tools)

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

### `model_disciplines_list`

List disciplines (metric categories like Environmental, Social, Governance).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Filter by name (case-insensitive substring match) |
| `limit` | int | No | Maximum results to return (default: 100) |

**Returns:** Tree grid of discipline nodes with names and hierarchy.

---

## Data Operations (6 tools)

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

Query raw input data (manually entered or imported values) with validation status and lock state. Returns a compact pivot table: one row per metric×org, time periods as columns. Use a capture template ID (from `templates_spreadsheetCaptures_list`) OR pass `templateJson` for ad-hoc queries. For calculated/aggregated results, use `data_computedValues_list` instead.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `templateId` | string (ID) | Conditional | Capture template ID (from `templates_spreadsheetCaptures_list`). Optional if `templateJson` is provided. |
| `templateJson` | string (JSON) | Conditional | Inline capture template JSON (alternative to `templateId`). Minimum: `{"name":"Ad-hoc","dataGrouping":{"id":1},"spreadsheetTemplateMetrics":[{"metric":{"id":"METRIC_GUID"},"sortOrder":0}]}`. `dataGrouping` id: 0=None, 1=OrgNode, 2=Discipline. Get metric IDs from `model_metrics_list`. |
| `timePeriodNames` | string | No | Comma-separated period names (e.g., `Jan 2025,Feb 2025`) |
| `periodType` | string | No | `month`, `quarter`, or `year` — alternative to `timePeriodNames` |
| `periodCount` | int | No | Number of recent periods (default: 4, used with `periodType`) |
| `dataInterval` | int | No | Day=0, Week=1, Month=2 (default), Quarter=3, Year=4 |
| `orgNodeIds` | string | No | Comma-separated org node IDs |
| `disciplineNodeIds` | string | No | Comma-separated discipline IDs |
| `frameworkNodeIds` | string | No | Comma-separated framework IDs |
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

**TimePeriodType values:**

| ID | Name |
|----|------|
| 0 | Day |
| 1 | Week |
| 2 | Month |
| 3 | Quarter |
| 4 | Year |

**Returns:** Save result with success/failure status.

---

### `data_computedValues_list`

**Primary data analysis tool.** Returns a compact pivot table: one row per metric×org, time periods as columns. Use a report template ID (from `templates_spreadsheetReports_list`) OR pass `templateJson` for ad-hoc queries. Set `dataInterval` for granularity: 0=day, 1=week, 2=month, 3=quarter, 4=year. Use for monthly summaries, weekly trends, daily drill-downs, cross-site comparisons. Returns read-only calculated results.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `templateId` | string (ID) | Conditional | Report template ID (from `templates_spreadsheetReports_list`). Optional if `templateJson` is provided. |
| `templateJson` | string (JSON) | Conditional | Inline report template JSON (alternative to `templateId`). Minimum: `{"name":"Ad-hoc","dataGrouping":{"id":1},"spreadsheetTemplateMetrics":[{"metric":{"id":"METRIC_GUID"},"sortOrder":0}]}`. `dataGrouping` id: 0=None, 1=OrgNode, 2=Discipline. Get metric IDs from `model_metrics_list`. |
| `timePeriodNames` | string | No | Comma-separated period names (e.g., `Jan 2025,Feb 2025`) |
| `periodType` | string | No | `month`, `quarter`, or `year` — alternative to `timePeriodNames` |
| `periodCount` | int | No | Number of recent periods (default: 4, used with `periodType`) |
| `dataInterval` | int | No | Day=0, Week=1, Month=2 (default), Quarter=3, Year=4 |
| `orgNodeIds` | string | No | Comma-separated org node IDs |
| `disciplineNodeIds` | string | No | Comma-separated discipline IDs |
| `frameworkNodeIds` | string | No | Comma-separated framework IDs |
| `metricNames` | string | No | Include only metrics matching these names (comma-separated, case-insensitive substring match) |
| `excludeMetricNames` | string | No | Exclude metrics matching these names (comma-separated, case-insensitive substring match, e.g. `Change %`) |
| `includeMetricDetails` | bool | No | Include metric formula, aggregation, and discipline info section (default: false) |
| `limit` | int | No | Maximum pivot rows to return (default: 500) |
| `offset` | int | No | Number of pivot rows to skip for pagination (default: 0) |

> **Time period precedence:** If both `timePeriodNames` and `periodType`/`periodCount` are provided, `timePeriodNames` takes priority. If neither is provided, defaults to the 4 most recent months.

**Returns:** Grid of computed values with metric name, period, and formatted value.

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

> **Note:** Computed values are read-only calculated results. Use `data_inputValues_save` to modify source data.

---

### `data_computedValues_query`

**Ad-hoc computed values query.** Query specific metrics by ID without needing a report template. Use this when you know which metrics you want — constructs an inline template and calls the same API endpoint as `data_computedValues_list`.

For template-based queries where a report template already defines the metric set, use `data_computedValues_list` instead.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `metricIds` | string | Yes | Comma-separated metric IDs to query |
| `orgNodeIds` | string | No | Comma-separated org node IDs. When multiple are provided, results are grouped by org node. |
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

## Reporting (2 tools)

### `reporting_dashboards_getData`

Get all widget data from a dashboard as CSV. Returns data at each widget's native interval (daily widgets produce daily rows) — you cannot control aggregation. For monthly/quarterly summaries or custom granularity, use `data_computedValues_list` with `dataInterval` instead. Best for: full dashboard export, cross-widget correlation.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `dashboardTemplateId` | string (ID) | Yes | Dashboard template ID (from `templates_dashboards_list`) |
| `orgNodeId` | string (ID) | Yes | Organization node ID |
| `timePeriodNames` | string | No | Comma-separated period names |
| `periodType` | string | No | `month`, `quarter` (default), or `year` |
| `periodCount` | int | No | Number of recent periods (default: 4) |
| `scope` | string | No | `peers` or `descendants` (default) |

**Returns:** CSV data for all widgets on the dashboard. Use for cross-widget correlation analysis.

---

### `reporting_widgets_getData`

Get one widget's data as CSV. Use a widget template ID (from `templates_widgets_list`) OR pass `templateJson` for ad-hoc queries. Returns at the widget's configured interval — you cannot change aggregation. For flexible interval control (daily/weekly/monthly), use `data_computedValues_list` instead.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `widgetTemplateId` | string (ID) | Conditional | Widget template ID (from `templates_widgets_list`). Optional if `templateJson` is provided. |
| `templateJson` | string (JSON) | Conditional | Inline widget template JSON (alternative to `widgetTemplateId`). Required fields: `name`, `discipline` (`{"id":"ID"}`), `widgetType` (`{"id":0}` InfoCard, 1 PieChart, 2 XYChart, 4 Table), `dataRangeMode` (`{"id":0}`), `metricSelectionMode` (`{"id":0}`), `dataItems` array. Table templates also require `rowFields` and `valueFields`; each Table value column is an explicit configured value field using row metric, selected metric, or selected calculation binding. Get metric/discipline IDs from `model_metrics_list` and `model_disciplines_list`. |
| `orgNodeId` | string (ID) | No | Organization node ID (from `model_orgNodes_list`) |
| `timePeriodNames` | string | No | Comma-separated period names (e.g., `Q1 FY 25,Q2 FY 25`) |
| `periodType` | string | No | `month`, `quarter`, or `year` — alternative to `timePeriodNames` |
| `periodCount` | int | No | Number of recent periods (default: 4, used with `periodType`) |

**Returns:** CSV data for the widget's metrics and time periods.

---

## Templates — Discovery (6 tools)

### `templates_dashboards_list`

List dashboard templates (name + ID). Use the ID with: `reporting_dashboards_getData` for CSV data export, `apps_dashboard_render` for visual HTML rendering. Dashboard names indicate their content (e.g. 'Executive P&L Overview', 'Production Performance').

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Filter by name (case-insensitive substring match) |
| `limit` | int | No | Maximum results to return (default: 100) |

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
| `search` | string | No | Filter by name (case-insensitive substring match) |
| `limit` | int | No | Maximum results to return (default: 100) |

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

### `templates_spreadsheetReports_list`

List available report templates (define which metrics appear in computed value queries).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Filter by name (case-insensitive substring match) |
| `limit` | int | No | Maximum results to return (default: 100) |

**Returns:** Grid of report templates with name, description, and ID.

---

### `templates_spreadsheetCaptures_list`

List available capture templates (define which metrics appear in data entry forms).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Filter by name (case-insensitive substring match) |
| `limit` | int | No | Maximum results to return (default: 100) |

**Returns:** Grid of capture templates with name, description, and ID.

---

### `templates_spreadsheetReports_get`

Get detailed configuration of a specific report template.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `templateId` | string (ID) | Yes | Report template ID |

**Returns:** Template configuration including filters (discipline, framework, metric type), org node assignments, and included metrics.

---

### `templates_spreadsheetCaptures_get`

Get detailed configuration of a specific capture template.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `templateId` | string (ID) | Yes | Capture template ID |

**Returns:** Template configuration including assigned metrics, org nodes, and data entry settings.

---

## MCP Apps — Visualization (6 tools)

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

**Returns:** HTML with Chart.js visualizations rendered in Claude's conversation.

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

Render a KPI info card as HTML showing metric values with labels, units, and trend indicators. Use when the user wants to SEE a metric card. For raw numbers, use `reporting_widgets_getData` or `data_computedValues_list` instead. Uses `startDate`/`endDate` (ISO yyyy-MM-dd).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `widgetTemplateId` | string (ID) | Yes | Widget template ID (InfoCard type) |
| `orgNodeIds` | string | Yes | Comma-separated org node IDs |
| `startDate` | string | Yes | Start date (ISO format) |
| `endDate` | string | Yes | End date (ISO format) |
| `timePeriodType` | string | No | Period type (default: `month`) |

**Returns:** HTML info card with metric values, labels, and units.

---

### `apps_widget_pieChart`

Render a pie/donut chart as interactive HTML (Chart.js). Shows part-to-whole breakdowns (e.g. cost structure, revenue mix). Use when the user wants to SEE a chart. For raw numbers, use `data_computedValues_list` instead. Uses `startDate`/`endDate` (ISO yyyy-MM-dd).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `widgetTemplateId` | string (ID) | Yes | Widget template ID (PieChart type) |
| `orgNodeIds` | string | Yes | Comma-separated org node IDs |
| `startDate` | string | Yes | Start date (ISO format) |
| `endDate` | string | Yes | End date (ISO format) |
| `timePeriodType` | string | No | Period type (default: `month`) |

**Returns:** HTML with Chart.js pie/donut chart.

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

**Returns:** HTML with Chart.js bar/line/column chart.

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

**Returns:** HTML table with row headers, value columns, warnings, and truncation state.

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

## Templates — CRUD (10 tools)

> Template CRUD tools create, update, and delete widget, report, and dashboard templates. For listing/discovery of existing templates, see the [Templates](#templates-6-tools) section above.

### Widget Templates (3 tools)

#### `templates_widgets_create`

Create a new widget template from JSON config. Checks for name collisions — if a widget with the same name already exists, the call **fails and returns the existing ID** so you can use `templates_widgets_save` instead.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `config` | string (JSON) | Yes | Widget template config (see required fields below) |

**Required fields:**

| Field | Type | Notes |
|-------|------|-------|
| `name` | string | Template name |
| `discipline` | `{id: "id"}` | Discipline category |
| `widgetType` | `{id: int}` | 0=InfoCard, 1=PieChart, 2=XYChart, 4=Table |
| `dataRangeMode` | `{id: int}` | Data range mode |
| `metricSelectionMode` | `{id: int}` | Metric selection mode |

**Additional requirements by widget type:**
- **PieChart** (`widgetType.id: 1`): Also requires `pieChartWidgetTemplateType` with `id`
- **XYChart** (`widgetType.id: 2`): Data items require `xyChartWidgetTemplateDataItemType` with `id`
- **Table** (`widgetType.id: 4`): Also requires `rowFields` and `valueFields`; configured period or selected metric/calculation columns are represented as separate value fields
- **All data items** need `metric` (with `id`) and `metricPartitioningMode` (with `id`)

**Returns:** Success message with the new template ID and name.

---

#### `templates_widgets_save`

Update an existing widget template by ID. Same required fields as `templates_widgets_create`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Template ID to update |
| `config` | string (JSON) | Yes | Widget template config with updated fields |

If the config contains an `id` field, it must match the `id` parameter or be the empty ID.

**Returns:** Success message with the updated template ID and name.

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

**Returns:** Success message with the new template ID and name.

---

#### `templates_reports_save`

Update an existing report template by ID. Same required fields as `templates_reports_create`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string (ID) | Yes | Template ID to update |
| `config` | string (JSON) | Yes | Report template config with updated fields |

If the config contains an `id` field, it must match the `id` parameter or be the empty ID.

**Returns:** Success message with the updated template ID and name.

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
  "treeItems": [
    {
      "id": "item-id",
      "path": "Overview",
      "isWidget": false
    },
    {
      "id": "item-id",
      "path": "Overview -> Energy Card",
      "isWidget": true,
      "widgetTemplate": {"id": "wt-id", "name": "Energy Card"},
      "widgetType": {"id": 0, "name": "InfoCard"},
      "sortOrder": 1,
      "widgetSize": {"id": 1}
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

**Tree item fields:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `path` | string | Yes | Use `->` for nesting (e.g. `"Tab -> Widget"`) |
| `isWidget` | boolean | Yes | `false` for tabs, `true` for widgets |
| `widgetTemplate` | `{id: "id"}` | Widget only | Widget template ID (from `templates_widgets_list`) |
| `widgetType` | `{id: int}` | Widget only | 0=InfoCard, 1=PieChart, 2=XYChart, 4=Table |
| `sortOrder` | int | No | Display order within tab |
| `widgetSize` | `{id: int}` | No | Widget size |
| `aiSummaryContext` | string | No | Context for AI summary generation |

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

## Status (1 tool)

### `GetStatus`

Check Capstone MCP server status including authentication state and configuration.

No parameters.

**Returns:** Server status including API URL and authentication state.

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
| **By type+count** | `periodType="quarter"`, `periodCount=4` | When you want N most recent periods |

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
