# Recipe: Create Widget Template

> Guided workflow for creating widget templates (InfoCard, PieChart, XYChart, Table).
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
- [ ] What type of widget (InfoCard, PieChart, XYChart, Table)
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

**Ask if not provided:**
- "What kind of visualization do you need?"
- "Are you showing a single value, a breakdown, a trend over time, or row-and-column detail?"

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

### Step 3: Select Metrics for Data Items

**Purpose:** Define what data the widget displays.

**Command:**
```bash
cap model metrics list --json
```

**For InfoCard:** Usually one primary metric
**For PieChart:** Multiple metrics that form parts of a whole
**For XYChart:** One or more metrics to plot over time
**For Table:** Metrics become row sources; value columns can use row metrics, selected metrics, or calculation metrics

**Note metric IDs** for the dataItems configuration.

---

### Step 4: Configure Widget Properties

**Required fields for all widget types:**
- `name` - Internal name for the template (follows naming convention, not shown to users)
- `title` - Displayed to dashboard users as the widget heading
- `description` - Optional text shown to dashboard users below the widget content (see User-Facing Fields rules below)
- `widgetType` - { id: 0|1|2|4, name: "InfoCard"|"PieChart"|"XYChart"|"Table" }
- `discipline` - { id: "<id>" }
- `dataRangeMode` - { id: 1, name: "Static" } or { id: 0, name: "Dynamic" }
- `dataInterval` - { id: 2, name: "Month" }, { id: 3, name: "Quarter" }, etc.
- `metricSelectionMode` - { id: 0, name: "Static" } (explicit metrics) or { id: 1, name: "Dynamic" }
- `dataItems` - Array of metric configurations

**Required fields for Table widgets:**
- `tableVersion` - Must be `1`
- `missingValuePlaceholder` - Usually `-` or `n/a`
- `rowFields` - At least one rendered row metadata column
- `valueFields` - At least one metric-backed value column
- `projectedRowBudget` - Required when row count cannot be inferred from static non-partitioned row sources
- `columnGroups` and `sortRules` - Arrays; use `[]` if not needed

**Data Interval Reference:**
| ID | Name |
|----|------|
| 0 | Day |
| 1 | Week |
| 2 | Month |
| 3 | Quarter |
| 4 | Year |

---

### User-Facing Fields: Description, Footnote, and Display Name

These fields are shown to **dashboard viewers** — they must add value for the person reading the dashboard, not the person building it.

#### `description` (widget-level)

Leave **empty** in most cases. The title, values, and footnote already tell the story. Only add a description when extra context genuinely helps the viewer understand what they're seeing.

| Good | Bad |
|------|-----|
| `""` (empty — widget is self-explanatory) | `"Shows total cost from all sites"` (repeats what the title says) |
| `"Excludes contractor hours"` (useful caveat) | `"For Executive P&L dashboard"` (developer note) |
| | `"Created for the Example demo"` (irrelevant to viewer) |

**Tip:** If you're tempted to add a short interpretive note (e.g., "Positive = Cola earns more per unit than Orange"), use the **footnote** instead — it renders more subtly beneath the widget content.

#### `footnote` (widget-level)


