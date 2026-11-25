# Recipe: Create Widget Template

> Guided workflow for creating widget templates (InfoCard, PieChart, XYChart, TextBlock, Table).
>
> **Visual standard:** when styling any widget, take the `styleConfiguration`
> variant from the
> [Dashboard Design System](../../reference/dashboard-design-system.md) (type
> ramp, chrome, brand slots, number-format matrix) rather than composing styles
> from scratch. Leave accent fields (`accentColor`, `accentSide`,
> `accentWidth`) unset on every widget type unless the user asks for accents.
>
> **MCP alternative:** Widget templates can also be created via MCP using the `templates_widgets_create` tool. See [MCP Tool Reference — Templates CRUD](../../../mcp/reference/tools.md#templates--crud-10-tools).

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
**For TextBlock:** Metrics and narratives are discovered from tokens in `title`, `subtitle`, `description`, and `footnote`; do not create hidden data-item rows for those references

**Note metric IDs** for `dataItems[]` configuration. Text tokens in raw CLI JSON take a metric ID or the metric's name (`[<metric-id>]` or `[Diesel Consumption]`), in TextBlock and InfoCard text: `lint`, `save` and `preview` resolve names to IDs, and `save` stores the IDs. A name that more than one metric uses is refused with `WIDGET_TOKEN_METRIC_REFERENCE_AMBIGUOUS`; use the ID then. Widget Template Excel takes metric names only.

**Narrative tokens (TextBlock, InfoCard and Table text only):** find narrative IDs with `cap model narratives list --json`. Raw CLI JSON uses `{<narrative-id>}` or `{<narrative-id>}|-1|`; the editor and Excel use `{Narrative Name}`. The token resolves at the narrative's own capture interval, whatever the widget's data interval.

---

### Step 4: Configure Widget Properties

**Required fields for all widget types:**
- `name` - Internal name for the template (follows naming convention, not shown to users)
- `title` - Displayed to dashboard users as the widget heading
- `description` - Optional text shown to dashboard users below the widget content (see User-Facing Fields rules below)
- `widgetType` - { id: 0|1|2|4|6, name: "InfoCard"|"PieChart"|"XYChart"|"Table"|"TextBlock" }
- `discipline` - { id: "<id>" }
- `dataRangeMode` - { id: 1, name: "Static" } or { id: 0, name: "Dynamic" }. XY charts and tables also take { id: 2, name: "SelectedYear" }: the periods of the tenant's year that holds the viewer's last selected period, from the year's start up to that period (FY 27 picked on a quarterly chart: Q1 to Q4; Q2 FY 27 picked: Q1 and Q2). XY charts also take { id: 3, name: "SelectedYearToLatestData" }, which ends at the latest period in which a series has a value; rows with the Reference line role don't count. Neither uses `dataPeriodStart` or `dataPeriodEnd`. Other widget types refuse them (`DataRangeModeNotSupported`). The window only picks periods: every value is the stored computed value of its period. For one year-so-far figure, read the metric at the Year interval or model a to-date calculation such as `SUM([Metric]|YEARSTART():NOW()|)`
- `dataInterval` - { id: 2, name: "Month" }, { id: 3, name: "Quarter" }, etc.
- `showDataIntervalFilter` (every widget type) - `true` follows the dashboard's interval, with `dataInterval` as the default when the dashboard sends none; `false` fixes `dataInterval` (lint error `IntervalFixedNeedsInterval` without one). Left out on create, the widget follows exactly when it has no `dataInterval`; left out on update, it keeps its saved value (clearing `dataInterval` makes it follow). Every period offset of a following widget counts in the interval the dashboard shows: the widget's window, each data item's (`dataItems[].dataPeriodStart`/`dataPeriodEnd`, an XY series' comparison shift included) and text-token offsets. A following card with a `Quarter` default on a Month dashboard reads `-1` as the previous month. When an offset must mean a fixed span of time ("a year earlier"), fix the interval. Excel column: `Table Show Data Interval Filter`, on every widget type's rows
- `metricSelectionMode` - { id: 0, name: "Static" } (explicit metrics) or { id: 1, name: "Dynamic" }
- `timePeriodAggregationMethod` - Required for Dynamic widgets that collapse multiple selected periods into one value. Use Sum for additive metrics, Average for rates/percentages, Last Value for snapshots, and None only for deliberate time-series output. Tables don't use it for their cells (each value is the stored value at the interval, rolled up with the metric's own time period aggregation); the web table editor has no control for it, so Save clears it on a table (note `SettingRetiredCleared`).
- `dataItems` - Array of metric configurations for metric-set widgets; omit or set to `[]` for TextBlock
- `countByReading` (InfoCard) - Default `false`. `true` counts how many of the card's metrics' values fall in a Good, Watch or Bad range (each metric's own ranges, read from their colour) instead of showing each value. `goodLabel`, `watchLabel` and `badLabel` (up to 100 characters each, `TextTooLong` beyond) name the readings on the card; empty shows Good, Watch or Bad. The render contract adds `readingCounts`: `good`, `watch` and `bad` hold indexes into `dataItems` (the "of N" is their total), and `notCounted` lists each other item as `item` (its index) and `reason` (1 no value, 2 not reported at the site, 3 no access, 4 no range reading). Excel columns: `Info Card Count By Reading`, `Info Card Good Label`, `Info Card Watch Label`, `Info Card Bad Label`
- `showConditionalFormatting` - Default `true`. Hard gate: when false, Table / Info Card / XY never paint band colours, even if a dashboard viewer has switched formatting on. The dashboard template's `allowConditionalFormattingToggle` decides whether viewers see the filter-row toggle. On a Table or Info Card, the widget's own `allowConditionalFormattingToggle: false` fixes formatting at `showConditionalFormatting` whatever the viewer chooses; XY ignores it. Spreadsheet/report templates still have their own Allow flag for that page.

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
- `dataGrouping` - `None`, `OrgNode`, `Discipline`, or `Framework` (Framework anywhere in the chain requires Dynamic metric selection)
- `additionalDataGrouping` - optional Then By levels (0–2 of OrgNode/Discipline/Framework, distinct from `dataGrouping`)
- `orgNodeRowSelectionMode` - Required when the grouping chain contains `OrgNode`. `Children` renders the dashboard org node's direct children and `Descendants` its whole subtree; when the dashboard org node has no children, it renders as the single row.
- Static metric selection needs `dataItems[]` unless metric filters provide the report-template fallback
- Dynamic metric selection (All matching) uses Metric Scope filters (discipline/framework filters and attribute filters; a table takes its metric types from `includedDataTypes` and ignores `metricTypeFilters`). Static selection (Only these) is a picked list: it needs at least one item and ignores every filter. Its `dataItems[]` are kept as overrides of the matching metrics they name (label, `numberFormat`, `bands`): they neither limit nor order the metrics, which keep model order
- `dataPeriodStart` / `dataPeriodEnd` when `dataRangeMode` is `Static`, in either metric selection mode: one window for the whole table, counted in `dataInterval` from the dashboard's last selected period (`-11` and `0` = the last 12 periods). Lint error `PeriodWindowRequired` when missing. The window ends at the dashboard's last selected period (`dataPeriodEnd` `0`): Save moves a window that ends earlier or later to end there, keeping its length (note `SettingRetiredMapped`). Table `dataItems[]` take no window: Save clears offsets they carry (note `SettingRetiredCleared`) and `get` returns none. In Excel the window is on the widget row; an older workbook that has windows on a Static table's metric rows and none on its widget row gets as many periods as its metric rows span, ending at the last selected period. Otherwise windows on metric rows are ignored, with a `TableItemWindowIgnored` warning at each such cell; a widget-row window always wins

**Authoring fields for Table widgets:**
- `showOnlyRowsWithValues` - Suppress rows whose resolved values are all empty
- `showMetricValue`, `metricPropertyColumns`, `metricAttributeTypeIds` - Current custom-column controls. `metricPropertyColumns` is an EnumDTO list of `MetricProperty` values (`0` Unit of Measure, `1` Precision, …); widget default is `[]`
- `showMetricsInColumns` - One column per metric instead of one row per metric. Works with any grouping; a table with no grouping needs `showTimePeriodsInRows` too
- `showTimePeriodsInRows` - One row per time period under each row, with the metrics as columns. Needs `showMetricsInColumns`
- `columnDataGroupingLevels` - EnumDTO list of grouping-chain levels (`1` OrgNode, `2` Discipline, `3` Framework) shown as column groups instead of rows. Empty (the default) puts every level on rows. Every level must be in the chain, and the layout must keep at least one row and one column (lint codes `ColumnDataGroupingLevelOutsideChain`, `LayoutRequiresRow`, `LayoutRequiresColumn`)
- `metricTooltipProperties`, `metricTooltipAttributeTypeIds` - Metric properties (same EnumDTO list as `metricPropertyColumns`) and metric attribute-type GUIDs shown in the tooltip on metric names
- `showTotalRow` - Engine-sourced footer for the dashboard org node's calculated values. Only rendered when the grouping chain is OrgNode-only, on rows, with `showMetricsInColumns` and periods as columns; otherwise the render returns `TABLE_TOTAL_UNAVAILABLE` and lint warns `TotalRowNeedsSiteOnly`
- `pageSize` - Rows per page: the page a dashboard viewer starts on: 10, 25, 50 or 100, the sizes the editor offers (viewers can still pick another size on the page toolbar). Save stores another whole number from 1 to 1000 as the nearest of those, a tie taking the smaller (note `SettingRetiredMapped`); outside 1 to 1000 it is `PageSizeOutOfRange` (in Excel, text or a fraction is an upload error on that cell); `50` (or a blank Excel cell) is the default, stored as none. Like the layout fields, an update or a workbook that leaves it out keeps the stored value. The CLI drops a JSON `null` before it posts, so `null` counts as left out and keeps the stored value: send `50` to go back to the default. A table data response's `pageSize` says how many (a request without `take` gets every row; a preview request without `take` gets this many); `cap reporting widgets table` without `--take` and `widget-templates preview` page through every row. Excel column: `Table Rows Per Page`
- `heightMode`, `maxHeightPx` - How tall the table's grid is on a dashboard. `heightMode` `{ "id": 0 }` Fixed (the default) grows with the rows up to `maxHeightPx` and then scrolls the rows inside the table; `{ "id": 1 }` FitRows is as tall as the page's rows, with no scrollbar inside the table (the page scrolls instead, and the page size, `pageSize` or the viewer's pick on the page toolbar, bounds how many rows a page holds). `maxHeightPx` is a whole number from 200 to 2000 (`MaxHeightOutOfRange` otherwise, checked only for Fixed: FitRows ignores it); `600` (or a blank Excel cell) is the default, stored as none. An unknown `heightMode` is `UnknownHeightMode`. Like `pageSize`, an update or a workbook that leaves either out keeps the stored value, and a JSON `null` sent through the CLI counts as left out: to go back to the default send `{ "id": 0 }` and `600` (or blank cells in Excel). A table data response's `maxHeightPx` is the height the grid stops at, or `null` for FitRows. PDF and Word exports always show every row. Excel columns: `Table Height Mode` (`Fixed` or `FitRows`) and `Table Max Height Px`
- `defaultExpandedLevels` - Open levels: how many levels of row groups a dashboard viewer starts with open, on a table whose rows are grouped (for example sites over their metrics). Group headings at a level below the number start open and deeper ones start closed: `0` closes every group, `1` opens the top level only. Viewers open and close groups with the caret on each group heading; their choice is not saved. A whole number from 0 to 10 (`DefaultExpandedLevelsOutOfRange` otherwise; in Excel, text or a fraction is an upload error on that cell); `10` (or a blank Excel cell) is the default, every group open, stored as none. Unlike a report template's `defaultExpandedLevels`, where `null` opens everything, here `10` does: the CLI drops a JSON `null` (it counts as left out), so send `10` to go back to every group open. Like `pageSize`, an update or a workbook that leaves it out keeps the stored value. It has no effect on a table without row groups, and PDF and Word exports always print every row. A table data response's `defaultExpandedLevels` is the stored number, or `null` when every group starts open. Excel column: `Table Default Expanded Levels`
- `showDataIntervalFilter` - `true` follows the dashboard's interval, with `dataInterval` as the default when the dashboard sends none; `false` fixes `dataInterval`, and the table's footer shows "Interval fixed: Quarter" when the dashboard asks for another interval. Fixed needs a `dataInterval` (lint error `IntervalFixedNeedsInterval`). Left out on create, a table follows exactly when it has no `dataInterval`. A following table's Static window counts in the interval the dashboard shows: `-3`..`0` is the last four months on a Month dashboard and the last four quarters on a Quarter dashboard
- `valueSelectionConfig` (order and limit) - Needs OrgNode on rows: in the grouping and not in `columnDataGroupingLevels`, since with sites on columns a row has no site to rank (`ValueSelectionNeedsSiteGrouping`: an error on a new table, a warning on a table saved before this rule)
- An update that leaves out `showTimePeriodsInRows`, `columnDataGroupingLevels`, `metricTooltipProperties`, `metricTooltipAttributeTypeIds` or `showDataIntervalFilter` keeps their saved values; a workbook without their columns does too. Leaving out `showDataIntervalFilter` while clearing `dataInterval` makes the table follow the dashboard, and while giving a table that had no `dataInterval` one fixes it, as setting and clearing the interval did before the flag
- `orgNodeAttributeFilters` - Filter the resolved org-node scope before values are loaded; this is not the deferred `OrgNodeAttribute` grouping mode
- `narrativeSelectionMode` - `Dynamic` for Narrative Scope, or `Static` for explicit `narratives[]`
- `narrativeScopeConfigured` and narrative scope arrays - Use when Dynamic Narrative Scope is intentionally authored
- `narratives` - Used when `narrativeSelectionMode` is Static. Each entry wraps the narrative: `"narratives": [{ "narrative": { "id": "<narrative-id>" }, "sortOrder": 0 }]`; a flat `{ "id": ..., "name": ... }` entry is refused with `NarrativeItemNarrativeRequired`
- `styleConfiguration` - Bounded Table style slots, row/column overrides, and categorical color tags
- `dataItems[].bands` - Optional metric-band override for a listed metric, or, under Dynamic metric selection, for the matching metric the item names. Omit or `null` inherits the metric default (each org node's effective bands: its input or calculation override's bands, else the metric's); `[]` disables formatting for that item. A ladder list has an unbounded first band; later bands use exactly one of `lowerBoundValue` or `lowerBoundMetric`. Interval lists may set both ends, including `upperBoundValue` / `upperBoundMetric`. Colours are registry tokens only (`success-subtle`, `warning-subtle`, `danger-subtle`, `info-subtle`, or other canonical tokens)

Do not author `includedDataTypes` or `orgNodeTemplateId` for new Table templates. The CLI/API accept them for legacy compatibility (Save clears `orgNodeTemplateId`, note `SettingRetiredCleared`), but current authoring uses the narrative fields (`narrativeSelectionMode`, `narratives[]`, Narrative Scope) and the dashboard template owns the org-node-template lens.

**Required fields for TextBlock widgets:**
- At least one of `title`, `subtitle`, `description`, or `footnote` - Text slots that support the same metric, trend, and period tokens as other widget text fields, plus narrative tokens
- `dataRangeMode`. A text block has no period total: Save clears `timePeriodAggregationMethod` (note `SettingRetiredCleared`), and `metricSelectionMode` is not read
- `dataInterval`, `dataPeriodStart`, and `dataPeriodEnd` when `dataRangeMode` is `Static`; the window ends at `0`, and Save moves one that ends elsewhere to end there, keeping its length
- `showDataIntervalFilter` (optional) - as for InfoCard: `true` follows the dashboard's interval with `dataInterval` as the default, `false` fixes `dataInterval`. Left out on create, a text block follows exactly when it has no `dataInterval`

TextBlock does not support `dataItems`; metrics and narratives are discovered only from explicit text tokens. `styleConfiguration` is optional and uses bounded TextBlock slots for `panel`, `title`, `subtitle`, `description`, `footnote`, `trend`, and `stateMessages`. TextBlock style colors accept registry tokens from `cap meta lookups get color-tokens` or `#RGB`/`#RRGGBB`/`#RRGGBBAA` hex values, font family uses the shared `theme`/`sans`/`serif`/`mono`/`nunito`/`roboto`/`poppins`/`arial` catalogue, and font weight is `Normal` or `Bold` (Save stores `Medium` as `Normal` and `Semibold` as `Bold`, as the app's fonts draw them).

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

Use a subtitle for a short secondary heading below the title. It supports the same metric, trend, period, and narrative tokens as title, description, and footnote. Keep it brief; longer commentary belongs in `description`.

#### `footnote` (widget-level)


| Use Case | Example |
|----------|---------|
| Trend indicator | `#trend[<change-calculation-metric-name>] since @[period:-1]` in the editor, Excel and raw CLI JSON, or `#trend[<change-calculation-metric-id>]` in raw CLI JSON. `#trend` takes a % change calculation metric: its value's sign sets the arrow, so a plain total shows a trend arrow for any non-zero value |
| Narrative text (TextBlock/InfoCard/Table) | `{<narrative-name>}` in the editor/Excel; `{<narrative-id>}` in raw CLI JSON. Renders the narrative's text for the selected org unit; `#trend` is not supported |
| Selection label (TextBlock/InfoCard/Table) | `For @[periods] at @[orgNode]` → "For Q1–Q3 FY 26 at Operation 2". `@[periods]` labels the whole selection (rolled up and collapsed into a range); `@[interval]` shows the interval, e.g. "Quarterly" |
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
| **Partitioned data item** (Children/Descendants) | `""` or omit | Org node names auto-populate the legend. At a leaf org node, the partition shows the selected node itself. |
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
  "showDataIntervalFilter": false,
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
      "unknown": { "color": "neutral", "icon": "circle" }
    }
  },
  "dataItems": [
    {
      "metric": { "id": "<metric-id>", "name": "Profit" },
      "name": "@[period]",
      "dataPeriodStart": 0,
      "dataPeriodEnd": 0,
      "sortOrder": 1,
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 0, "name": "None" }
    },
    {
      "metric": { "id": "<metric-id>", "name": "Profit" },
      "name": "@[period:-1]",
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
- `showDataIntervalFilter: false` — fixes the card at `dataInterval`. Offsets count in the interval the card reads at, so a comparison card should fix its interval
- `dataPeriodStart` / `dataPeriodEnd` — `0` = last period with data, `-1` = previous period. Set both to the same offset: a card reads one period per row (its start)
- `name` — use `@[period]` tokens so labels show the actual period name
- `timePeriodAggregationMethod: None` is intentional here because each data item resolves exactly one static period by offset. Do not copy `None` to Dynamic cards that summarize a user-selected range; choose Sum, Average, or Last Value by metric role.
- `styleConfiguration.trend` — controls only visual presentation. It does not define whether a metric is good or bad.
- `styleConfiguration.panel/title/value/...` — controls card-level slot styling. Font families are allowlisted token IDs, widths/sizes are bounded integers, and `accentSide`/`accentWidth` are panel-only. Partial border/accent settings are retained but render as no border/accent until the color/side/positive width combination is complete.

For reusable KPI, period-comparison, alert/exceedance, tinted metric,
left-accent, and full-bleed style fragments, use
[Configure Info Card Styles](./configure-info-card-styles.md). That recipe is
the CLI-first guide for translating dashboard mockups into
`styleConfiguration` JSON.

Raw CLI JSON posts directly to the API. Metric tokens inside `title`, `description`, `footnote`, and data item `name` take a metric ID or the metric's name; `save` stores IDs. The widget editor and Widget Template Excel also accept metric names and transpose them to IDs. InfoCard and Table `title`, `description`, and `footnote` also accept narrative tokens (`{<narrative-id>}`); data item `name` does not.

**Naming convention:** `{Metric} | Comparison | {Interval} | Card`

#### InfoCard with Filter Metrics

With `metricSelectionMode` Dynamic the filters decide which metrics the card shows, in model order. `dataItems[]` then keep per-metric changes as overrides, as on a Table: an item changes the label, `numberFormat` and `bands` of the matched metric with the same `metric.id`. An override never adds a metric; one whose metric the filters don't match shows nothing. A matched metric without an override paints its metric's own bands. The card's own window, combining and value selection apply to every matched metric, so `save` clears an override's period offsets, site breakdown, value selection and its own combining, with the `OverrideFieldsCleared` note. `cap templates widget-templates resolve-content --file widget.json` lists the metrics the filters match, with their ids.

#### InfoCard Lint

Besides the shared codes (`NameRequired`, `DataItemsRequired`, `NumberFormatInvalid`, the `Band*` codes, `DASHSELECT_*`, `WIDGET_TOKEN_*`, the `StyleSlot*` codes in [Configure Info Card Styles](./configure-info-card-styles.md)), an info card reports these. Each code has one English text, the same in the web editor, `lint`, `save`, Excel and MCP; row issues carry the row's `index` in `params`:

| Code | Severity | Message |
|------|----------|---------|
| `DataItemDuplicate` | Error | `{metric} is on this card twice with the same period, sites and combining. Remove one, or change its period.` A card saved with the same repeat gets a warning instead, so it still saves; a new repeat is an error |
| `DataItemLabelDuplicate` | Warning | `Two metrics on this card are labelled "{label}"; readers can't tell them apart.` A label that is empty or equals the metric's name counts as the metric's friendly name (else its name), as the card shows it; rows split by site are skipped |
| `DataItemWindowSinglePeriod` | Warning | `{metric} shows only period {start}: with One period, a card reads one period per metric.` A picked row under Static range with `dataPeriodEnd` ≠ `dataPeriodStart` |
| `TrendIconWithoutLabel` | Warning | `The {direction} trend has no icon and no label, so only its colour shows it. Pick an icon or turn on Show direction label.` |
| `TrendRangesGiveNoVerdict` | Note | `#trend on {metric} is coloured by direction, not by its ranges: none of its ranges reads good, watch or bad.` On the text field that holds the token, when `styleConfiguration.trend.colorBy` is not `Direction` and the metric's own ranges give no verdict; a text block gets it too. See [Configure Info Card Styles](./configure-info-card-styles.md) |
| `OverrideFieldsCleared` | Note | `{metric} is matched by the filters, so its own period and sites were cleared; its label, format and ranges stay.` |
| `UnknownMetrics` | Error | `These metrics don't exist: {ids}.` A row whose metric is not in the tenant |
| `MetricNotViewable` | Error | `This widget refers to a metric you can't view: {metric}.` A row whose metric's discipline you can't view |
| `IntervalFixedNeedsInterval` | Error | `Choose the interval this widget fixes, or let it follow the dashboard.` |

A metric token in `title`, `description`, `footnote` or a row label reports the shared token codes (see [TextBlock token lint](#textblock-token-lint)): `WIDGET_TOKEN_METRIC_REFERENCE_INVALID` for a bracket that is no metric id or name (a name with spaces included), `WIDGET_TOKEN_METRIC_REFERENCE_MISSING` for a deleted metric and `WIDGET_TOKEN_METRIC_REFERENCE_INACCESSIBLE` for one you can't view. They replace the retired `INFOCARD_TOKEN_METRIC_REFERENCE_INVALID`, `INFOCARD_METRIC_REFERENCE_MISSING` and `INFOCARD_METRIC_REFERENCE_INACCESSIBLE`. Ranges (`bands`) on a filtered card's items are allowed (`BandsRequireStaticSelection` applies to XY only).

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

Pie and donut charts collapse the selected range into one slice value per data item, so a pie must say how its slices combine periods: a widget `timePeriodAggregationMethod` (Sum, Average, Last Value, Min or Max), or one on every data item, which Save moves onto the pie (the most common one, note `SettingRetiredMapped` when they differ) and clears from the slices. A Dynamic pie needs the widget method. Without one, `lint`, `create` and `save` report `TimePeriodAggregationRequired` as an error; a pie already saved without one gets a warning instead, so it still saves. Keep `showInLegend` true on each data item when `showLegend` is true. This example leaves `showDataIntervalFilter` out and has a `dataInterval`, so it fixes the pie at Month; set `"showDataIntervalFilter": true` to follow the dashboard's interval with Month as the default.

Pies draw no bands: `save` removes `bands` on a data item, with the note `PieSliceBandsFolded`. A data item with no `name` of its own shows the metric's friendly name, then its name.

**Pie/donut display fields:**

| Field | Use |
|-------|-----|
| `pieChartWidgetTemplateType` | `0` Pie Chart or `1` Donut Chart. Required (`PieChartTypeRequired`). |
| `centerMode` | Donut-only center content: `0` None, `1` Static Text, `2` Metric Value, `3` Total (the sum of the drawn slices), or `5` Percentage (one metric's share of the drawn slices). `4` Share of Total is rejected (`PieCenterModeReserved`): use `5`. A Pie with another mode is `PieCenterNeedsDonut`; Standard Pie templates must omit center fields or use `None`. |
| `centerStaticText` | Required only when `centerMode.id` is `1`. |
| `centerMetricId` | Required when `centerMode.id` is `2` or `5`; references an existing Input or Calculation metric you can view. For `5` it must be one of the pie's own metrics (`PieCenterMetricNotInPie`). Not allowed on Total (`3`). `""` or `null` reads as not picked. |
| `centerNumberFormat` | Only for a Metric Value or Total center. On any other chart or center, `save` removes it, with the note `PieCenterNumberFormatFolded`. |
| `centerMetricLabel` | Optional display-label override for Metric Value, Total and Percentage centers. If blank, a Metric Value or Percentage center uses the metric friendly name, then the metric name; a Total center reads "Total". |
| `legendValueMode` | `0` Hidden, `1` Value, `2` Percent, or `3` Value and Percent: what each legend row adds to its label. `showLegend` and data-item `showInLegend` still control row visibility. |
| `sliceLabelMode` | `0` Hidden, `1` Label, `2` Value, `3` Label and Value, `4` Percent, or `5` Label and Percent for labels rendered on slices. This is separate from tooltips and legend rows. |
| `pieShape` | `0` Full Circle (default) or `1` Half Circle: the top half, for a pie or a donut. |
| `otherSliceMode` | `0` None (default), `1` Top N (keep the largest N slices, group the rest into Other), or `2` Below Percent (group slices under N% of the total into Other; the largest slice always keeps its name). Other forms only when it would hold two or more slices. |
| `otherSliceThreshold` | The number for `otherSliceMode`: `2` to `20` for Top N, `1` to `25` for Below Percent. Required with a mode and refused without one (`PieOtherThresholdInvalid`). |

Percent is each slice's share of the drawn total, to one decimal, rounded after grouping so the shown shares (Other included) add up to 100.0%. A negative slice is left out of the total, a zero slice counts at 0%, and a slice with no value is not a slice. Other's fill is the `styleConfiguration.otherSlice.backgroundColor` slot (default `#8E96A8`).

The same fields round-trip through widget-template Excel download/upload as `Pie Chart Type`, `Center Mode`, `Center Static Text`, `Center Metric`, `Center Metric Label`, `Center Number Format`, `Legend Value Mode`, `Slice Label Mode`, `Pie Shape`, `Other Slice Mode` and `Other Slice Threshold`. On upload, `Center Metric` names the center metric (an exact-name lookup). Legacy workbooks that omit these columns keep the default behavior: no center, legend values visible, slice labels hidden, a full circle, and no Other slice.

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
`Center: <label>: <value>` when configured and includes `Share`, `Legend` and `Slice
Label` columns for quick smoke checks. Other's label reads `Other (5 slices)`.

A negative value can't be a share of a pie, so the slice is left out of the
ring and only the legend lists it: in JSON it has `notDrawn: true`, and text
mode prints its value with `(not drawn)`. Each slice also carries
`dataItemIndex`, the position in `dataItems` of the row it came from (slices a
row splits into share it; a filtered pie's slices have none). The JSON's
`dataWarnings` list says what the data at these sites and periods leaves the
pie unable to draw well; text mode prints them after the table as
`- [Code] message`:

| Code | Severity | Message |
|------|----------|---------|
| `PieNegativeValuesLeftOut` | Warning | `Below zero for {period} at {site}, so left out of the pie: {names}. A pie can't draw a negative share.` |
| `PieManySlices` | Warning | `This pie has {count} slices. Above 7, small slices are hard to read: group small slices into Other, use Keep → Top 6, or split less.` |
| `PieNoValues` | Note | `No slice has a value for {period} at {site}.` |
| `PieMixedUnits` | Warning | `Slices use different units ({units}), so the total shows – and Other shows only its share.` |
| `PieCenterMetricNotInPie` | Warning (a filtered pie's Percentage metric) | `The centre shows the share of a metric that isn't one of this pie's slices. Pick one of its metrics.` |

They depend on the data, so they never block a save and `lint` does not report
them. `dataWarnings` is separate from `warnings` (the CLI's own no-data hints)
and `styleWarnings`.

Render a draft pie without saving it with `preview` (see
[Step 6](#step-6-lint-then-create-the-widget-template)).

#### PieChart Lint

Besides the shared codes (`NameRequired`, `DataItemsRequired`, `MetricFiltersRequired`, `UnknownMetrics`, `MetricNotViewable`, the period window codes, `IntervalFixedNeedsInterval`, `NumberFormatInvalid` and the `StyleSlot*` codes in [Configure Pie Chart Styles](./configure-pie-chart-styles.md)), a pie reports these. Each code has one English text, the same in the web editor, `lint`, `save`, Excel and MCP. Each issue's `stage` is `layout` for the chart type, shape, center, small-slice grouping, legend and slice labels, `look` for the text, `style` for `styleConfiguration` and data-item `presentation` (except `presentation.slice.backgroundColor`, which is `data`), and `data` for the rest:

| Code | Field | Severity | Message |
|------|-------|----------|---------|
| `PieChartTypeRequired` | `pieChartWidgetTemplateType` | Error | `Choose Pie or Donut.` |
| `PieCenterModeReserved` | `centerMode` | Error | `Centre mode {mode} isn't available. Use Percentage to show one metric's share.` (only Share of Total) |
| `PieCenterNeedsDonut` | `centerMode` | Error | `A centre needs a donut. Set Chart to Donut, or set the centre to None.` |
| `PieCenterFieldsNeedMode` | the first of `centerStaticText`, `centerMetricId`, `centerMetricLabel` that is set | Error | `{field} needs a centre mode. Pick Text, Metric value, Total or Percentage, or clear it.` |
| `PieCenterTextRequired` | `centerStaticText` | Error | `Enter the centre text.` |
| `PieCenterFieldNotForMode` | the field the mode doesn't use | Error | `{field} doesn't apply to a {mode} centre. Clear it.` |
| `PieCenterMetricRequired` | `centerMetricId` | Error | `Pick the metric for the centre.` |
| `PieCenterMetricNotInPie` | `centerMetricId` | Error on a picked pie | `The centre shows the share of a metric that isn't one of this pie's slices. Pick one of its metrics.` |
| `PieOtherThresholdInvalid` | `otherSliceThreshold` | Error | `Grouping small slices needs a rule and a number: keep the largest 2 to 20, or group below 1 to 25% of the total.` |
| `TimePeriodAggregationRequired` | `timePeriodAggregationMethod` | Error; a warning on a pie already saved without one | `Choose how slices combine periods: Sum, Average or Last value.` |
| `EnumValueInvalid` | `pieChartWidgetTemplateType`, `centerMode`, `legendValueMode`, `sliceLabelMode`, `pieShape`, `otherSliceMode` | Error | `{value} isn't one of {allowed}.` |
| `PieCenterNumberFormatFolded` | `centerNumberFormat` | Note | `The centre number format only applies to a Metric value or Total centre, so Save removes it.` |
| `PieCenterShareAlwaysFull` | `centerMetricId` | Warning | `Every slice is this metric, so the centre always shows 100%. Pick Total, or add other metrics.` |
| `PieSliceBandsFolded` | `dataItems` (with `index` and `name`) | Note | `Pies don't draw bands, so Save removes the bands on {name}.` |

`UnknownMetrics` and `MetricNotViewable` apply to each data item's metric (with the row's `index`) and to `centerMetricId`. A data item `presentation` issue carries the row's `index` and the full `path`. Pies skip the band rules (`BandsRequireStaticSelection` and the `Band*` codes), because `save` removes slice bands.

#### XYChart Example:

> **Important:** XY Charts require an `axes` array, and every data item must include `xyChartWidgetTemplateAxis` with a `name` matching an axis entry. The API matches by name (not ID) during creation.

**Axes, orientation and series types:**

- **Each axis has a side.** An axis' `side` is `{ "id": 0, "name": "Left" }` or `{ "id": 1, "name": "Right" }`; on a horizontal chart Left draws at the bottom and Right at the top. One axis can sit on the right, and every axis can share one side. Within a side the order of `axes` is the drawing order: the first axis on a side sits nearest the plot. An update that leaves `side` out keeps the stored side; a new axis without one takes the side with fewer axes (Left on a tie), so axes listed without sides alternate Left, Right, Left. Reordering `axes` no longer changes sides: set `side` instead. An unknown side is `EnumValueInvalid` on `axes[i].side`. An axis without an id (from Excel) keeps its stored place and a new one goes last. Excel column: `Axis Side` (`Left` or `Right`; blank keeps the stored side).
- **An axis name may be empty.** An unnamed axis draws the unit of its first series as its title.
- **Start at zero is the axis' `dynamicAxis`.** `false` starts the axis at zero; `true` fits it to the values. `styleConfiguration.valueAxis.startAtZero` is deprecated: it is still accepted, and Save moves it into `dynamicAxis` on every axis and removes it from the style.
- **Horizontal charts use `invertAxes: true`.** Series types are Column (0), Line (2) and Area (3). Bar (1) is still accepted on input, but Save stores it as Column and sets `invertAxes`, which draws the same chart; `lookups xy-chart-data-item-types` no longer lists it.
- **Periods are one window per series.** With `dataRangeMode` Dynamic the series read the viewer's periods; with Static each series reads its own `dataPeriodStart`..`dataPeriodEnd`; with SelectedYear or SelectedYearToLatestData every series reads the viewer's year up to their period, and a metric listed twice is a duplicate. The web editor writes one "last N" window to every series and sets a series' own window with the series menu's **Periods…** and **Compare with an earlier period…**.
- **A series ending earlier draws over the periods it trails.** Under Static, every series' last period sits on the chart's last category: a series at `-7`..`-4` beside one at `-3`..`0` (quarterly: the same four quarters a year earlier) draws its four values over the last four quarters, not beside them. With one such shift, the categories it draws on name both periods (`Q1 FY 26 · Q1 FY 25`); with several, each category names its own period. Series that end at the same offset draw as before. This is how a metric listed twice compares periods: the second series needs another window (`XySeriesDuplicate` otherwise). The shift counts in the interval the chart reads at, so a comparison that must mean "a year earlier" needs `showDataIntervalFilter: false`.
- **Series take the chart's period total.** The widget's `timePeriodAggregationMethod` totals every series while the periods follow the viewer (`dataRangeMode` Dynamic). Under Static or a year mode each point is its period's value: Save clears a widget total there (note `SettingRetiredCleared`). A data item's own `timePeriodAggregationMethod` is still accepted, but Save clears it, with the note `XySeriesPeriodTotalCleared` (none for `None`).
- **A series without a name of its own shows its metric's friendly name.** Save stores a blank `name` as the metric's Name, and the chart shows the friendly name for it, as cards and pies do.
- **Axis text is styled chart-wide.** `styleConfiguration.axisLabel` styles both axes' labels and `axisTitle` both axis titles. `categoryAxis`/`valueAxis` still accept `labelStyle` and `titleStyle`: Save moves each setting into `axisLabel` or `axisTitle` where that part leaves it empty (the category axis' first), removes them from the axis, and adds the note `XyAxisTextStyleMerged`.
- **Lines and zones across the plot.** An axis' `referenceLines` lists fixed lines and zones: `{ "value": 100, "label": "Target", "color": "chrome", "dashArray": "Dash", "width": 2 }` draws a line; adding `"toValue": 85` to `"value": 80` draws a shaded zone from 80 to 85. `color` is a colour token or hex (default `chrome` for a line, `warning-subtle` for a zone), `dashArray` a dash preset (default `Dash`), `width` 1–3 (default 2), `"showLabel": false` draws it without its label on the plot (left out, the label is drawn). Unknown members are refused. A line drawn at a metric's value is a series row with `"role": { "id": 1, "name": "ReferenceLine" }`: in a chart that totals its periods (Period total, By site) it draws one line across the bars at the metric's total for the dashboard's site and periods, not a bar. Over time it draws as a fixed line does, labelled at its end, when its value is the same in every period, and as a step labelled at its last value when the value changes or a period has none. The label is the row's `name` (empty is the metric's friendly name) with the formatted value; the colour, dash and width are the row's `presentation.series` `color`, `dashArray` and `strokeWidth` (1–3), with a line's defaults. The row's `presentation.showLabel: false` draws the line without its label. Over time the row's `showInLegend` and `showTooltip` decide whether the line is in the legend and the tooltip. Capture a target as reference data with **Average** time aggregation: Average repeats a yearly value in every month, so the line is straight; Last Value puts it in the year's last month only and Sum divides it, and both draw a step. A reference row can't be broken down by site, stacked, or drawn on a 100% axis (`XyReferenceLineBySite`, `XyReferenceLineStacked`, `XyReferenceLineOnPercentAxis`).
- **Axis scale.** An axis takes `minimum` and `maximum` (blank = from the data) and `logarithmic: true` (needs `dynamicAxis: true` and values above 0, `XyAxisLogScaleNotPositive`; no stacking on it).
- **Axis step.** `majorStep` labels the axis on every whole multiple of that number, in the axis unit (in percent on a 100% axis, at most 100); a blank range rounds out to whole steps, a set `minimum` or `maximum` stays as typed. `minorTicksPerStep` (0–9; left out with a step = 1) draws unlabelled ticks between the labels. Blank `majorStep` lets the chart choose, as before. A log axis picks its own steps (`XyAxisStepLogScale`); a step of 0 or below is refused (`XyAxisStepInvalid`), as are minor ticks outside 0–9 (`XyAxisMinorTicksInvalid`). A step that would give more labels than the axis has room for (or more than 100, or fewer than 2) draws the chart's own labels instead.
- **Stacks.** `isStacked: true` puts a series in Stack 1; `stackGroup: 2` (or higher) puts it in another stack drawn beside the first. An axis with `percentStacked: true` adds each stack up to 100%: every series on it must be in one stack (`XyAxisPercentNeedsOneStack`), and it takes no `minimum` or `maximum`. A stack mixing columns and lines draws as two stacks (warning `XyStackMixed`).
- **Line shapes.** A Line or Area series takes `"lineShape": { "id": 1, "name": "Smooth" }` or `{ "id": 2, "name": "Step" }`; Straight (0) is the default.
- **Height.** `heightMode` Fit (0, default: fills the dashboard placement, at least 400 px), Grow (1: a horizontal chart gets about 28 px per bar, 240–1,200 px) or Fixed (2, with `height` 240–1,200 px, `XyHeightInvalid`). The plot never gets shorter than the placement.
- **Leaving these out keeps them.** A payload without `role`, an axis without `logarithmic` or `side`, or a chart without `heightMode` keeps the stored settings, so older payloads and workbooks don't reset them. An axis without `logarithmic` keeps each of `minimum`, `maximum`, `majorStep` and `minorTicksPerStep` that it leaves out; an axis that sends `logarithmic` sends its scale in full, so one without `majorStep` clears the step.
- **Dashes are presets.** `series.dashArray` takes `Solid`, `Dash`, `Dot`, `DashDot` or `LongDash`. A raw `strokeDashArray` (chart-wide or per series) is still accepted: Save replaces it with the preset of the same pattern (`[6,4]` Dash, `[2,4]` Dot, `[6,4,2,4]` DashDot, `[12,6]` LongDash), otherwise DashDot when its dashes differ in length, otherwise the preset with the nearest dash length (Dot 2, Dash 6, LongDash 12; a tie takes Dash), with the note `XyDashPatternMapped`.

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
    { "name": "kWh", "dynamicAxis": true, "side": { "id": 0, "name": "Left" } }
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
      "sortOrder": 1
    }
  ],
  "timePeriodAggregationMethod": { "id": 0, "name": "None" }
}
```

`None` is correct for this XY example because it is a time series and should render one point per selected month. For an aggregated XY summary, set the widget's `timePeriodAggregationMethod` to Sum, Average, Last Value, Min, or Max; every series takes it.

**XY style fields:**

| Area | Notes |
|------|-------|
| `styleConfiguration.version` | Use `"1"` for the bounded XY style contract. |
| Slot objects | Supported slots include `panel`, `title`, `description`, `footnote`, `chartArea`, `plotArea`, `categoryAxis`, `valueAxis`, `axisTitle`, `axisLabel`, `gridline`, `series`, `marker`, `dataLabel`, `tooltip`, `legend`, `legendMarker`, `legendLabel`, `legendValue`, `stateMessages`, and `numberFormat`. |
| Axis controls | `categoryAxis`/`valueAxis` accept `labelRotation` (-90..90), `labelDensity` (`Auto`/`Comfortable`/`Compact`), `minLabelGap` (10..300px, overrides density) and `labelMaxWidth`. Their `labelStyle` and `titleStyle` are deprecated: style axis text with `axisLabel` and `axisTitle`; Save folds them in. Value-axis `startAtZero` is deprecated: set each axis' `dynamicAxis` instead (`false` = zero baseline); Save folds a `startAtZero` it receives into the axes. |
| Safety bounds | The CLI and API reject raw CSS, HTML, JavaScript, callbacks, URL/CSS functions, unknown slots, unknown properties, and out-of-range sizes. Errors include `styleConfiguration.<field>` paths. |
| Data-item presentation | Per-series visual overrides live in `dataItems[].presentation.series`; use `color`, `strokeWidth`, `dashArray`, border fields, radius, and opacity there. `strokeDashArray` is deprecated: Save replaces it with a `dashArray` preset. |
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

In the web app, table templates open in the staged table editor with a live preview, and the widget type is fixed once a template is created. To change a table into another type, create a new template; to start a table from an existing one, use **Duplicate** in the widget template list (or, from the CLI, `get` the JSON, clear `id`, give it a new name and pass it to `create`; `save` only updates an existing template).

```json
{
  "name": "Ambient Noise | Site Matrix | Quarterly | Table",
  "title": "Ambient Noise by Site",
  "widgetType": { "id": 4, "name": "Table" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 3, "name": "Quarter" },
  "dataPeriodStart": -3,
  "dataPeriodEnd": 0,
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "dataGrouping": { "id": 1, "name": "OrgNode" },
  "additionalDataGrouping": [{ "id": 2, "name": "Discipline" }],
  "orgNodeRowSelectionMode": { "id": 0, "name": "Children" },
  "showOnlyRowsWithValues": true,
  "showMetricValue": true,
  "metricPropertyColumns": [{ "id": 0, "name": "Unit of Measure" }],
  "showMetricsInColumns": false,
  "missingValuePlaceholder": "-",
  "dataItems": [
    {
      "metric": { "id": "<metric-id>", "name": "Ambient Noise Day Time" },
      "sortOrder": 0,
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null,
      "bands": [
        { "backgroundColor": "danger-subtle", "foregroundColor": "text-primary" },
        { "lowerBoundValue": 90, "backgroundColor": "warning-subtle", "foregroundColor": "text-primary" },
        { "lowerBoundValue": 95, "backgroundColor": "success-subtle", "foregroundColor": "text-primary" }
      ]
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
    "valueCells": { "textAlign": "End" }
  }
}
```

**Key fields:**
- `dataItems` are the static metric selections. When `metricSelectionMode` is Dynamic, the metric filters select the metrics and `dataItems` only override the ones they name.
- `dataPeriodStart` / `dataPeriodEnd` are the table's one Static window (here the last four quarters); the items carry none.
- `dataItems[].bands` overrides the metric's colour bands for this widget. Omit to inherit; `[]` disables formatting.
- `dataGrouping` plus optional `additionalDataGrouping` and `orgNodeRowSelectionMode` control row grouping under the dashboard-selected org node.
- `showMetricsInColumns` switches from one row per metric to metric columns, with any grouping (a flat table needs `showTimePeriodsInRows` too). `showTimePeriodsInRows` gives each row one row per period.
- `columnDataGroupingLevels` moves grouping levels onto columns: for example OrgNode grouping with Discipline on columns gives one row per site and one column group per discipline.
- `showTotalRow` adds an engine-sourced total row for the dashboard org node. It requires OrgNode-only grouping on rows with `showMetricsInColumns` and periods as columns. The footer is the parent engine value, not a client or filtered sum of body rows.
- `showMetricValue`, `metricPropertyColumns`, and `metricAttributeTypeIds` control the Columns tab (metric value plus extra metadata columns).
- `narrativeSelectionMode: Dynamic` uses the narrative scope arrays. Empty arrays mean all permitted values when `narrativeScopeConfigured` is true.
- `narrativeSelectionMode: Static` ignores narrative scope arrays and renders the explicit `narratives[]` list.
- `styleConfiguration` paints the table's slots (every header and cell of the dashboard grid). Keep it sparse and use safe tokens or validated hex values.
- Arbitrary static-text columns, org-node-attribute display columns, expression columns, and `OrgNodeAttribute` row grouping are deferred; do not add a standalone `columns[]` array to current Table JSON.

**Naming convention:** `{Subject} | {Qualifier} | {Interval} | Table`

---

#### TextBlock Example:

TextBlock widgets render metric- and narrative-aware dashboard text without chart, card-value, or table rows. Metrics and narratives are discovered only from explicit text tokens in `title`, `subtitle`, `description`, and `footnote`.

```json
{
  "name": "Portfolio Status | Monthly | TextBlock",
  "title": "Portfolio Status",
  "subtitle": "Performance for @[period]",
  "description": "Total emissions were [<metric-id>] for @[period].\n{<monthly-narrative-id>}",
  "footnote": "#trend[<change-metric-id>] since @[period:-1]",
  "widgetType": { "id": 6, "name": "TextBlock" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "dataPeriodStart": -1,
  "dataPeriodEnd": 0,
  "showDataIntervalFilter": false,
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

`showDataIntervalFilter: false` fixes this text block at Month; leave it out, or set `true`, to follow the dashboard's interval.

Widget-template Excel roundtrips TextBlock text through `Title`, `Subtitle`, `Description`, and `Footnote`, style JSON through `Text Block Style`, and the interval flag through `Table Show Data Interval Filter` (a blank cell keeps the saved flag). In Excel, metric-name and narrative-name tokens (`[Metric Name]`, `{Narrative Name}`) are accepted in the text slots and transposed to IDs on upload. Each of these refuses the row, reported on its cell, while the other rows save: a bracket that is no metric name (a typo, or prose such as `[1]`) is `WIDGET_TOKEN_METRIC_REFERENCE_INVALID`; a name more than one metric uses is `WIDGET_TOKEN_METRIC_REFERENCE_AMBIGUOUS`; a metric ID that is no metric in the tenant is `WIDGET_TOKEN_METRIC_REFERENCE_MISSING`. Unknown or ambiguous narrative names, and other GUIDs, are row/column errors too.

`@[…]` is only for period and selection tokens (`@[period]`, `@[period:-1]`, `@[periods]`, `@[orgNode]`, `@[interval]`), so `@[Metric Name]` in a workbook is reported as `WIDGET_TOKEN_UNKNOWN`; write `[Metric Name]` for a metric.

#### TextBlock token lint

`lint`, `save`, `preview`, Excel and MCP report the same codes for tokens in `title`, `subtitle`, `description` and `footnote` (InfoCard text and row labels too). Each issue's `field` is the slot and `params.key` the token as written:

| Code | Severity | Message |
|------|----------|---------|
| `WIDGET_TOKEN_METRIC_REFERENCE_INVALID` | Error | `{key} is not a metric id or name. Use an existing metric, or remove the brackets.` |
| `WIDGET_TOKEN_METRIC_REFERENCE_AMBIGUOUS` | Error | `More than one metric is named {name}. Give the metrics distinct names, then try again.` |
| `WIDGET_TOKEN_METRIC_REFERENCE_MISSING` | Error | `{key} refers to a metric that no longer exists. Remove the token or use another metric.` |
| `WIDGET_TOKEN_METRIC_REFERENCE_INACCESSIBLE` | Error | `This text refers to a metric you cannot view. Use a metric you can view, or ask an administrator for access.` |
| `WIDGET_TOKEN_UNKNOWN` | Error | `{key} is not a valid token. Check its spelling and any \|n\| offset.` An `@[…]` that is not `period`, `period:n`, `periods`, `orgNode` or `interval`, an offset that is not a whole number (`\|+1\|`, `\|a\|`), or an offset after `@[period]` or `@[orgNode]` |
| `WIDGET_TOKEN_TREND_OFFSET_ORDER` | Error | `Put #trend before the offset, as #trend[metric]\|-1\| or [metric]#trend\|-1\|.` |
| `WIDGET_TOKEN_PERIOD_OFFSET_FUTURE` | Warning | `{key} points after the latest period with data. Offsets usually count back, as \|-1\|.` |
| `TextBlockTextRequired` | Error | `Write a title, subtitle, description or footnote.` |

A token break the saved template already has stays a warning, so it still saves; a new one is an error. `WIDGET_TOKEN_METRIC_REFERENCE_INACCESSIBLE` is always an error. `TextTooLong` counts each metric token as its stored 38-character ID.

A narrative token resolves at the narrative's own capture interval, whatever the widget's `dataInterval`: a quarterly narrative on a monthly widget shows the quarter containing the selected month, and on a yearly widget the latest quarter with text. Offsets such as `|-1|` count in capture periods. Each resolved narrative span carries `periodName` (for example `"Q2 FY 26"`). Narrative save codes are `WIDGET_TOKEN_NARRATIVE_NOT_SUPPORTED`, `WIDGET_TOKEN_NARRATIVE_TREND_NOT_SUPPORTED`, `WIDGET_TOKEN_NARRATIVE_REFERENCE_INVALID` (braces that aren't a `{narrative-guid}` reference), and `WIDGET_TOKEN_NARRATIVE_REFERENCE_MISSING`.

Use the typed render command to inspect resolved text, spans, style, warnings, and diagnostics:

```bash
cap reporting widgets text-block <widget-template-id> \
  --org-nodes "<org-node-id>" \
  --data-interval month \
  --periods "Jan 2026" \
  --json
```

Narrative spans have `tokenKind: 4` and `sourceNarrativeId`. Pass a single org node: with none or several, narrative tokens return `AmbiguousOrgNode` ("Select a single org unit").

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
| `tableRowScope` | `All`, `LeafOrgNodes` | Optional, TableRows only. Use `LeafOrgNodes` for project/asset watchlists that should rank leaf org-node rows instead of parent aggregates |
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
  "tableRowScope": "LeafOrgNodes",
  "predicates": [
    { "predicateType": "Null", "operator": "IsNotNull" }
  ]
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

### Step 6: Lint, then Create the Widget Template

**Purpose:** Check the template, then execute the creation command.

#### Settings Save retires

Every setting a widget stores is one its web editor can set. Older payloads and workbooks may carry settings the editors don't offer: they are still accepted, and Save folds each one into what the editor shows, with a note. `SettingRetiredCleared` (`The web editor doesn't offer {path}, so Save clears it.`) when it goes; `SettingRetiredMapped` (`The web editor doesn't offer {path} as {from}, so Save stores {to}.`) when it becomes the nearest setting the editor offers. Saving the folded widget again notes nothing.

| Widget | Setting | Save stores |
|--------|---------|-------------|
| Every type | `fontWeight` `Medium` / `Semibold` in any style part | `Normal` / `Bold` (the fonts load at 400 and 700, so they already drew this way) |
| Table | `timePeriodAggregationMethod`; each data item's `timePeriodAggregationMethod`, window, `metricPartitioningMode`, `orgNodeAttributeFilters`, `partitioningRankMode`/`Limit`, `valueSelectionConfig`; `orgNodeTemplateId` | Cleared |
| Table | `pageSize` other than 10, 25, 50, 100 | The nearest of those |
| Table | A Static window not ending at `0` | The same length, ending at `0` |
| Table | A column or row override part's colour that is not one of the band meanings the Style stage offers | The nearest offered fill, with its text colour |
| Table | `valueSelectionConfig` fields the rank doesn't show (`nullPlacement`, `diagnosticsLabel`, `boundsWarningMode`, the selector's `fallbackWhenMissing`) | Their defaults |
| InfoCard | Picked rows whose `timePeriodAggregationMethod` (theirs, else the card's) differ | The most common one, as the card's, with none on the rows |
| InfoCard | `accentColor` on a text part (only the panel draws one); the trend's colours, fill, border and opacity | Cleared; the trend's colour or accent becomes the colour of each direction without one |
| InfoCard | A filtered card on One period with a card window other than `0..0` | `0..0` |
| PieChart | Data items' own `timePeriodAggregationMethod` | The pie's (see above) |
| PieChart | `showConditionalFormatting` / `allowConditionalFormattingToggle` `false` | `true` (pies draw no ranges) |
| TextBlock | `timePeriodAggregationMethod`; the trend's `accentColor`; a Static window not ending at `0` | Cleared; the trend's `foregroundColor` where it has none; the same length ending at `0` |
| XYChart | `timePeriodAggregationMethod` under Static or a year mode | Cleared |
| InfoCard, PieChart, XYChart | Keep (`valueSelectionConfig`, the widget's or a data item's) | Ranks and filters by what the widget shows: an `AggregatedRange` selector with its owner's period total (Last Value when none), and the fields the Keep editor doesn't show at their defaults. A rule that differs only in that total is folded without a note |

**Lint first (nothing is saved):**
```bash
cap templates widget-templates lint --file widget.json
```

`lint` takes the same payload as `create` and `save` (with or without the `widgetTemplate` wrapper) and runs the same local checks first. It prints one line per issue, errors first, each with a stable code, for example:

```text
Error   [MetricsRequired] dataItems: Pick at least one metric, or choose Filter metrics.
Note    [SettingRetiredCleared] orgNodeTemplateId: The web editor doesn't offer orgNodeTemplateId, so Save clears it.

1 error(s), 0 warning(s), 1 note(s).
```

Errors are exactly what `create` and `save` would reject (exit code `2`); fix them and lint again. Warnings and notes never block a save. Use `--id <guid>` instead of `--file` to check a saved template as it is stored, and `--json` for the raw `{ "errors", "warnings", "notes" }` result. `create` and `save` run the same check first: a template with errors is not sent, and the warnings print after saving. Against an older server without lint they save without it.

**Preview a draft table, info card, text block or pie chart (nothing is saved):**
```bash
cap templates widget-templates resolve-content --file widget.json
cap templates widget-templates preview --file widget.json --org-nodes <org-node-id> --periods "Jan 2026,Feb 2026"
```

`resolve-content` lists the metrics and narratives the table resolves to. `preview` renders every row for the org node and the named periods, with the same options and the same grid as `cap reporting widgets table`; add `--data-interval month` (or another interval) to set the request interval and `--json` for the full render contract. Table, info card and pie chart templates preview and resolve; an info card previews in one request and prints its items. A text block previews in one request and prints its resolved title, subtitle, description and footnote; metric names in its text resolve as `save` stores them, and `--json` returns the text block render contract, each value span with the `periodName` it was read at. A pie previews in one request and prints what `cap reporting widgets pie-chart` prints once it is saved: the center, the slices (a negative one marked `(not drawn)`) and its data warnings; `--json` returns the pie render contract the server returned, without the CLI's `noData` hint. A filtered pie whose filters match no metric is refused with HTTP 400 "No metrics match this widget's filters." `resolve-content` lists a pie's slice metrics (picked, or matched by its filters), not its center metric. `resolve-content` does not take text blocks: their references are the tokens in their text.

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
- Success response with new widget template ID, plus any lint warnings
- On a rejected save, the error lists each field's messages (for example `name: ...`); run `lint` to see the same failures with their codes

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

### Creating or Updating Many Templates via Excel

```bash
cap templates widget-templates download-excel --output widget-templates.xlsx
# edit rows, then
cap templates widget-templates upload-excel -f widget-templates.xlsx --json
```

Upload runs the same lint as `lint` on each template. Templates without errors are saved; a template with an error is not saved and is reported on its first row and the column of the failing field, with the code. An info card's issue on one of its metrics is reported on that metric's row instead, in the column of the setting: `Number Format`, `Name` (`DataItemLabelDuplicate`), `Data Period Start` (`DataItemWindowSinglePeriod`), `Bands` for band codes, else `Metric`; its style issues are in the `Info Card Style` column. The result starts with the counts, for example "12 saved, 2 not saved"; warnings are reported and the template saves. Fix the reported rows and upload again: rows match existing templates by name, so the saved ones are updated in place.

The table narrative settings have their own columns: `Table Narrative Selection Mode`, `Table Narrative Scope Configured` and the five narrative scope filter columns. A workbook downloaded before these columns existed can still be uploaded: on update the template keeps its stored narrative settings, and a new template takes the defaults. When the columns are there, a blank mode or scope-configured cell also keeps the stored value, while a blank filter cell clears that filter. Filter cells use the same `Attribute: Value|Value` text as the other attribute filter columns, and each value is looked up within its own attribute. Other missing columns behave as described for each column.

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
- For a horizontal chart (`invertAxes: true` with Column series), apply the same readability settings to the category axis; the renderer maps the configured category-axis style to the displayed category labels.
- Use `showLegend: true` when multiple series are plotted on the same chart.
- Rank many bars horizontally with `heightMode` Grow, so each bar keeps about 28 px.
- Show a target across bars that compare sites or metrics with a reference line (an axis' fixed `referenceLines`, or a target metric row with role ReferenceLine), not with another bar.

---

## Related Recipes

- [Discover Tenant Structure](../exploration/discover-tenant-structure.md) — Find disciplines and metrics
- [Verify Data](../reporting/verify-data.md) — Confirm widget returns expected data
- [Talk to Your Dashboard](../exploration/talk-to-your-dashboard.md) — View widget data
- [Create Metric Wizard](./create-metric.md) — Create metrics for widgets
