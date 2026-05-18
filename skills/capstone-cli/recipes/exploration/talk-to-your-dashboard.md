# Recipe: Talk to Your Dashboard

> Answer questions about a specific dashboard's data and insights.

## When to Use

- "Explain the ESG dashboard"
- "What insights from the operations dashboard?"
- "Walk me through [dashboard name]"
- "What does this dashboard show?"

## Required Context

Before starting, Claude needs:
- [ ] **Dashboard** — Which dashboard to analyze (by name)
- [ ] **Focus** — Specific section or overall summary

---

## Step-by-Step

### Step 1: Identify Dashboard

**Purpose:** Find the dashboard configuration.

**Command:**
```bash
cap templates dashboard-templates list --json
```

**If user gave name:**
Find matching dashboard.

**If not specified:**
Present available dashboards for selection.

---

### Step 2: Load Dashboard Structure

**Purpose:** Understand the dashboard's layout, sections, and widget composition.

**Command:**
```bash
cap templates dashboard-templates get <id> --json
```

**Response (nested hierarchy):**
```json
{
  "tenant": "Tharisa",
  "id": "abc-123",
  "name": "ESG Executive Dashboard",
  "useTabSheet": true,
  "nodes": [
    {
      "id": "node-001",
      "name": "Environmental Performance",
      "path": "Environmental Performance",
      "isWidget": false,
      "children": [
        {
          "id": "widget-001",
          "name": "Emissions",
          "isWidget": true,
          "widgetTemplateId": "wt-emissions-123",
          "widgetTemplateName": "Emissions Card",
          "widgetType": "InfoCard",
          "sortOrder": 1
        },
        {
          "id": "widget-002",
          "name": "Energy Mix",
          "isWidget": true,
          "widgetTemplateId": "wt-energy-456",
          "widgetType": "PieChart",
          "sortOrder": 2
        }
      ]
    },
    {
      "id": "node-002",
      "name": "Social Performance",
      "isWidget": false,
      "children": [...]
    }
  ]
}
```

**What Claude now knows:**
- **Sections:** Environmental Performance, Social Performance, etc.
- **Widgets in each section:** Emissions (InfoCard), Energy Mix (PieChart)
- **Widget template IDs:** For bulk lookup of detailed configurations
- **Node IDs:** For targeted `get-data` calls via `--layout-node`

**Example parsing:**
```
Dashboard: ESG Executive
├── Environmental Performance
│   ├── Emissions (InfoCard) → wt-emissions-123
│   ├── Energy Mix (PieChart) → wt-energy-456
│   └── Detailed Metrics
│       └── Emissions Trend (XYChart) → wt-trend-789
└── Social Performance
    └── ...
```

---

### Step 3: Get Org Node Context

**Purpose:** Determine which org node to use for widget data.

**Command:**
```bash
cap masterdata org-nodes list --json
```

Pick the root org node for company-wide view, or a specific location.

---

### Step 3.5: Get Widget Configurations (Optional)

**Purpose:** Understand what metrics each widget tracks before fetching data.

**When to use:** When you need to understand widget configurations or find relationships between widgets (shared metrics).

**Command:**
```bash
# Extract widget template IDs from Step 2 response (comma-separated)
cap templates widget-templates get-bulk -i <wt-id-1>,<wt-id-2>,<wt-id-3> --json
```

**Response:**
```json
{
  "tenant": "Tharisa",
  "widgets": [
    {
      "id": "wt-emissions-123",
      "name": "Emissions Card",
      "widgetType": "InfoCard",
      "dataItems": [
        { "metric": { "id": "...", "name": "Scope 1 Emissions" }, "aggregation": "Sum" }
      ]
    },
    {
      "id": "wt-energy-456",
      "name": "Energy Mix Chart",
      "widgetType": "PieChart",
      "dataItems": [...]
    }
  ]
}
```

**What this enables:**
- Identify all metrics tracked across the dashboard
- Find shared metrics between widgets
- Understand aggregation methods (Sum, Average, etc.)
- Make informed decisions about which data to request

---

### Step 4: Get Widget Data

**For each widget:**

Identify widget type and get appropriate data:

> **Tip:** Use `cap data time-periods list --data-interval <interval>` to discover available period names before fetching widget data.

**Info Card:**
```bash
cap reporting widgets info-card <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
```

**Pie Chart:**
```bash
cap reporting widgets pie-chart <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
```

