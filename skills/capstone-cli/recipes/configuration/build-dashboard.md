# Recipe: Build Dashboard

> Guided workflow for creating a dashboard with widget templates.
>
> **MCP alternative:** Dashboards can also be created and modified via MCP using the `templates_dashboards_create`, `templates_dashboards_get`, and `templates_dashboards_save` tools. See [MCP Tool Reference — Dashboard Templates](../../../capstone-mcp/reference/tools.md#dashboard-templates-4-tools).

## When to Use

- "I need a dashboard for energy metrics"
- "Build a dashboard showing our KPIs"
- "Create a dashboard with charts and cards"
- "Set up a reporting dashboard"
- "How do I create a dashboard?"

## Required Context

Before starting, Claude should know:
- [ ] What the dashboard should show (metrics, KPIs, domain)
- [ ] Optional: Preferred widget types (cards, charts, pies)
- [ ] Optional: Target audience (executive, operational, site-level)

**If missing:** Claude will ask clarifying questions in Step 1.

---

## Step-by-Step

### Step 1: Plan the Dashboard Layout

**Purpose:** Determine what widgets the dashboard needs.

**Ask if not provided:**
- "What metrics or KPIs should appear on this dashboard?"
- "Who will use it? (Executives want summaries, operators want details)"
- "Do you want trend charts, single-value cards, or breakdowns?"

**Suggest a layout based on common patterns:**

```
Executive Energy Dashboard
┌─────────────────────────────────────────────────┐
│ Section: Overview                                │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐         │
│ │ InfoCard │ │ InfoCard │ │ InfoCard │         │
│ │ Total    │ │ Diesel   │ │ Energy   │         │
│ │ Energy   │ │ CO2e     │ │ Intensity│         │
│ └──────────┘ └──────────┘ └──────────┘         │
├─────────────────────────────────────────────────┤
│ Section: Trends                                  │
│ ┌────────────────────────────────────────┐      │
│ │ XYChart — Monthly Energy Trend         │      │
│ └────────────────────────────────────────┘      │
├─────────────────────────────────────────────────┤
│ Section: Breakdown                               │
│ ┌──────────────────┐ ┌──────────────────┐       │
│ │ PieChart         │ │ PieChart         │       │
│ │ Energy by Source │ │ Emissions by Type│       │
│ └──────────────────┘ └──────────────────┘       │
└─────────────────────────────────────────────────┘
```

**Get user confirmation on the layout before proceeding.**

---

### Step 2: Create Widget Templates

**Purpose:** Build the individual widgets that will populate the dashboard.

For each widget in the plan, follow the [Create Widget Template](./create-widget-template.md) recipe.

**Example — create 3 InfoCards:**

```bash
# 1. Total Energy InfoCard
echo '{
  "name": "Total Energy | Static | Month | Card",
  "title": "Total Energy",
  "widgetType": { "id": 0, "name": "InfoCard" },
  "discipline": { "id": "<energy-discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "dataItems": [
    { "metric": { "id": "<total-energy-id>" }, "sortOrder": 1 }
  ]
}' | cap templates widget-templates create --json
# → Returns widget ID: widget-1-id

# 2. Diesel CO2e InfoCard
echo '{
  "name": "Diesel CO2e | Static | Month | Card",
  "title": "Diesel Emissions",
  "widgetType": { "id": 0, "name": "InfoCard" },
  "discipline": { "id": "<emissions-discipline-id>" },
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 2, "name": "Month" },
  "metricSelectionMode": { "id": 0, "name": "Static" },
  "dataItems": [
    { "metric": { "id": "<diesel-co2e-id>" }, "sortOrder": 1 }
  ]
}' | cap templates widget-templates create --json
# → Returns widget ID: widget-2-id
```

**Record all widget template IDs** — they're needed for the dashboard template in Step 4.

---

### Step 3: Verify Widget Templates

**Purpose:** Confirm all widgets were created before assembling the dashboard.

**Command:**
```bash
cap templates widget-templates list --json
```

**Check that all planned widgets exist:**
```
Created widget templates:
- widget-1-id: Total Energy (InfoCard)
- widget-2-id: Diesel Emissions (InfoCard)
- widget-3-id: Energy Intensity (InfoCard)
- widget-4-id: Monthly Energy Trend (XYChart)
- widget-5-id: Energy by Source (PieChart)
- widget-6-id: Emissions by Type (PieChart)
```

---

### Step 4: Create the Dashboard Template

**Purpose:** Assemble widgets into sections with a layout hierarchy.

Dashboard templates use a tree structure where **sections** are parent nodes and **widgets** are leaf nodes.

**Command:**
```bash
cat <<'EOF' | cap templates dashboard-templates create --json
{
  "id": "<empty-id>",
  "name": "Executive Energy Dashboard",
  "description": "Monthly energy KPIs, trends, and breakdowns",
  "useTabSheet": false,
  "treeItems": [
    {
      "id": "<id>",
      "name": "Overview",
      "isWidget": false,
      "sortOrder": 1,
      "parent": null
    },
    {
      "id": "<id>",
      "name": "Total Energy",
      "isWidget": true,
      "sortOrder": 1,
      "parent": { "id": "<id>" },
      "widgetTemplate": { "id": "<widget-1-id>" },
      "widgetType": { "id": 0, "name": "InfoCard" },
      "widgetSize": { "id": 2, "name": "50%" }
    },
    {
      "id": "<id>",
      "name": "Diesel Emissions",
      "isWidget": true,
      "sortOrder": 2,
      "parent": { "id": "<id>" },
      "widgetTemplate": { "id": "<widget-2-id>" },
      "widgetType": { "id": 0, "name": "InfoCard" },
      "widgetSize": { "id": 2, "name": "50%" }
    },
    {
      "id": "<id>",
      "name": "Energy Intensity",
      "isWidget": true,
      "sortOrder": 3,
      "parent": { "id": "<id>" },
      "widgetTemplate": { "id": "<widget-3-id>" },
      "widgetType": { "id": 0, "name": "InfoCard" },
      "widgetSize": { "id": 2, "name": "50%" }
    },
    {
      "id": "<id>",
      "name": "Trends",
      "isWidget": false,
      "sortOrder": 2,
      "parent": null
    },
    {
      "id": "<id>",
      "name": "Monthly Energy Trend",
      "isWidget": true,
      "sortOrder": 1,
      "parent": { "id": "<id>" },
      "widgetTemplate": { "id": "<widget-4-id>" },
      "widgetType": { "id": 2, "name": "XYChart" },
      "widgetSize": { "id": 5, "name": "100%" }
    },
    {
      "id": "<id>",
      "name": "Breakdown",
      "isWidget": false,
      "sortOrder": 3,
      "parent": null
    },
    {
      "id": "<id>",
      "name": "Energy by Source",
      "isWidget": true,
      "sortOrder": 1,
      "parent": { "id": "<id>" },
      "widgetTemplate": { "id": "<widget-5-id>" },
      "widgetType": { "id": 1, "name": "PieChart" },
      "widgetSize": { "id": 1, "name": "33%" }
    },
    {
      "id": "<id>",
      "name": "Emissions by Type",
      "isWidget": true,
      "sortOrder": 2,
      "parent": { "id": "<id>" },
      "widgetTemplate": { "id": "<widget-6-id>" },
      "widgetType": { "id": 1, "name": "PieChart" },
      "widgetSize": { "id": 1, "name": "33%" }
    }
  ]
}
EOF
```

**Tree structure key points:**
- Sections (`isWidget: false`) are parent nodes with `parent: null` for top-level
- Widgets (`isWidget: true`) reference a section via `parent: { "id": "<section-id>" }`
- `sortOrder` controls display order within each section
- Use sequential zero IDs for `id` on create — the API assigns real IDs

**Narrative widget shape:**
```json
{
  "id": "<id>",
  "name": "Executive Summary",
  "isWidget": true,
  "sortOrder": 1,
  "parent": { "id": "<section-id>" },
  "narrative": { "id": "<narrative-definition-id>", "name": "Executive Summary" },
  "widgetType": { "id": 5, "name": "Narrative" },
  "widgetSize": { "id": 5, "name": "100%" }
}
```
- The CLI `save` command accepts both bare JSON and the `get --json` wrapper format (`{"tenant": "...", "dashboardTemplate": {...}}`) — it auto-unwraps the `dashboardTemplate` property

**What to look for:**
- Success response: `{ "success": true, "id": "<new-dashboard-id>" }`

---

### Step 5: Verify the Dashboard

**Purpose:** Confirm the dashboard structure is correct.

**Command:**
```bash
cap templates dashboard-templates get <dashboard-id> --json
```

**Check the hierarchy:**
```
Dashboard: Executive Energy Dashboard

Sections:
1. Overview
   - Total Energy (InfoCard)
   - Diesel Emissions (InfoCard)
   - Energy Intensity (InfoCard)
2. Trends
   - Monthly Energy Trend (XYChart)
3. Breakdown
   - Energy by Source (PieChart)
   - Emissions by Type (PieChart)
```

---

### Step 6: Query Dashboard Data

**Purpose:** Test the dashboard with real data.

**Commands:**
```bash
# Get all widget data as CSV (for AI analysis)
cap reporting dashboards get-data <dashboard-id> \
  --org-node <org-node-id> \
  --data-interval month \
  --periods "Jan 2026" \
  --json

# Get AI-generated insights for a specific section
cap reporting dashboards get-insights <dashboard-id> <section-node-id> \
  --org-node <org-node-id> \
  --data-interval month \
  --periods "Jan 2026" \
  --json
```

**To drill into a specific widget:**
```bash
# get-data auto-detects the data interval from the widget template
cap reporting widgets get-data <widget-template-id> \
  --org-node <org-node-id> \
  --periods "Jan 2026" \
  --json
```

> **Tip:** Use `dashboards get-data` for a full dashboard sweep, then `widgets get-data` to drill into specific widgets that need attention. See [Verify Data](../reporting/verify-data.md) for the complete data verification workflow including troubleshooting empty values.

**What to check:**
- All expected metrics have values (empty = missing data or misconfigured calculation)
- Values appear at the right org node levels (enterprise, sites, etc.)
- Calculation-derived metrics are computing (may need 15-30 seconds after creation)

**Present confirmation:**
```
Dashboard created and verified!

ID: <dashboard-id>
Name: Executive Energy Dashboard
Sections: 3 (Overview, Trends, Breakdown)
Widgets: 6
Data: All metrics returning values

Query data:
  cap reporting dashboards get-data <dashboard-id> --org-node <id> --data-interval month --periods "Jan 2026" --json

View in web UI:
  Navigate to Dashboards > Executive Energy Dashboard
```

---

## Example Interaction

**User:** "Create a dashboard for our monthly safety metrics"

**Claude:** I'll help you build a safety dashboard. Let me find the relevant metrics and disciplines.

```bash
cap masterdata disciplines list --json
cap model metrics list --json
```

Found under "Social > Health & Safety":
- Safety Incidents (Input, count)
- Near Misses (Input, count)
- Training Hours (Input, hours)
- LTIFR (Calculation, rate)

**Claude:** Here's a suggested layout:

```
Safety Dashboard
├── KPIs: Incidents (Card), Near Misses (Card), LTIFR (Card)
├── Trends: Monthly Incidents (XYChart)
└── Breakdown: Incidents by Type (PieChart)
```

Does this work, or would you like to adjust?

**User:** "Looks good, go ahead"

**Claude:** I'll create the widget templates first, then assemble them into a dashboard.

*(Creates each widget template, then assembles the dashboard template)*

Dashboard created! You can view it in the web UI or query data:
```bash
cap reporting dashboards get-data <id> --org-node <id> --data-interval month --periods "Jan 2026" --json
```

---

## Section Visual Properties

Dashboard sections support visual properties that make the dashboard navigable and professional:

| Property | Type | Max Length | Description |
|----------|------|------------|-------------|
| `icon` | string | 50 chars | FontAwesome CSS class (e.g., `"fas fa-gauge-high"`) |
| `backgroundColor` | string | 10 chars | Hex colour (e.g., `"#004A7C"`) |
| `image` | string | — | Base64-encoded image (alternative to icon) |

### Example Section with Visual Properties

```json
{
  "name": "Key Metrics",
  "isWidget": false,
  "sortOrder": 1,
  "parent": null,
  "icon": "fas fa-gauge-high",
  "backgroundColor": "#004A7C"
}
```

### Colour Strategy

Use **one primary colour per dashboard** (or per tab in tabbed dashboards) rather than varying colours per section. This creates a cohesive visual identity without a "Christmas tree" effect.

| Dashboard Theme | Suggested Colour | Hex |
|----------------|-----------------|-----|
| Financial / P&L | Dark Blue | `#004A7C` |
| OEE / Efficiency | Amber/Orange | `#e09e00` |
| Production / Operations | Green | `#00806a` |
| Cost Analysis | Brown | `#b77a46` |
| Revenue Analysis | Dark Gray | `#2C3E50` |
| General / Mixed | Blue | `#4196cb` |

### Icon Selection

Use [FontAwesome Pro 6.6 Solid](https://fontawesome.com/icons) icons. Format: `"fas fa-{icon-name}"`. Choose icons that communicate the section's purpose at a glance:

| Section Type | Suggested Icon | Class |
|-------------|---------------|-------|
| KPIs / Headlines | Gauge | `fas fa-gauge-high` |
| Trends / Charts | Trend arrow | `fas fa-arrow-trend-up` |
| Breakdowns | Pie chart | `fas fa-chart-pie` |
| Comparisons | Scale | `fas fa-scale-balanced` |
| Cost / Financial | Invoice | `fas fa-file-invoice-dollar` |
| Production | Factory | `fas fa-industry` |
| Targets | Bullseye | `fas fa-bullseye` |
| AI / Insights | Lightbulb | `fas fa-lightbulb` |

---

## Widget Sizing Guidelines

Choose widget sizes based on content type and screen compatibility:

| Widget Type | Recommended Size | ID | Rationale |
|-------------|-----------------|-----|-----------|
| **InfoCard** | 50% | 2 | Two cards per row; readable on smaller monitors |
| **PieChart** | 33% | 1 | Compact; pairs well alongside other widgets |
| **XYChart (daily)** | 100% | 5 | Daily data has many points; full width prevents label crowding |
| **XYChart (comparison)** | 50% | 2 | Fewer points; side-by-side enables visual comparison |
| **Table** | 100% | 5 | Dense rows/columns usually need full dashboard width |
| **AI Summary** | 100% | 5 | Always full width; placed last in a section |

Specify widget size in the treeItems entry:

```json
{
  "name": "Filler Throughput | Card",
  "isWidget": true,
  "sortOrder": 1,
  "parent": { "id": "<section-id>" },
  "widgetTemplate": { "id": "<widget-template-id>" },
  "widgetType": { "id": 0, "name": "Info Card" },
  "widgetSize": { "id": 2, "name": "50%" }
}
```

---

## Variations

### Tab-Based Dashboard

For dashboards with multiple pages/tabs:

```json
{
  "name": "Operations Dashboard",
  "useTabSheet": true,
  "treeItems": [
    { "name": "Energy", "path": "Energy", "isWidget": false, "sortOrder": 1, "parent": null },
    { "name": "Water", "path": "Water", "isWidget": false, "sortOrder": 2, "parent": null },
    { "name": "Waste", "path": "Waste", "isWidget": false, "sortOrder": 3, "parent": null }
  ]
}
```

Each top-level section becomes a tab.

### 3-Level Tab Nesting (Tab → Section → Widget)

> **CRITICAL:** For 3-level nesting, every tree item AND parent reference **must** include a `path` field. The API resolves parent references using path-based fallback when placeholder IDs can't be matched. Without paths, sections become orphaned top-level nodes.

```json
{
  "useTabSheet": true,
  "treeItems": [
    {
      "id": "<id>",
      "name": "Monthly Summary",
      "path": "Monthly Summary",
      "isWidget": false, "sortOrder": 1, "parent": null
    },
    {
      "id": "<id>",
      "name": "Key Metrics",
      "path": "Monthly Summary->Key Metrics",
      "isWidget": false, "sortOrder": 1,
      "parent": { "id": "<id>", "path": "Monthly Summary" }
    },
    {
      "name": "Revenue Card",
      "path": "Monthly Summary->Key Metrics->Revenue Card",
      "isWidget": true, "sortOrder": 1,
      "parent": { "id": "<id>", "path": "Monthly Summary->Key Metrics" },
      "widgetTemplate": { "id": "<widget-template-id>" },
      "widgetType": { "id": 0, "name": "Info Card" },
      "widgetSize": { "id": 2, "name": "50%" }
    }
  ]
}
```

**Path rules:**
- Path separator is `->` (matches the `TreeNodeService.PathSeparator`)
- Tabs: `path` = node name (e.g., `"Monthly Summary"`)
- Sections: `path` = `"Tab->Section"` (e.g., `"Monthly Summary->Key Metrics"`)
- Widgets: `path` = `"Tab->Section->Widget"` (e.g., `"Monthly Summary->Key Metrics->Revenue Card"`)
- Parent references must include both `id` and `path`

### Adding AI Summary Widgets

AI Summary widgets generate AI-powered insights from peer or descendant widgets. They have **no backing widget template** (`widgetTemplate: null`).

**Preferred pattern:** Place the AI Summary in its **own dedicated section** named "Insights" with a `fas fa-lightbulb` icon, using the same background colour as the other sections in that tab. Set the context to `Peers` so it summarizes data from all sibling sections' widgets.

```
Tab: Cost Analysis [bg: #b77a46]
├── Section: Cost Headlines [bg: #b77a46]
│   └── (info cards...)
├── Section: Cost Breakdown [bg: #b77a46]
│   └── (<id>)
└── Section: Insights [icon: fas fa-lightbulb, bg: #b77a46]   ← dedicated section
    └── AI Summary (Peers)                                     ← analyzes all peer sections
```

**Section JSON:**
```json
{
  "name": "Insights",
  "isWidget": false,
  "sortOrder": 99,
  "icon": "fas fa-lightbulb",
  "backgroundColor": "<same-as-other-sections-in-tab>",
  "parent": { "id": "<tab-id>", "path": "<tab-path>" }
}
```

**Widget JSON:**
```json
{
  "name": "AI Summary",
  "isWidget": true,
  "sortOrder": 1,
  "parent": { "id": "<insights-section-id>", "path": "<tab>->Insights" },
  "widgetTemplate": null,
  "widgetType": { "id": 3, "name": "AI Summary" },
  "widgetSize": { "id": 5, "name": "100%" },
  "aiSummaryContext": { "id": 0, "name": "Peers" }
}
```

**Key fields:**
- `widgetTemplate` — must be `null` (AI Summary has no template)
- `widgetType` — `{ "id": 3, "name": "AI Summary" }`
- `widgetSize` — always `100%` (full width)
- `aiSummaryContext` — `Peers` (id: 0) analyzes widgets in sibling sections at the same level; `Descendants` (id: 1) analyzes all widgets in child sections below

**Design rules:**
- Always place in a dedicated "Insights" section — not mixed in with other widgets
- Use `fas fa-lightbulb` icon for the section
- Match the section's `backgroundColor` to the tab's other sections
- Set context to `Peers` (default) — this summarizes all widgets across the sibling sections in the tab
- Place the Insights section last (highest `sortOrder`) within the tab

### Validating a Dashboard

After creating or updating a dashboard, verify the structure matches your design:

```bash
# Structure check — CLI outputs the raw API DashboardTemplateDTO format
cap templates dashboard-templates get <id> --json
```

> **Tip:** The CLI's `get --json` output contains the full `DashboardTemplateDTO` with flat `treeItems` and `EnumDTO` objects — the same format the API uses for GET and SAVE.

### Updating an Existing Dashboard

#### Roundtrip Format

The CLI's `get --json` output is directly roundtrippable with `save --file`:

```bash
# Full roundtrip — no conversion needed
cap templates dashboard-templates get <id> --json > dashboard.json
# Edit dashboard.json as <id>
cap templates dashboard-templates save --file dashboard.json --json
```

The `get --json` output wraps the raw API `DashboardTemplateDTO` in `{ "tenant": "...", "dashboardTemplate": { ... } }`. The `save` command auto-unwraps via the `dashboardTemplate` property.

The `dashboardTemplate` contains a flat `treeItems` array with `EnumDTO` objects — this is the same format the API uses internally.

#### API DTO Contract Reference

The `DashboardTemplateTreeItemDTO` defines the SAVE format:

```
DashboardTemplateTreeItemDTO extends TreeNodeDTO:
  IsWidget: bool
  WidgetTemplate: NamedDTO?       ← { "id": "id", "name": "template name" }
  Narrative: NamedDTO?            ← { "id": "id", "name": "narrative name" }
  WidgetType: EnumDTO?            ← { "id": 2, "name": "XYChart" }
  WidgetSize: EnumDTO?            ← { "id": 2, "name": "50%" }
  SortOrder: int
  AISummaryContext: EnumDTO?      ← { "id": 0, "name": "Peers" }

TreeNodeDTO extends TreePathDTO:
  Parent: TreePathDTO?            ← { "id": "parent-id" }
  Icon: string?
  BackgroundColor: string?        ← "#00806a"

TreePathDTO extends NamedDTO:
  Path: string?                   ← "Tab->Section->Widget"
```

#### Widget Type Rules (CRITICAL)

`widgetType` is **REQUIRED on ALL widgets**. The validator rejects widget placements that omit it because the frontend and reporting endpoints route by widget type.

There are three kinds of widgets:

| Widget Kind | Has `widgetTemplate`? | Has `widgetType`? | Extra fields |
|-------------|----------------------|-------------------|-------------|
| **Regular widget** (InfoCard, PieChart, XYChart, Table) | Yes — `{ "id": "<id>" }` | Yes — REQUIRED | `widgetSize` |
| **AI Summary** | No — omit entirely | Yes — `{ "id": 3, "name": "AISummary" }` | `aiSummaryContext`, `widgetSize` |
| **Narrative** | No — use `narrative` instead | Yes — `{ "id": 5, "name": "Narrative" }` | `narrative`, `widgetSize` |

> **Common errors:**
> - **Missing `widgetType` on regular widgets:** validation fails; all widget placements must declare their type.
> - **Missing `widgetTemplate` on regular widgets:** validation fails.
> - **Missing `narrative` on Narrative widgets:** validation fails. Use `cap model narratives list --json` to discover narrative definition IDs.

#### Enum Reference Tables

**Widget Size** (`WidgetSize` EnumDTO):

| Display | EnumDTO |
|---------|---------|
| `25%` | `{"id": 0, "name": "25%"}` |
| `33%` | `{"id": 1, "name": "33%"}` |
| `50%` | `{"id": 2, "name": "50%"}` |
| `66%` | `{"id": 3, "name": "66%"}` |
| `75%` | `{"id": 4, "name": "75%"}` |
| `100%` | `{"id": 5, "name": "100%"}` |

**Widget Type** (`WidgetType` EnumDTO):

| Display | EnumDTO |
|---------|---------|
| `Info Card` | `{"id": 0, "name": "InfoCard"}` |
| `Pie Chart` | `{"id": 1, "name": "PieChart"}` |
| `XY Chart` | `{"id": 2, "name": "XYChart"}` |
| `AI Summary` | `{"id": 3, "name": "AISummary"}` |
| `Table` | `{"id": 4, "name": "Table"}` |
| `Narrative` | `{"id": 5, "name": "Narrative"}` |

**AI Summary Context** (`AISummaryContext` EnumDTO):

| Display | EnumDTO |
|---------|---------|
| `Peers` | `{"id": 0, "name": "Peers"}` |
| `Descendants` | `{"id": 1, "name": "Descendants"}` |

#### Workflow: Modify an Existing Dashboard

The CLI's `get --json` output is directly roundtrippable — no conversion needed:

```bash
# 1. GET the current dashboard
cap templates dashboard-templates get <id> --json > dashboard.json

# 2. Edit dashboard.json (the treeItems array inside dashboardTemplate):
#    - To ADD a widget: append a new treeItem with the parent section's ID
#    - To REMOVE a widget: delete its treeItem (omitted items get deleted!)
#    - To SWAP a widget: change the widgetTemplate.id to the new template
#    - To RESIZE: change widgetSize to a different EnumDTO
#    - Existing items MUST keep their real IDs
#    - ALL widgets MUST have widgetType EnumDTO

# 3. Save (auto-unwraps the dashboardTemplate wrapper)
cap templates dashboard-templates save --file dashboard.json --json
```

**Important:** The save endpoint uses the flat `treeItems` format (same as create). When modifying an existing dashboard, include **all existing items** (with their real IDs) plus any new items. Omitting existing items will delete them.

---

## Related Recipes

- [Create Widget Template](./create-widget-template.md) — Create individual widgets ✅
- [Create Metric Wizard](./create-metric.md) — Create metrics for widgets ✅
- [Verify Data](../reporting/verify-data.md) — Confirm widgets and dashboards return expected data ✅
- [Talk to Your Dashboard](../exploration/talk-to-your-dashboard.md) — Analyze dashboard data ✅
- [Deep Dashboard Analysis](../analysis/deep-dashboard-analysis.md) — Cross-metric pattern discovery ✅
