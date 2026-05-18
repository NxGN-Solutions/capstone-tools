# Recipe: Deep Dashboard Analysis

> Comprehensive analysis of all dashboard data in a single request for cross-metric pattern discovery.

## When to Use

- "Analyze the entire ESG dashboard"
- "What's the full picture from this dashboard?"
- "Find correlations across all metrics"
- "Are there any patterns across the dashboard?"
- "Give me a holistic view of the dashboard"

## Required Context

Before starting, Claude needs:
- [ ] **Dashboard** — Which dashboard to analyze (by name or ID)
- [ ] **Org Node** — Which location scope (default: company-wide)
- [ ] **Period** — Time frame for analysis

---

## Step-by-Step

### Step 1: Get Dashboard Structure

**Purpose:** Understand the dashboard's layout, sections, and widget composition.

**Command:**
```bash
cap templates dashboard-templates list --json
# Find the dashboard by name

cap templates dashboard-templates get <dashboard-id> --json
```

**Response (nested hierarchy):**
The response now includes full nested structure:
- **Sections** with names and paths
- **Widgets** with template IDs and types
- **Node IDs** for targeted data retrieval

**Example parsing:**
```
Dashboard: ESG Executive (dash-123)
├── Environmental Performance (node-001)
│   ├── Emissions (InfoCard) → wt-emissions-123
│   ├── Energy Mix (PieChart) → wt-energy-456
│   └── Detailed Metrics (node-001a)
│       └── Emissions Trend (XYChart) → wt-trend-789
└── Social Performance (node-002)
    └── Safety Score (InfoCard) → wt-safety-321
```

**Extract:**
- Section names and their widget composition
- Widget template IDs for bulk lookup
- Node IDs for section-focused data retrieval
- Overall dashboard focus (ESG, Operations, Financial, etc.)

---

### Step 1.5: Fetch Widget Configurations

**Purpose:** Understand what metrics each widget tracks and how data is aggregated.

**Command:**
```bash
# Use widget template IDs from Step 1
cap templates widget-templates get-bulk -i wt-emissions-123,wt-energy-456,wt-trend-789 --json
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
        { "metric": { "id": "m-001", "name": "Scope 1 Emissions" }, "aggregation": "Sum" },
        { "metric": { "id": "m-002", "name": "Scope 2 Emissions" }, "aggregation": "Sum" }
      ]
    },
    {
      "id": "wt-energy-456",
      "name": "Energy Mix Chart",
      "widgetType": "PieChart",
      "dataItems": [
        { "metric": { "id": "m-003", "name": "Renewable Energy" }, "aggregation": "Sum" },
        { "metric": { "id": "m-004", "name": "Grid Electricity" }, "aggregation": "Sum" }
      ]
    }
  ]
}
```

**What this enables:**
- **Metric Discovery:** Identify all metrics tracked across the dashboard
- **Relationship Mapping:** Find shared metrics between widgets (e.g., emissions appearing in multiple widgets)
- **Aggregation Understanding:** Know if data is summed, averaged, or latest value
- **Cross-Widget Analysis:** Correlate metrics from different widgets (e.g., Energy → Emissions)

---

### Step 2: Get Org Node Context

**Purpose:** Determine which org node to use for data.

**Command:**
```bash
cap masterdata org-nodes list --json
```

Pick the root org node for company-wide view, or a specific location for focused analysis.

---

### Step 3: Get Comprehensive Dashboard Data

**Purpose:** Retrieve all widget data in a single call.

> **Tip:** Use `cap data time-periods list --data-interval <interval>` to discover available period names.

**Command:**
```bash
cap reporting dashboards get-data <dashboard-id> \
  --org-node <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
```

**Response:** CSV data containing all widget values across the entire dashboard.

**Alternative — Section Focus:**
To analyze just one section of a larger dashboard:
```bash
cap reporting dashboards get-data <dashboard-id> \
  --layout-node <section-node-id> \
  --scope descendants \
  --org-node <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
```

---

### Step 4: Analyze Cross-Metric Patterns

**Purpose:** Find relationships and correlations across all metrics.

**Look for:**
- **Correlations** — Do emissions rise with energy use? Does safety training correlate with fewer incidents?
- **Common trends** — Are multiple metrics improving/declining together?
- **Inverse relationships** — When one metric goes up, does another go down?
- **Lag patterns** — Does a change in one metric predict a change in another?
- **Anomalies** — Metrics that behave differently than expected

**Pattern types:**
| Pattern | Example |
|---------|---------|
| Direct correlation | Energy use ↑ → Emissions ↑ |
| Inverse correlation | Training hours ↑ → Incidents ↓ |
| Leading indicator | Complaints ↑ → Employee turnover ↑ (3 months later) |
| Seasonal pattern | Water use peaks in summer across all metrics |
| Outlier | All metrics on track except water consumption |

---

### Step 5: Synthesize Findings