**XY Chart:** (JSON-only by design — chart structure is the value)
```bash
cap reporting widgets xy-chart <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval month \
  --periods "Jan 25, Feb 25, Mar 25" \
  --json
```

**Table:** (dashboard render rows/columns)
```bash
cap reporting widgets table <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
```

**Required parameters for all type-specific widgets:**
- Widget template ID (positional argument)
- `--org-nodes` — At least one org node
- `--data-interval` — day/week/month/quarter/year
- `--periods` — Comma-separated period names

---

### Step 4 (Alternative): Get All Widget Data at Once

**Purpose:** Get comprehensive dashboard data in a single call for cross-widget analysis.

**When to use this approach:**
- Need to analyze correlations across all widgets
- Dashboard has many widgets (5+) — faster than individual calls
- Want complete data in one response for comprehensive analysis

**Command:**
```bash
cap reporting dashboards get-data <dashboard-template-id> \
  --org-node <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
```

**Optional parameters:**
- `--layout-node <id>` — Start from a specific dashboard section (default: root = all widgets)
- `--scope peers|descendants` — Control which widgets to include (default: descendants)

**Scope behavior:**
- `descendants` (default): All widgets from the layout node and its children
- `peers`: Only widgets from sibling nodes at the same level

**Response:**
CSV data containing all widget values, which Claude can parse and analyze for cross-metric patterns.

**Benefits over widget-by-widget:**
- ✅ Single API call instead of N widget calls
- ✅ Cross-widget correlation analysis possible
- ✅ Complete data context for holistic insights
- ✅ More efficient for large dashboards

**Trade-off:**
- Less granular control than widget-by-widget approach
- CSV format requires parsing for structured analysis

---

### Step 5: Interpret Each Widget

**Info Card:** Key metric highlight
- Current value
- Trend indicator
- vs Target status

**Pie Chart:** Distribution breakdown
- Category proportions
- Largest/smallest segments
- Notable imbalances

**XY Chart:** Time series
- Trend direction
- Comparison between series
- Notable periods

**Table:** Metric-backed grid/list
- Row labels and metadata from `rows[].headers`
- Rendered values from `rows[].cells[].displayValue`
- Row/column counts from `renderedRowCount`, `totalResolvedRowCount`, and `renderedColumnCount`
- Truncation and data issues from `truncated` and `warnings`

---

### Step 6: Synthesize Dashboard Insights

**Format:**
```
## Dashboard Analysis: ESG Overview

### Purpose
This dashboard provides a comprehensive view of environmental,
social, and governance performance for executive leadership.

### Section 1: Environmental Performance

**Emissions Card (Info Card)**
- Current: 1,850 tonnes CO2e
- Trend: ↘️ Down 8% from prior quarter
- Target Status: 🟢 On track

**Energy Mix (Pie Chart)**
- Renewable: 45%
- Natural Gas: 35%
- Grid Electricity: 20%
- Insight: Renewable share growing (+7% this quarter)

**Emissions Trend (XY Chart)**
- 12-month downward trajectory
- Q4 spike (seasonal heating) now resolved
- On pace for year-end target

### Section 2: Social Performance

[Similar analysis for each widget]

### Key Takeaways

1. **Positive:** Emissions and energy efficiency on track
2. **Watch:** Safety training hours below target
3. **Action Needed:** Water consumption requires intervention

### Recommendations

- Focus Q2 on water reduction initiatives
- Accelerate safety training to catch up by Q3
```

---

## Example Interaction

**User:** "What's on the ESG dashboard?"

**Claude:**
```bash
cap templates dashboard-templates list --json
# Found: ESG Executive Dashboard (dash-123)

cap templates dashboard-templates get dash-123 --json
# Structure: 3 sections, 9 widgets (widget-001, widget-002, widget-003, ...)

cap masterdata org-nodes list --json
# Found root org node: org-root

cap reporting widgets info-card widget-001 \
  --org-nodes org-root \
  --data-interval quarter \
  --periods "Q1 FY 25, Q2 FY 25, Q3 FY 25, Q4 FY 25" \
  --json

cap reporting widgets pie-chart widget-002 \
  --org-nodes org-root \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json

cap reporting widgets xy-chart widget-003 \
  --org-nodes org-root \
  --data-interval month \
  --periods "Jan 25, Feb 25, Mar 25" \
  --json
# ... data for each widget
```

**Claude:**

## ESG Executive Dashboard Overview

This dashboard shows key sustainability metrics for leadership review.

### What You're Looking At

