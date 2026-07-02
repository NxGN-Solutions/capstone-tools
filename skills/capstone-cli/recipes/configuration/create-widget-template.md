# Recipe: Create Widget Template

> Guided workflow for creating widget templates (InfoCard, PieChart, XYChart, TextBlock, Table).
>
> **MCP alternative:** Widget templates can also be created via MCP using the `templates_widgets_create` tool. See [MCP Tool Reference — Templates CRUD](../../../capstone-mcp/reference/tools.md#templates--crud-10-tools).

## When to Use

- "Create a chart for energy consumption"
- "Add an InfoCard to show total emissions"
- "Set up a pie chart for waste breakdown"
- "Create a table/grid for portfolio KPIs"
- "I need a widget to display [metric]"
- "How do I add a widget to a dashboard?"

## Required Context

Before starting, Claude should know:
- [ ] What type of widget (InfoCard, PieChart, XYChart, TextBlock, Table)
- [ ] Which metric(s) to display
- [ ] Which discipline to categorize under

**If missing:** Claude will ask clarifying questions.

---

## Step-by-Step

### Step 1: Understand the Widget Types

**Purpose:** Choose the right visualization for the data.

| Widget Type | Best For | Example |
|-------------|----------|---------|
| **InfoCard** (id: 0) | Single KPI value with optional comparison | "Total Emissions: 1,234 tonnes" |
| **PieChart** (id: 1) | Part-to-whole relationships | "Waste by Type: 40% recycled, 35% landfill, 25% composted" |
| **XYChart** (id: 2) | Time series, comparisons | "Monthly Energy Consumption (Jan-Dec)" |
| **Table** (id: 4) | Metric-backed dashboard tables/grids | "Portfolio KPI matrix by company and period" |
| **TextBlock** (id: 6) | Metric-aware dashboard text with title, subtitle, description, and footnote | "Portfolio summary for Q1 with metric values and trend text" |

**Ask if not provided:**
- "What kind of visualization do you need?"
- "Are you showing a single value, a breakdown, a trend over time, text commentary, or row-and-column detail?"

---

### Step 2: Select the Discipline

**Purpose:** Categorize the widget for organization and permissions.

**Command:**
```bash
cap masterdata disciplines list --json
```

**Get discipline ID** for the widget's discipline field.

**Discipline Assignment Rules:**

1. **Match the primary metric's discipline.** If the widget shows "Total Cost", assign it to the same discipline as the Total Cost metric (e.g., `Finance->Cost`).
2. **Cross-discipline widgets use the perspective's discipline.** A widget showing OEE + Profit together belongs to `Finance` (the financial perspective), not `Production`.
3. **Be consistent with sibling templates.** If `Cost Breakdown | Pie Chart` is at `Finance->Cost`, then `Cost Composition | Pie Chart` (same metrics) should be too.
4. **Use the most specific discipline.** Prefer `Finance->Cost` over `Finance` for cost-specific widgets. Reserve the parent discipline for widgets that genuinely span sub-disciplines (e.g., `Revenue vs Cost` under `Finance` root).

**Include the full discipline path** in the JSON — `name` and `path` help with readability:
```json
"discipline": {
  "id": "<discipline-id>",
  "name": "Cost",
  "path": "Finance->Cost"
}
```

---

### Step 3: Select Metrics for Data Items or Text Tokens

**Purpose:** Define what data the widget displays.

**Command:**
```bash
cap model metrics list --json
```

**For InfoCard:** Usually one primary metric
**For PieChart:** Multiple metrics that form parts of a whole
**For XYChart:** One or more metrics to plot over time
**For Table:** Metrics become row sources; value columns can use row metrics, selected metrics, or calculation metrics
**For TextBlock:** Metrics are discovered from tokens in `title`, `subtitle`, `description`, and `footnote`; do not create hidden data-item rows for those references

**Note metric IDs** for `dataItems[]` configuration or raw CLI text tokens. Widget Template Excel can accept metric-name tokens and transpose them to IDs during upload.

---

### Step 4: Configure Widget Properties

**Required fields for all widget types:**
- `name` - Internal name for the template (follows naming convention, not shown to users)
- `title` - Displayed to dashboard users as the widget heading
- `description` - Optional text shown to dashboard users below the widget content (see User-Facing Fields rules below)
- `widgetType` - { id: 0|1|2|4|6, name: "InfoCard"|"PieChart"|"XYChart"|"Table"|"TextBlock" }
- `discipline` - { id: "<id>" }
- `dataRangeMode` - { id: 1, name: "Static" } or { id: 0, name: "Dynamic" }
- `dataInterval` - { id: 2, name: "Month" }, { id: 3, name: "Quarter" }, etc.
- `metricSelectionMode` - { id: 0, name: "Static" } (explicit metrics) or { id: 1, name: "Dynamic" }
- `timePeriodAggregationMethod` - Required for Dynamic widgets that collapse multiple selected periods into one value. Use Sum for additive metrics, Average for rates/percentages, Last Value for snapshots, and None only for deliberate time-series output.
- `dataItems` - Array of metric configurations for metric-set widgets; omit or set to `[]` for TextBlock

> **Dynamic metric selection:** Set `metricSelectionMode` to
> `{ id: 1, name: "Dynamic" }` to resolve metrics at render time instead of
> listing each metric in `dataItems`. Info Card, Pie Chart, and Table use the
> widget-level filter arrays `metricTypeFilters`,
> `metricDisciplineFilters`, `metricFrameworkFilters`, `metricAttributeFilters`,
> `disciplineAttributeFilters`, and `frameworkAttributeFilters`. The shared DTO
> also accepts legacy/report-template-compatible `disciplineNodeAttributeFilters`
> and `frameworkNodeAttributeFilters` (empty array = select all on that axis).
> Dynamic Table templates must author at least one
> non-empty metric-scope filter family. Info Card and Pie Chart can also sort and
> Top/Bottom-rank a resolved dynamic metric set with a **widget-level**
> `valueSelectionConfig` whose `ownerScope` is `WidgetDynamicMetricSet`. XY Chart
> has no dynamic metric discovery. Table uses Dynamic metric scope for row
> candidates, then uses `ownerScope: "TableRows"` when ranking/filtering resolved
> rows.
> See **Widget Value Selection → Dynamic metric selection** in
> [reference/commands.md](../../reference/commands.md) for the full field reference
> and a copy-pasteable example, or run
> `cap templates widget-templates sample --widget-type pie --json`.

**Required fields for Table widgets:**
- `dataGrouping` - `None`, `OrgNode`, `Discipline`, or `Framework` (Framework requires Dynamic metric selection)
- `orgNodeRowSelectionMode` - Required when `dataGrouping` is `OrgNode`
- Static metric selection needs `dataItems[]` unless metric filters provide the report-template fallback
- Dynamic metric selection uses Metric Scope filters (`metricTypeFilters`, discipline/framework filters, and attribute filters) and clears explicit `dataItems[]` on save

**Authoring fields for Table widgets:**
- `showOnlyRowsWithValues` - Suppress rows whose resolved values are all empty
- `showMetricValue`, `showUnitOfMeasure`, `metricAttributeTypeIds` - Current custom-column controls
- `orgNodeAttributeFilters` - Filter the resolved org-node scope before values are loaded; this is not the deferred `OrgNodeAttribute` grouping mode
- `narrativeSelectionMode` - `Dynamic` for Narrative Scope, or `Static` for explicit `narratives[]`
- `narrativeScopeConfigured` and narrative scope arrays - Use when Dynamic Narrative Scope is intentionally authored
- `narratives` - Used when `narrativeSelectionMode` is Static
- `styleConfiguration` - Bounded Table style slots, row/column overrides, conditional formatting, and categorical color tags

Do not author `includedDataTypes` or `orgNodeTemplateId` for new Table templates. The CLI/API may accept them for legacy compatibility, but current authoring uses Narrative Properties and the dashboard template owns the org-node-template lens.

**Required fields for TextBlock widgets:**
- At least one of `title`, `subtitle`, `description`, or `footnote` - Text slots that support the same metric, trend, and period tokens as other widget text fields
- `dataRangeMode`, `metricSelectionMode`, and `timePeriodAggregationMethod`
- `dataInterval`, `dataPeriodStart`, and `dataPeriodEnd` when `dataRangeMode` is `Static`

TextBlock does not support `dataItems`; metrics are discovered only from explicit text tokens. `styleConfiguration` is optional and uses bounded TextBlock slots for `panel`, `title`, `subtitle`, `description`, `footnote`, `trend`, and `stateMessages`. TextBlock style colors accept safe theme tokens or `#RGB`/`#RRGGBB` hex values, font family uses the shared `theme`/`sans`/`serif`/`mono`/`nunito`/`roboto`/`poppins`/`arial` catalogue, and font weight accepts `Normal`/`Medium`/`Semibold`/`Bold`.

**Data Interval Reference:**
| ID | Name |
|----|------|
| 0 | Day |
| 1 | Week |
| 2 | Month |
| 3 | Quarter |
| 4 | Year |

---

### User-Facing Fields: Subtitle, Description, Footnote, and Display Name

These fields are shown to **dashboard viewers** — they must add value for the person reading the dashboard, not the person building it.

#### `description` (widget-level)

Leave **empty** in most cases. The title, values, and footnote already tell the story. Only add a description when extra context genuinely helps the viewer understand what they're seeing.

| Good | Bad |
|------|-----|
| `""` (empty — widget is self-explanatory) | `"Shows total cost from all sites"` (repeats what the title says) |
| `"Excludes contractor hours"` (useful caveat) | `"For Executive P&L dashboard"` (developer note) |
| | `"Created for the Example demo"` (irrelevant to viewer) |

**Tip:** If you're tempted to add a short interpretive note (e.g., "Positive = Cola earns more per unit than Orange"), use the **footnote** instead — it renders more subtly beneath the widget content.

#### `subtitle` (TextBlock only)

Use a subtitle for a short secondary heading below the title. It supports the same metric, trend, and period tokens as title, description, and footnote. Keep it brief; longer commentary belongs in `description`.

#### `footnote` (widget-level)


| Use Case | Example |
|----------|---------|
| Trend indicator | `#trend[<change-calculation-metric-name>] since @[period:-1]` in the editor/Excel; `#trend[<change-calculation-metric-id>] since @[period:-1]` in raw CLI JSON |
| Interpretive hint | `Positive = Cola earns more per unit than Orange` |
| Data caveat | `Excludes weekends and public holidays` |

#### `displayName` / `name` (data item-level)

Controls the label shown in legends, tooltips, and next to values. The same rule applies across **all widget types** — avoid redundancy with the widget title.

| Scenario | `displayName` | Why |
|----------|--------------|-----|
| **Data item matches the widget title** | `""` or omit | The title already tells the viewer what this is |
| **Single data item** with a different metric name | `""` or omit | The metric name is already visible |
| **Multiple data items** on the same widget | Set a short differentiating label | Viewers need to tell the values apart |
| **Period comparison** (InfoCard, current vs previous) | `"@[period]"` / `"@[period:-1]"` | Tokens resolve to actual period names |
| **Partitioned data item** (Children/Descendants) | `""` or omit | Org node names auto-populate the legend |
| **Table row-source item** | `""`, omit, or use a short author-facing label | Rendered row labels come from row fields such as MetricName or OrgNodeName |

**InfoCard — single data item (no displayName needed):**
```json
"title": "Total Cost",
"dataItems": [
  { "metric": { "id": "...", "name": "Total Cost" }, "sortOrder": 1 }
]
```

**InfoCard — multiple data items (displayName differentiates):**
```json
"title": "Revenue Per Unit",
"dataItems": [
  { "metric": { "id": "...", "name": "Cola Revenue Per Unit" }, "name": "Cola", "sortOrder": 1 },
  { "metric": { "id": "...", "name": "Orange Revenue Per Unit" }, "name": "Orange", "sortOrder": 2 }
]
```

**XY Chart — data item matches title (leave name blank):**
```json
"title": "Cola Price Premium % Trend",
"dataItems": [
  { "metric": { "id": "...", "name": "Cola Price Premium %" }, "name": "", "sortOrder": 1 }
]
```
The legend entry would redundantly show "Cola Price Premium %" under a chart already titled "Cola Price Premium % Trend".

---

### Step 5: Build the JSON

**Purpose:** Construct the widget template definition.

#### InfoCard Example:
```json
{
  "name": "Total Energy Consumption",
  "title": "Energy Usage",
  "widgetType": { "id": 0, "name": "InfoCard" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "dataItems": [
    {
      "metric": { "id": "<metric-id>", "name": "Electricity Consumption" },
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null
    }
  ]
}
```

#### InfoCard with Period Comparison:

This pattern shows two data items (current vs previous period) with a trend indicator footnote. Requires `dataRangeMode: Static` and a dedicated % change calculation metric.

```json
{
  "name": "Profit | Comparison | Daily | Card",
  "title": "Daily Profit",
  "footnote": "#trend[<change-metric-id>] since @[period:-1]",
  "widgetType": { "id": 0, "name": "InfoCard" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 0, "name": "Day" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "timePeriodAggregationMethod": { "id": 0, "name": "None" },
  "styleConfiguration": {
    "panel": {
      "fontFamily": "theme",
      "borderWidth": 0
    },
    "title": {
      "borderWidth": 0
    },
    "trend": {
      "showLabel": false,
      "up": { "color": "success", "icon": "arrow-up" },
      "down": { "color": "danger", "icon": "arrow-down" },
      "flat": { "color": "neutral", "icon": "equals" },
      "unknown": { "color": "unknown", "icon": "none" }
    }
  },
  "dataItems": [
    {
      "metric": { "id": "<metric-id>", "name": "Profit" },
      "displayName": "@[period]",
      "dataPeriodStart": 0,
      "dataPeriodEnd": 0,
      "sortOrder": 1,
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 0, "name": "None" }
    },
    {
      "metric": { "id": "<metric-id>", "name": "Profit" },
      "displayName": "@[period:-1]",
      "dataPeriodStart": -1,
      "dataPeriodEnd": -1,
      "sortOrder": 2,
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 0, "name": "None" }
    }
  ]
}
```

**Key fields:**
- `dataRangeMode: Static` — required for period offsets to work
- `dataPeriodStart` / `dataPeriodEnd` — `0` = last period with data, `-1` = previous period
- `displayName` — use `@[period]` tokens so labels show the actual period name
- `timePeriodAggregationMethod: None` is intentional here because each data item resolves exactly one static period by offset. Do not copy `None` to Dynamic cards that summarize a user-selected range; choose Sum, Average, or Last Value by metric role.
- `styleConfiguration.trend` — controls only visual presentation. It does not define whether a metric is good or bad.
- `styleConfiguration.panel/title/value/...` — controls card-level slot styling. Font families are allowlisted token IDs, widths/sizes are bounded integers, and `accentSide`/`accentWidth` are panel-only. Partial border/accent settings are retained but render as no border/accent until the color/side/positive width combination is complete.

For reusable KPI, period-comparison, alert/exceedance, tinted metric,
left-accent, and full-bleed style fragments, use
[Configure Info Card Styles](./configure-info-card-styles.md). That recipe is
the CLI-first guide for translating dashboard mockups into
`styleConfiguration` JSON.

Raw CLI JSON posts directly to the API, so metric tokens inside `title`, `subtitle`, `description`, `footnote`, and data item `name` should use stored metric IDs. The widget editor and Widget Template Excel accept metric names and transpose them to IDs internally.

**Naming convention:** `{Metric} | Comparison | {Interval} | Card`

#### PieChart Example:
```json
{
  "name": "Waste Breakdown by Type",
  "title": "Waste Distribution",
  "widgetType": { "id": 1, "name": "PieChart" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 0, "name": "Dynamic" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "showLegend": true,
  "pieChartWidgetTemplateType": { "id": 1, "name": "Donut Chart" },
  "centerMode": { "id": 2, "name": "Metric Value" },
  "centerStaticText": null,
  "centerMetricId": "<total-waste-metric-id>",
  "centerMetricLabel": null,
  "legendValueMode": { "id": 1, "name": "Value" },
  "sliceLabelMode": { "id": 3, "name": "Label and Value" },
  "dataItems": [
    {
      "metric": { "id": "<metric-id>", "name": "Recycled Waste" },
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "showInLegend": true,
      "showTooltip": true,
      "sortOrder": 1
    },
    {
      "metric": { "id": "<metric-id>", "name": "Landfill Waste" },
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "showInLegend": true,
      "showTooltip": true,
      "sortOrder": 2
    },
    {
      "metric": { "id": "<metric-id>", "name": "Composted Waste" },
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "showInLegend": true,
      "showTooltip": true,
      "sortOrder": 3
    }
  ]
}
```

Pie and donut charts collapse the selected range into one slice value per data item, so a Dynamic pie must set a non-None time aggregation method. Keep `showInLegend` true on each data item when `showLegend` is true.

**Pie/donut display fields:**

| Field | Use |
|-------|-----|
| `pieChartWidgetTemplateType` | `0` Pie Chart or `1` Donut Chart. |
| `centerMode` | Donut-only center content: `0` None, `1` Static Text, or `2` Metric Value. Standard Pie templates must omit center fields or use `None`. |
| `centerStaticText` | Required only when `centerMode.id` is `1`. |
| `centerMetricId` | Required only when `centerMode.id` is `2`; references an existing Input or Calculation metric. In the dashboard editor, this is selected through the searchable metric grid picker. |
| `centerMetricLabel` | Optional display-label override for Metric Value centers. If blank, rendering uses the center metric friendly name, then the metric name. |
| `legendValueMode` | `0` Hidden or `1` Value. This is the stored enum behind the UI's **Show Legend Value** checkbox; `showLegend` and data-item `showInLegend` still control row visibility. |
| `sliceLabelMode` | `0` Hidden, `1` Label, `2` Value, or `3` Label and Value for labels rendered on slices. This is separate from tooltips and legend rows. |

The same fields round-trip through widget-template Excel download/upload as `Pie Chart Type`, `Center Mode`, `Center Static Text`, `Center Metric Id`, `Center Metric`, `Center Metric Label`, `Legend Value Mode`, and `Slice Label Mode`. On upload, set `Center Metric Id` for a stable metric reference or `Center Metric` for an exact-name lookup; if both are present they must resolve to the same metric. Legacy workbooks that omit these columns keep the default behavior: no center, legend values visible, and slice labels hidden.

For styled Pie/Donut templates, use `styleConfiguration`, `centerNumberFormat`,
`dataItems[].presentation`, and `dataItems[].numberFormat`. These are bounded
JSON contracts shared by the API, Excel, CLI, MCP, and browser validator. Keep
them separate from `valueSelectionConfig`, which remains the ranking/filtering
contract. Per-slice fill belongs in `dataItems[].presentation.slice.backgroundColor`;
legacy workbook `Colour` values are import-only aliases into that slot. See
[Configure Pie Chart Styles](./configure-pie-chart-styles.md) for copy-pasteable
style fragments and unsupported-payload boundaries.

Use the live schema before constructing styled Pie JSON:

```bash
cap templates widget-templates schema --widget-type pie --json
cap templates widget-templates sample --widget-type pie --json
```

The sample's editable body is under `.payload.widgetTemplate`. Pie/Donut
`centerMode`, `legendValueMode`, and `sliceLabelMode` enum values are under
`schema.fieldLookups`; they may not be available through standalone
`templates lookups get` names.

Use the typed render command to verify the live dashboard contract:

```bash
cap reporting widgets pie-chart <widget-template-id> \
  --org-nodes "<org-node-id>" \
  --periods "Jan 2026" \
  --json
```

JSON output is a top-level render contract and includes resolved `center`,
`labelDisplay`, `legend`, `styleConfiguration`, `dataItems[].presentation`,
`dataItems[].numberFormat`, `dataItems[].legendValue`, and
`dataItems[].sliceLabel` metadata. Non-JSON output prints
`Center: <label>: <value>` when configured and includes `Legend` and `Slice
Label` columns for quick smoke checks.

#### XYChart Example:

> **Important:** XY Charts require an `axes` array, and every data item must include `xyChartWidgetTemplateAxis` with a `name` matching an axis entry. The API matches by name (not ID) during creation.

```json
{
  "name": "Monthly Energy Trend",
  "title": "Energy Over Time",
  "widgetType": { "id": 2, "name": "XYChart" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 0, "name": "Dynamic" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "showLegend": true,
  "invertAxes": false,
  "styleConfiguration": {
    "version": "1",
    "panel": {
      "backgroundColor": "surface",
      "borderColor": "neutral",
      "borderWidth": 1,
      "padding": 16
    },
    "title": {
      "fontSize": 18,
      "fontWeight": "Bold",
      "foregroundColor": "text-primary"
    },
    "categoryAxis": {
      "labelRotation": 45,
      "labelDensity": "Comfortable"
    },
    "series": {
      "strokeWidth": 3,
      "dashArray": "Solid"
    },
    "marker": {
      "shape": "Circle",
      "size": 6
    },
    "dataLabel": {
      "displayMode": "Auto"
    },
    "numberFormat": {
      "axisValue": {
        "precision": 1,
        "magnitude": "Auto"
      },
      "tooltipValue": {
        "precision": 1,
        "magnitude": "Auto"
      },
      "legendValue": {
        "precision": 1,
        "magnitude": "Auto"
      },
      "dataLabel": {
        "precision": 1,
        "magnitude": "Auto"
      }
    }
  },
  "axes": [
    { "name": "kWh", "dynamicAxis": true }
  ],
  "dataItems": [
    {
      "metric": { "id": "<metric-id>", "name": "Electricity Consumption" },
      "xyChartWidgetTemplateDataItemType": { "id": 2, "name": "Line" },
      "xyChartWidgetTemplateAxis": { "name": "kWh", "dynamicAxis": true },
      "presentation": {
        "series": {
          "color": "#1565C0",
          "strokeWidth": 2
        }
      },
      "showBullets": true,
      "showInLegend": true,
      "showTooltip": true,
      "isStacked": false,
      "timePeriodAggregationMethod": { "id": 0, "name": "None" },
      "sortOrder": 1
    }
  ],
  "timePeriodAggregationMethod": { "id": 0, "name": "None" }
}
```

`None` is correct for this XY example because it is a time series and should render one point per selected month. For an aggregated XY summary, set both the widget and each data item to Sum, Average, Last Value, Min, or Max.

**XY style fields:**

| Area | Notes |
|------|-------|
| `styleConfiguration.version` | Use `"1"` for the bounded XY style contract. |
| Slot objects | Supported slots include `panel`, `title`, `description`, `footnote`, `chartArea`, `plotArea`, `categoryAxis`, `valueAxis`, `axisTitle`, `axisLabel`, `gridline`, `series`, `marker`, `dataLabel`, `tooltip`, `legend`, `legendMarker`, `legendLabel`, `legendValue`, `stateMessages`, and `numberFormat`. |
| Axis controls | `categoryAxis`/`valueAxis` accept `labelRotation` (-90..90), `labelDensity` (`Auto`/`Comfortable`/`Compact`), `minLabelGap` (10..300px, overrides density), and value-axis `startAtZero` (`true` = zero baseline, `false` = auto-range). |
| Safety bounds | The CLI and API reject raw CSS, HTML, JavaScript, callbacks, URL/CSS functions, unknown slots, unknown properties, and out-of-range sizes. Errors include `styleConfiguration.<field>` paths. |
| Data-item presentation | Per-series visual overrides live in `dataItems[].presentation.series`; use `color`, `strokeWidth`, `dashArray`, `strokeDashArray`, border fields, radius, and opacity there. |
| Behavioral fields | `showBullets`, `showInLegend`, `showTooltip`, stacking, series type, and axis assignment stay as data-item behavior fields. |
| Excel parity | Widget-template Excel exports chart-wide JSON through `XY Chart Style` and per-series JSON through data-item `Presentation`. Legacy workbook `Rotate Category Axis Labels`, `Colour`, `Stroke Width`, and `Stroke Dash Array` columns are import-only aliases mapped into style/presentation slots. |

Use the schema and sample commands to inspect the full contract:

```bash
cap templates widget-templates schema --widget-type xy --json
cap templates widget-templates sample --widget-type xy --json
```

Use the typed reporting command to inspect resolved render metadata:

```bash
cap reporting widgets xy-chart <widget-template-id> \
  --org-nodes "<org-node-id>" \
  --data-interval Month \
  --periods "Jan 2026" \
  --json
```

The JSON output includes `styleMetadata`, `styleDiagnostics`, `displayMetadata`, `accessibilityMetadata`, and `styleDefaultsApplied` alongside the existing axes, series, diagnostics, warnings, and no-data fields. MCP exposes the same metadata and emits structured diagnostics for Chart.js-inapplicable primitives.

> **`styleDiagnostics` are author guidance.** Low-contrast warnings, native-field
> precedence notes, and renderer adjustments are always returned in the render
> contract (for the CLI, MCP, the template style editor, and audit). In the
> browser they surface **only in the widget-template style editor**, never on
> live dashboards — end-users never see styling guidance. Use these CLI fields
> while authoring to validate a style (e.g. raise series-colour contrast to clear
> a `LowContrast` warning); the published dashboard renders cleanly regardless.

#### Table Example:

Table widgets are the dashboard table widget type. They use normal Input and Calculation metrics as row sources and can render dynamic or static narrative rows.

```json
{
  "name": "Ambient Noise | Site Matrix | Quarterly | Table",
  "title": "Ambient Noise by Site",
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
      "metric": { "id": "<metric-id>", "name": "Ambient Noise Day Time" },
      "sortOrder": 0,
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null
    }
  ],
  "metricAttributeTypeIds": ["<metric-attribute-type-id>"],
  "narrativeSelectionMode": { "id": 0, "name": "Dynamic" },
  "narrativeScopeConfigured": true,
  "narrativeDisciplineFilters": [
    { "id": "<discipline-id>", "name": "Safety" }
  ],
  "narrativeFrameworkFilters": [],
  "narrativeMetricAttributeFilters": [],
  "narrativeDisciplineNodeAttributeFilters": [],
  "narrativeFrameworkNodeAttributeFilters": [],
  "narrativeAttributeFilters": [],
  "narratives": [],
  "styleConfiguration": {
    "rowLabelHeader": "Site",
    "zebraStripeColor": "surface",
    "gridHeader": {
      "foregroundColor": "text-primary",
      "fontWeight": "Bold",
      "textTransform": "Uppercase",
      "letterSpacing": 1
    },
    "valueCells": { "textAlign": "End" },
    "conditionalFormatRules": [
      {
        "configuredColumnId": "metric:<metric-guid-n>",
        "operator": "Negative",
        "style": { "foregroundColor": "danger" }
      }
    ]
  }
}
```

**Key fields:**
- `dataItems` are the static metric selections. Use dynamic metric filters instead when `metricSelectionMode` is Dynamic.
- `dataGrouping` and `orgNodeRowSelectionMode` control row grouping under the dashboard-selected org node.
- `showMetricsInColumns` switches OrgNode-grouped tables from one row per metric to metric columns.
- `showMetricValue`, `showUnitOfMeasure`, and `metricAttributeTypeIds` control custom columns.
- `narrativeSelectionMode: Dynamic` uses the narrative scope arrays. Empty arrays mean all permitted values when `narrativeScopeConfigured` is true.
- `narrativeSelectionMode: Static` ignores narrative scope arrays and renders the explicit `narratives[]` list.
- `styleConfiguration` controls the semantic HTML table renderer. Keep it sparse and use safe tokens or validated hex values.
- Arbitrary static-text columns, org-node-attribute display columns, expression columns, and `OrgNodeAttribute` row grouping are deferred; do not add a standalone `columns[]` array to current Table JSON.

**Naming convention:** `{Subject} | {Qualifier} | {Interval} | Table`

---

#### TextBlock Example:

TextBlock widgets render metric-aware dashboard text without chart, card-value, or table rows. Metrics are discovered only from explicit text tokens in `title`, `subtitle`, `description`, and `footnote`.

```json
{
  "name": "Portfolio Status | Monthly | TextBlock",
  "title": "Portfolio Status",
  "subtitle": "Performance for @[period]",
  "description": "Total emissions were [<metric-id>] for @[period].",
  "footnote": "#trend[<change-metric-id>] since @[period:-1]",
  "widgetType": { "id": 6, "name": "TextBlock" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "dataPeriodStart": -1,
  "dataPeriodEnd": 0,
  "styleConfiguration": {
    "panel": {
      "backgroundColor": "surface",
      "borderColor": "border",
      "borderWidth": 1,
      "padding": 16
    },
    "title": {
      "fontSize": 20,
      "fontWeight": "Bold",
      "foregroundColor": "text-primary"
    },
    "subtitle": {
      "fontSize": 14,
      "foregroundColor": "text-secondary"
    },
    "description": {
      "fontSize": 16,
      "lineHeight": 1.4,
      "foregroundColor": "text-primary"
    },
    "footnote": {
      "fontSize": 12,
      "foregroundColor": "text-muted"
    }
  }
}
```

Widget-template Excel roundtrips TextBlock text through `Title`, `Subtitle`, `Description`, and `Footnote`, and style JSON through `Text Block Style`. In Excel, metric-name tokens are accepted in the text slots and transposed to metric IDs on upload.

Use the typed render command to inspect resolved text, spans, style, warnings, and diagnostics:

```bash
cap reporting widgets text-block <widget-template-id> \
  --org-nodes "<org-node-id>" \
  --data-interval month \
  --periods "Jan 2026" \
  --json
```

**Naming convention:** `{Subject} | {Interval} | TextBlock`

---

#### Value Selection (Ranking & filters)

`valueSelectionConfig` adds ordering, Top/Bottom ranking, and value/null
filtering. It can sit at widget level for an explicit static metric set
(`widgetTemplate.valueSelectionConfig`, `ownerScope: "WidgetDynamicMetricSet"`,
Info Card, Pie Chart, and XY Chart), on a partitioned data item
(`dataItems[].valueSelectionConfig`, `ownerScope: "DataItem"`), on a dynamic
metric set (`ownerScope: "WidgetDynamicMetricSet"`, Info Card and Pie Chart
only), or on resolved Table rows (`ownerScope: "TableRows"`). All enum values are
**bare strings**, and the config rejects unknown members. Do not combine
widget-level static metric-set selection with data-item-level selection on the
same static chart/card.

| Field | Type / values | Rule |
|-------|---------------|------|
| `orderMode` | `None`, `ValueAscending`, `ValueDescending` | |
| `rankMode` | `None`, `Top`, `Bottom` | |
| `rankLimit` | integer `1`..`1000` (MaxPageSize) | Required when `rankMode` is `Top`/`Bottom`; omit otherwise |
| `nullPlacement` | `Last`, `DefaultLast` | `First` is rejected (engine default) |
| `predicates` | array, max 5 | Filters applied before ranking |
| `diagnosticsLabel` | string (optional) | Free-text label echoed in widget selection diagnostics |
| `boundsWarningMode` | `Default`, `Warn`, `Suppress` (optional) | Advisory warnings when projected element count is large |

Each predicate is `{ predicateType, operator, operandKind?, lowerValue?, upperValue?, metricId?, metricStableKey?, tolerance? }`:

- `predicateType`: `Value` or `Null`.
- `Value` predicates use `operator` ∈ `GT`, `GTE`, `LT`, `LTE`, `EQ`, `NE`, `Between` and `operandKind` `Number` (default) or `Metric`.
  - `Number` operand: `lowerValue` required (plus `upperValue` for `Between`).
  - `Metric` operand: `metricId` **or** `metricStableKey` required; cannot use `Between`; cannot carry numeric values; not supported in `WidgetDynamicMetricSet` selection.
- `Null` predicates use `operator` `IsNull` or `IsNotNull` and carry no operand fields.
- `tolerance` (optional) must be ≥ 0.

Per-data-item example (Top 5 by sum, drop empty values):

```json
"valueSelectionConfig": {
  "ownerScope": "DataItem",
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
    { "predicateType": "Null", "operator": "IsNotNull" }
  ],
  "diagnosticsLabel": "Top 5 data-item values",
  "boundsWarningMode": "Warn"
}
```

Table row example (Top 5 rows by a metric column):

```json
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
  "predicates": []
}
```

For `TableRows`, only `TableMetricAggregate` and `TableRowAggregate` selector
kinds are valid; `tableColumnKey` uses the stored `metric:<guid>` form (the
dashless `N` GUID format) and must not be set for `TableRowAggregate`. Run
`cap templates widget-templates sample --widget-type info --json` for a working
data-item `valueSelectionConfig` example. See
[Commands Reference → Widget Value Selection](../../reference/commands.md#widget-value-selection)
for the full field/selector matrix.

---

### Step 6: Create the Widget Template

**Purpose:** Execute the creation command.

**Command (auto-wrapped JSON):**
```bash
echo '{
  "name": "Total Energy Consumption",
  "widgetType": { "id": 0, "name": "InfoCard" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "dataItems": [
    {
      "metric": { "id": "<metric-id>", "name": "Total Energy Consumption" },
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "sortOrder": 1
    }
  ]
}' | cap templates widget-templates create --json
```

> **Note:** The CLI auto-wraps JSON in `{"widgetTemplate": {...}}` if the root object has `name` or `widgetType` properties. You don't need to add the wrapper yourself.

**Or with explicit wrapper:**
```bash
echo '{"widgetTemplate": {...}}' | cap templates widget-templates create --json
```

**What to look for:**
- Success response with new widget template ID
- Error handling for validation failures

---

### Step 7: Verify Creation and Data

**Purpose:** Confirm the widget template was created correctly and returns data.

**Command — check structure:**
```bash
cap templates widget-templates get <new-id> --json
```

**Command — verify data flows (recommended):**
```bash
# get-data auto-detects the data interval from the template
cap reporting widgets get-data <new-id> \
  --org-node <org-node-id> \
  --periods "2026-01-20,2026-01-21,2026-01-22" \
  --json
```

> **Tip:** `widgets get-data` is the fastest way to verify a widget has data — it reads the template's data interval automatically, so you only need `--org-node` and `--periods`. See [Verify Data](../reporting/verify-data.md) for the full verification workflow.

**Command — verify Table dashboard render shape:**
```bash
cap reporting widgets table <new-id> \
  --org-nodes <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 26" \
  --json
```

Use the type-specific Table command when you need to validate `timePeriodColumns`, `metricColumns`, `metricNumberFormats`, `gridRows`, `metadataColumns`, `selectionDiagnostics`, `styleConfiguration`, `warnings`, `totalCount`, and paging. Use `widgets get-data` when you need the legacy CSV compatibility payload.

**Present confirmation:**
```
Widget template created and verified!

ID:           <id>
Name:         Total Energy Consumption
Widget Type:  InfoCard
Discipline:   Environmental > Energy
Data Items:   1 metric(s) configured
Data:         Values confirmed for 3 periods

Next steps:
- Add to dashboard template
```

---

## Example Interaction

**User:** "Create an InfoCard to show our total electricity usage"

**Claude:** I'll help you create an InfoCard widget template. First, let me find the discipline and metric.

```bash
cap masterdata disciplines list --json
```
→ Found "Energy" discipline under Environmental: `<id>`

```bash
cap model metrics list --json | grep -i electricity
```
→ Found "Electricity Consumption" metric: `<id>`

**Claude:** Here's the widget I'll create:

```
Name:         Total Electricity Usage
Widget Type:  InfoCard
Discipline:   Environmental > Energy
Metric:       Electricity Consumption (Sum)
Data Interval: Monthly
```

Proceed?

**User:** "Yes"

**Claude:**
```bash
echo '{
  "name": "Total Electricity Usage",
  "title": "Electricity Usage",
  "widgetType": { "id": 0, "name": "InfoCard" },
  "discipline": { "id": "<id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "dataItems": [
    {
      "metric": { "id": "<id>", "name": "Electricity Consumption" },
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "sortOrder": 1
    }
  ]
}' | cap templates widget-templates create --json
```

Widget template created! ID: `<id>`

You can view widget data using:
```bash
cap reporting widgets info-card <id> \
  --org-nodes <org-node-id> \
  --data-interval month \
  --periods "Jan 2026" \
  --json
```

---

## Variations

### Creating from File

For complex templates, save JSON to a file:
```bash
cap templates widget-templates create --file widget.json --json
```

### Updating an Existing Template

Use `save` instead of `create` (ID required):
```bash
cap templates widget-templates save --file widget.json --json
```

**Important:** When using auto-wrap with updates, the `id` must be in the inner widget object:
```json
{
    "id": "<id>",
    "name": "Updated Widget Name",
    "widgetType": { "id": 0, "name": "InfoCard" },
    ...
}
```

The CLI will auto-wrap this to `{"widgetTemplate": {...}}`, preserving the ID inside.

### Get → Modify → Save Round-Trip

The most practical workflow for updating templates: get the full JSON, modify fields, and save back.

```bash
# 1. Export current template to file
cap templates widget-templates get <id> --json > template.json

# 2. Edit the file (modify title, data items, etc.)
# The output format includes { "tenant": "...", "widgetTemplate": { ... } }
# The save command recognizes the "widgetTemplate" wrapper automatically.

# 3. Save back
cap templates widget-templates save --file template.json --json
```

This is useful for:
- Bulk modifications via scripting (loop over templates)
- Backing up templates before changes
- Copying a template as a starting point for a new one (change the `id` to zero ID)

### Listing Available Lookups

Get valid enum values:
```bash
cap templates lookups get widget-types --json
cap templates lookups get data-range-modes --json
cap templates lookups get metric-selection-modes --json
cap templates lookups get timePeriodAggregationMethod --json
```

---

## JSON Structure Reference

### Time Period Aggregation Methods
| ID | Name | Use Case |
|----|------|----------|
| 0 | None | Show each period separately (time series) |
| 1 | Sum | Totals (consumption, hours) |
| 2 | Average | Rates, percentages |
| 3 | Last Value | Current state, point-in-time snapshots |
| 4 | Min | Minimum values |
| 5 | Max | Peak values |

See [Widget Time Period Aggregation](../../reference/widget-time-aggregation.md) for widget-level and per-data-item guidance.

### Data Range Modes
| ID | Name | Description |
|----|------|-------------|
| 0 | Dynamic | Relative to current date |
| 1 | Static | Fixed date range |

### Metric Selection Modes
| ID | Name | Description |
|----|------|-------------|
| 0 | Static | Explicitly defined metrics |
| 1 | Dynamic | Filter-based metric selection |

---

## Widget Sizing Guidelines

Choose widget sizes based on the content type and how they appear on different screen sizes:

| Widget Type | Recommended Size | Rationale |
|-------------|-----------------|-----------|
| **InfoCard** | 50% (id: 2) | Two cards per row; readable on smaller monitors without horizontal scroll |
| **PieChart** | 33% (id: 1) | Compact — pairs well alongside other widgets |
| **XYChart (daily time series)** | 100% (id: 5) | Daily data has many data points; full width prevents label crowding |
| **XYChart (site comparison)** | 50% (id: 2) | Fewer data points; two charts side-by-side enables visual comparison |
| **Table** | 100% (id: 5) | Dense rows/columns usually need full dashboard width |
| **AI Summary** | 100% (id: 5) | Always full width; placed last in a section |

### XY Chart Best Practices

- Configure dense category labels with `styleConfiguration.categoryAxis.labelRotation`, `labelDensity`, `minLabelGap`, or `labelMaxWidth`.
- For horizontal Bar charts, apply the same readability settings to the category axis; the renderer maps the configured category-axis style to the displayed category labels.
- Use `showLegend: true` when multiple series are plotted on the same chart.

---

## Related Recipes

- [Discover Tenant Structure](../exploration/discover-tenant-structure.md) — Find disciplines and metrics
- [Verify Data](../reporting/verify-data.md) — Confirm widget returns expected data
- [Talk to Your Dashboard](../exploration/talk-to-your-dashboard.md) — View widget data
- [Create Metric Wizard](./create-metric.md) — Create metrics for widgets