**Format:**
```
## Deep Analysis: ESG Overview Dashboard

### Dashboard Purpose
This dashboard provides a comprehensive view of environmental,
social, and governance performance for executive leadership.

### Cross-Metric Patterns Discovered

**1. Energy-Emissions Correlation (Strong)**
- Correlation: 0.92 (near-perfect)
- When energy consumption drops 5%, emissions drop ~4.6%
- Implication: Energy efficiency directly drives emissions reduction

**2. Training-Safety Relationship (Moderate)**
- Training hours have lagged behind target
- Safety score dipped in the same period
- Possible cause-effect relationship

**3. Governance Leading Indicator**
- Policy compliance tends to improve 1 quarter before audit scores
- Current high compliance (98%) suggests Q2 audit will be strong

### Section-by-Section Summary

**Environmental**
- All metrics trending positive
- Emissions on track for annual target
- Key driver: Renewable energy investment

**Social**
- Mixed performance
- Training hours behind (-17.5%)
- Safety score declined from 95 to 88

**Governance**
- Strong overall
- Compliance at 98%
- Supplier assessment improving

### Key Insights

1. **Pattern:** Environmental metrics correlate strongly with energy decisions
2. **Risk:** Social metrics showing early warning signs
3. **Opportunity:** Governance strength could offset social metric concerns

### Recommendations

1. Accelerate training program to address safety correlation
2. Maintain energy efficiency investments — direct emissions impact
3. Leverage governance strength in stakeholder communications
```

---

## Example Interaction

**User:** "Give me a full analysis of the ESG dashboard"

**Claude:**
```bash
cap templates dashboard-templates list --json
# Found: ESG Executive Dashboard (dash-123)

cap templates dashboard-templates get dash-123 --json
# Full nested structure with sections and widgets:
# - Environmental Performance: Emissions (wt-123), Energy Mix (wt-456), Trend (wt-789)
# - Social Performance: Safety Score (wt-321), Training (wt-654)
# - Governance: Compliance (wt-987)

cap templates widget-templates get-bulk -i wt-123,wt-456,wt-789,wt-321,wt-654,wt-987 --json
# Widget configurations showing:
# - Metrics tracked: Scope 1 Emissions, Scope 2 Emissions, Renewable Energy, etc.
# - Aggregation methods: Sum, Average
# - Shared metrics between widgets

cap masterdata org-nodes list --json
# Found root org node: org-root

cap reporting dashboards get-data dash-123 \
  --org-node org-root \
  --data-interval quarter \
  --periods "Q1 FY 25, Q2 FY 25, Q3 FY 25, Q4 FY 25" \
  --json
# Full CSV with all widget data
```

**Claude:**

## Deep Analysis: ESG Executive Dashboard

### Cross-Metric Patterns Discovered

**Energy-Emissions Correlation** (Strong positive: 0.92)
Your emissions track almost perfectly with energy consumption. This Q1 data shows:
- Energy: 485 MWh (down 3% from target)
- Emissions: 1,850 tonnes (down 7.5% from target)

This is good news — your energy efficiency investments are paying dividends.

**Training-Safety Warning Sign**
I notice both training hours and safety scores are underperforming:
- Training: 165/200 hours (-17.5%)
- Safety: 88/95 score (-7.4%)

These often correlate. Consider whether reduced training is contributing to safety score decline.

**Governance as a Leading Indicator**
Your 98% policy compliance is excellent. Historically, high compliance precedes strong audit performance. Q2 audit outlook is positive.

### Action Items

1. **Urgent:** Investigate training-safety connection; accelerate training
2. **Monitor:** Energy efficiency — current path is working
3. **Communicate:** Governance strength to stakeholders

Want me to drill deeper into any of these patterns?

---

## Variations

### Time Series Analysis

Compare cross-metric patterns over multiple periods:
```bash
cap reporting dashboards get-data <dashboard-id> \
  --org-node <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 24, Q2 FY 24, Q3 FY 24, Q4 FY 24, Q1 FY 25, Q2 FY 25, Q3 FY 25, Q4 FY 25" \
  --json
```

Look for how correlations change over time.

### Section-Only Analysis

Focus on a specific dashboard section:
```bash
cap reporting dashboards get-data <dashboard-id> \
  --layout-node <section-node-id> \
  --scope descendants \
  --org-node <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
```

### Multi-Location Correlation

Check if patterns hold across locations:
```bash
# Run for each location
cap reporting dashboards get-data <dashboard-id> \
  --org-node <location-1-id> --data-interval quarter --periods "Q1 FY 25" --json

cap reporting dashboards get-data <dashboard-id> \
  --org-node <location-2-id> --data-interval quarter --periods "Q1 FY 25" --json
```

Compare correlation strength by location.

---

## Related Recipes

- [Talk to Your Dashboard](../exploration/talk-to-your-dashboard.md) — Widget-by-widget analysis
- [Find Trends](./find-trends.md) — Time-series trend analysis
- [Compare Operations](./compare-operations.md) — Location comparisons
- [Dashboard Comparison](./dashboard-comparison.md) — Period-over-period analysis