**Section 1: Environmental**
- 🔢 **Emissions Card:** 1,850 tonnes (on track for target)
- 📊 **Energy Pie:** 45% renewable, 35% gas, 20% grid
- 📈 **12-Month Trend:** Consistent improvement

**Section 2: Social**
- 🔢 **Safety Score:** 92/100 (good)
- 🔢 **Training Hours:** 165/200 (behind)
- 📈 **Incident Trend:** Flat (goal: declining)

**Section 3: Governance**
- 🔢 **Policy Compliance:** 98% (excellent)
- 📊 **Supplier Assessment:** 78% rated sustainable

### Quick Insights

✅ **Doing Well:** Emissions, compliance, renewable energy
⚠️ **Watch:** Safety training hours behind schedule
🔴 **Concern:** Water usage (not on this dashboard but related)

Want me to drill into any specific widget?

---

## Choosing Your Approach

| Scenario | Recommended Approach |
|----------|---------------------|
| Deep dive into specific widget | Widget-by-widget (Step 4) |
| Quick overview of few widgets (1-3) | Widget-by-widget (Step 4) |
| Comprehensive dashboard analysis | get-data (Step 4 Alternative) |
| Cross-widget correlation analysis | get-data (Step 4 Alternative) |
| Large dashboard (5+ widgets) | get-data (Step 4 Alternative) |

---

## Intelligent Dashboard Selection

When a user asks for analysis without specifying a dashboard (e.g., "Analyze my ESG performance"), Claude should use the nested structure to intelligently select the right dashboard.

### Selection Workflow

**1. List available dashboards:**
```bash
cap templates dashboard-templates list --json
```
Identify candidates by name (e.g., "ESG Executive", "Sustainability Overview").

**2. For promising candidates, get structure:**
```bash
cap templates dashboard-templates get <dashboard-id> --json
```
The nested response reveals what's inside each dashboard.

**3. Evaluate content:**
Check if the dashboard has the widgets the user needs:
- Does it have environmental widgets? ✓
- Does it have the metrics user cares about? ✓
- Is it executive-level or detailed? (consider user preference)

**4. Confirm selection with user:**
```
"I found the ESG Executive Dashboard with:
 - Environmental Performance section (Emissions, Energy Mix, Trend)
 - Social Performance section (Safety Score, Training Hours)
 - Governance section (Policy Compliance)

Is this the dashboard you'd like to analyze?"
```

**5. Proceed with data retrieval:**
After user confirms, continue with get-data or widget-by-widget analysis.

### Example Interaction

**User:** "What's the status of our sustainability metrics?"

**Claude:**
```bash
cap templates dashboard-templates list --json
# Found: ESG Executive (dash-001), Sustainability Overview (dash-002), Operations (dash-003)

cap templates dashboard-templates get dash-001 --json
# ESG Executive: Environmental (3 widgets), Social (2 widgets), Governance (1 widget)

cap templates dashboard-templates get dash-002 --json
# Sustainability Overview: Energy (4 widgets), Water (2 widgets), Waste (2 widgets)
```

**Claude:**
"I found two dashboards relevant to sustainability:

1. **ESG Executive Dashboard** — High-level ESG summary with Emissions, Energy Mix, Safety, and Governance widgets

2. **Sustainability Overview** — Detailed environmental metrics: Energy consumption, Water usage, Waste management

Which would you like me to analyze? Or should I look at both?"

---

## Variations

### Full Dashboard Analysis (using get-data)

For comprehensive analysis of all dashboard data:
```bash
cap reporting dashboards get-data <dashboard-id> \
  --org-node <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
```

Claude can then:
- Parse the CSV for all widget values
- Find correlations (e.g., emissions ↔ energy use)
- Identify common trends across metrics
- Provide holistic insights across all sections

### Section Focus

If user asks about specific section:
```
"Tell me about the social section"
→ Deep dive on just that section's widgets
```

### Comparison View

Compare dashboard data across periods:
```
"How does this dashboard compare to last quarter?"
→ Show changes in each widget's values
```

### Anomaly Highlight

Flag unusual values in dashboard context:
```
"The emissions card shows a spike in the last week
of February. This may warrant investigation."
```

---

## Related Recipes

- [Talk to Your Data](./talk-to-your-data.md) — Drill into specific metrics
- [Find Trends](../analysis/find-trends.md) — Analyze trends deeper
- [Generate Narrative](../reporting/generate-narrative.md) — Write summary
- [Discover Tenant Structure](./discover-tenant-structure.md) — See all dashboards
