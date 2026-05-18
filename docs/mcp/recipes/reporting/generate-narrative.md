# Recipe: Generate Narrative

> Generate an AI-written performance summary suitable for reports.

## When to Use

- "Write a summary of Q1 performance"
- "Executive summary for the board"
- "Narrative report for [period]"
- "Put this data into words"

## Prompt Shortcut

```
/generate-narrative dashboardName="ESG Executive Dashboard" period="Q4 2024"
```

Both arguments are required.

---

## Required Context

| What's Needed | Source | If Missing |
|--------------|--------|------------|
| Dashboard name | User request | Ask: "Which dashboard should I summarize?" |
| Time period | User request | Ask: "Which period should I cover?" |
| Audience | User request | Default: management-level professional tone |
| Comparison basis | User request | Default: prior period if available |

---

## Workflow

### 1. Find the dashboard

```
templates_dashboards_list()
```

Match the dashboard name from `gridRows`. Extract the `id`.

### 2. Get dashboard data

```
reporting_dashboards_getData({
  dashboardTemplateId: "<dashboard-guid>",
  orgNodeId: "<root-org-guid>",
  periodType: "quarter",
  periodCount: 4
})
```

Returns CSV data with all widget values and metrics.

### 3. Get comparison data

If comparison is useful, the query above already includes multiple periods. Calculate changes between the current and prior period.

### 4. Get AI insights (optional)

```
apps_widget_aiSummary({
  dashboardTemplateId: "<dashboard-guid>",
  orgNodeId: "<root-org-guid>",
  startDate: "2024-10-01",
  endDate: "2025-03-31",
  timePeriodType: "quarter"
})
```

Returns pre-analyzed trend patterns for the dashboard.

### 5. Structure the narrative

Write a professional report with these sections:

**Executive Summary** (2-3 sentences)
High-level overview of performance for the period.

**Key Metrics Performance**
For each major metric: current value, trend, comparison to target or prior period, status.

**Trends and Changes**
Notable movements since last period. Explain significant increases or decreases.

**Areas of Concern**
Metrics below target or trending negatively. Root cause analysis where possible.

**Recommendations**
Actionable suggestions based on the data. Priority order by impact.

### 6. Apply tone and formatting

Adjust language based on audience:
- **Board:** Concise, strategic implications, key risks
- **Management:** Operational detail, specific metrics, action items
- **External/Public:** Achievements framing, framework references (GRI, SASB)

---

## Present Results

```
## Q1 FY 2025 ESG Performance Summary

In the first quarter of FY 2025, Acme Corporation continued its
trajectory toward sustainability goals, achieving notable reductions
in greenhouse gas emissions while maintaining strong safety performance.
Water consumption and waste diversion rates present opportunities for
improvement.

### Key Highlights
- Emissions reduced 8% from Q4 FY 2024 (22% year-over-year)
- Zero recordable safety incidents for third consecutive quarter
- Renewable energy adoption reached 45%, up from 38%

### Areas Requiring Attention
- Water consumption increased 12% due to expanded operations at Site C
- Waste diversion rate declined to 68% (below 75% target)

### Recommendations
1. Commission water recycling system at Site C (Q2 priority)
2. Engage suppliers on sustainable packaging alternatives
3. Continue safety training program expansion
```

---

## Example Interaction

**User:** "Write an executive summary of Q1 performance"

**Claude:**
```
templates_dashboards_list()
→ Found: ESG Executive Dashboard (dash-abc123)

model_orgNodes_list()
→ Root org node: Corporate (org-root)

reporting_dashboards_getData({
  dashboardTemplateId: "dash-abc123",
  orgNodeId: "org-root",
  periodType: "quarter",
  periodCount: 4
})
→ Full CSV with 4 quarters of data

apps_widget_aiSummary({
  dashboardTemplateId: "dash-abc123",
  orgNodeId: "org-root",
  startDate: "2024-04-01",
  endDate: "2025-03-31",
  timePeriodType: "quarter"
})
→ AI-generated insights
```

**Claude:**

## Q1 FY 2025 ESG Performance Summary

Acme Corporation delivered strong environmental performance in Q1 FY 2025, with greenhouse gas emissions declining 8% quarter-over-quarter to 1,850 tonnes CO2e. Renewable energy adoption reached a record 45% of the total energy mix. Social metrics present a mixed picture, with training hours behind schedule at 82.5% of target.

### Environmental Performance
- **Emissions:** 1,850 t CO2e (-8% QoQ, -22% YoY) — **On Track**
- **Renewable Energy:** 45% of mix (+7 percentage points) — **Exceeding Target**
- **Water Use:** 42,000 kL (+12% due to Site C expansion) — **At Risk**

### Social Performance
- **Safety Score:** 92/100 — Good but declining from 95
- **Training Hours:** 165/200 target hours (-17.5%) — **Behind**
- **Note:** Training shortfall may be contributing to safety score decline

### Governance
- **Compliance:** 98% — Excellent
- **Supplier Assessment Coverage:** +5% quarter-over-quarter

### Recommendations
1. **Priority:** Accelerate training program to close 35-hour gap
2. **Investigate:** Safety-training correlation — root cause analysis
3. **Plan:** Water management strategy for expanded Site C operations
4. **Maintain:** Current energy efficiency investments showing strong ROI

---

## Variations

### Board-level summary

Ultra-concise, 3-5 sentences focusing on strategic positioning and key risks.

### Detailed report

Expand with data tables for each metric, location breakdowns, and trend charts:

```
apps_dashboard_render({
  dashboardTemplateId: "<dashboard-guid>",
  orgNodeId: "<root-org-guid>",
  startDate: "2024-04-01",
  endDate: "2025-03-31",
  timePeriodType: "quarter"
})
```

### Framework-aligned narrative

Structure the narrative to align with a specific reporting framework (GRI, SASB) using `model_frameworks_list()` to get the disclosure topics.

---

## Edge Cases

- **Sparse data:** If the dashboard has many empty widgets, note data gaps and focus the narrative on available metrics.
- **No prior period:** First reporting period — skip comparisons and focus on baseline establishment.
- **Multiple dashboards:** If user doesn't specify which dashboard, present options from `templates_dashboards_list`.

---

## CLI Equivalent

[CLI: Generate Narrative](../../../cli/recipes/reporting/generate-narrative.md)