| Use Case | Example |
|----------|---------|
| Trend indicator | `#trend[<id>] since @[period:-1]` |
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
  "dataItems": [
    {
      "metric": { "id": "<metric-id>", "name": "Electricity Consumption" },
      "aggregationMethod": { "id": 1, "name": "Sum" }
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
- `timePeriodAggregationMethod: None` — each data item shows a single period's value

**Naming convention:** `{Metric} | Comparison | {Interval} | Card`

#### PieChart Example:
```json
{
  "name": "Waste Breakdown by Type",
  "title": "Waste Distribution",
  "widgetType": { "id": 1, "name": "PieChart" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "showLegend": true,
  "pieChartWidgetTemplateType": { "id": 0, "name": "Standard" },
  "dataItems": [
    { "metric": { "id": "<metric-id>", "name": "Recycled Waste" } },
    { "metric": { "id": "<metric-id>", "name": "Landfill Waste" } },
    { "metric": { "id": "<metric-id>", "name": "Composted Waste" } }
  ]
}
```

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
  "rotateCategoryAxisLabels": true,
  "axes": [
    { "name": "kWh", "dynamicAxis": true }
  ],
  "dataItems": [
    {
      "metric": { "id": "<metric-id>", "name": "Electricity Consumption" },
      "xyChartWidgetTemplateDataItemType": { "id": 2, "name": "Line" },
      "xyChartWidgetTemplateAxis": { "name": "kWh", "dynamicAxis": true },
      "colour": "#1565C0",
      "strokeWidth": 2,
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

#### Table Example:

Table widgets are the dashboard grid/table widget type. They use normal Input and Calculation metrics as row sources and value bindings.

```json
{
  "name": "Ambient Noise | Site Matrix | Quarterly | Table",
  "title": "Ambient Noise by Site",
  "widgetType": { "id": 4, "name": "Table" },
  "discipline": { "id": "<discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 3, "name": "Quarter" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "timePeriodAggregationMethod": { "id": 0, "name": "None" },
  "tableVersion": 1,
  "missingValuePlaceholder": "-",
  "projectedRowBudget": 25,
  "projectedCellBudget": 250,
  "defaultRowSortMode": { "id": 1, "name": "DisplayText" },
  "dataItems": [
    {
      "metric": { "id": "<metric-id>", "name": "Ambient Noise Day Time" },
      "sortOrder": 0,
      "metricPartitioningMode": { "id": 1, "name": "Children" },
      "timePeriodAggregationMethod": { "id": 0, "name": "None" },
      "partitioningRankMode": { "id": 0, "name": "None" }
    }
  ],
  "rowFields": [
    {
      "stableKey": "org-node",
      "sourceType": { "id": 6, "name": "OrgNodeName" },
      "headerTranslations": [{ "value": "Org Node", "language": { "code": "en" } }],
      "sortOrder": 0,
      "widthMode": { "id": 2, "name": "Fill" },
      "alignment": { "id": 0, "name": "Left" }
    },
    {
      "stableKey": "metric",
      "sourceType": { "id": 0, "name": "MetricName" },
      "headerTranslations": [{ "value": "Metric", "language": { "code": "en" } }],
      "sortOrder": 1,
      "widthMode": { "id": 2, "name": "Fill" },
      "alignment": { "id": 0, "name": "Left" }
    }
  ],
  "valueFields": [
    {
      "stableKey": "current-value",
      "bindingMode": { "id": 0, "name": "RowMetricValue" },
      "headerTranslations": [{ "value": "Current Value", "language": { "code": "en" } }],
      "periodOffset": 0,
      "aggregationMethod": { "id": 0, "name": "None" },
      "formatter": { "id": 0, "name": "Number" },
      "precision": 2,
      "unitMode": { "id": 3, "name": "Cell" },
      "alignment": { "id": 2, "name": "Right" },
      "sortOrder": 0
    }
  ],
  "columnGroups": [],
  "sortRules": [
    {
      "targetKind": { "id": 0, "name": "RowField" },
      "targetStableKey": "metric",
      "direction": { "id": 0, "name": "Ascending" },
      "nullPlacement": { "id": 0, "name": "Last" },
      "sortOrder": 0
    }
  ]
}
```

**Key fields:**
- `dataItems` are the Table row sources.
- `rowFields` define rendered row header/metadata columns.
- `valueFields` define metric-backed value columns.
- `RowMetricValue` binds each value cell to the row's metric.
- `SelectedMetricValue` and `CalculationMetricValue` bind a value column to an explicit Input or Calculation metric.
- `projectedCellBudget` should stay below the default 2,000 cells unless there is a specific reason to allow more.

**Naming convention:** `{Subject} | {Qualifier} | {Interval} | Table`

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
  "dataItems": []
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

Use the type-specific Table command when you need to validate `columns`, `columnGroups`, `rows`, budgets, truncation, and warnings. Use `widgets get-data` when you need the generic CSV/AI extraction payload.

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
  "dataItems": [
    {
      "metric": { "id": "<id>", "name": "Electricity Consumption" },
      "aggregationMethod": { "id": 1, "name": "Sum" }
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
cap templates lookups widget-types --json
cap templates lookups data-range-modes --json
cap templates lookups metric-selection-modes --json
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

- **Always set `rotateCategoryAxisLabels: true`** for daily-interval charts. Without rotation, date labels overlap and become unreadable.
- Monthly/quarterly charts can use `false` since there are fewer labels.
- Use `showLegend: true` when multiple series are plotted on the same chart.

---

## Related Recipes

- [Discover Tenant Structure](../exploration/discover-tenant-structure.md) — Find disciplines and metrics
- [Verify Data](../reporting/verify-data.md) — Confirm widget returns expected data
- [Talk to Your Dashboard](../exploration/talk-to-your-dashboard.md) — View widget data
- [Create Metric Wizard](./create-metric.md) — Create metrics for widgets
