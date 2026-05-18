# Recipe: Talk to Dashboard

> Answer questions about a specific dashboard's data and insights.

## When to Use

- "Explain the ESG dashboard"
- "What insights from the operations dashboard?"
- "Walk me through [dashboard name]"
- "What does this dashboard show?"

## Prompt Shortcut

```
/talk-to-dashboard dashboardName="ESG Executive Dashboard"
```

Dashboard name is required.

---

## Required Context

| What's Needed | Source | If Missing |
|--------------|--------|------------|
| Dashboard name | User request | Ask: "Which dashboard should I analyze?" |
| Org node | `capstone://model/organization` resource or `model_orgNodes_list` | Default: root org node (company-wide) |
| Time period | User request or auto-detect | Default: most recent quarter |

---

## Workflow

### 1. Find the dashboard

```
templates_dashboards_list()
```

Match the user's dashboard name against `gridRows[].name`. Extract the `id`.

If name is ambiguous, present options for user selection.

### 2. Get dashboard data

```
reporting_dashboards_getData({
  dashboardTemplateId: "<dashboard-guid>",
  orgNodeId: "<root-org-guid>",
  periodType: "quarter",
  periodCount: 4
})
```

Returns CSV data with all widget values.

### 3. Interpret each section

For each data section in the CSV:
- What metric(s) are shown?
- What time period does it cover?
- Current status (good/warning/critical)?

### 4. Get AI insights (optional)

```
apps_widget_aiSummary({
  dashboardTemplateId: "<dashboard-guid>",
  orgNodeId: "<root-org-guid>",
  startDate: "2024-07-01",
  endDate: "2025-03-31",
  timePeriodType: "quarter"
})
```

Returns AI-generated trend analysis and patterns.

### 5. Synthesize cross-section insights

Look for:
- Correlations between metrics (energy use ↔ emissions)
- Leading indicators (training hours → safety score)
- Areas needing attention

### 6. Highlight key takeaways

Provide top positives, top concerns, and recommended actions.

---

## Present Results

```
## Dashboard Analysis: ESG Executive

### Section 1: Environmental Performance
- Emissions: 1,850 t (down 8% from prior quarter, on track)
- Energy Mix: 45% renewable, 35% gas, 20% grid
- Trend: Consistent improvement over 12 months

### Section 2: Social Performance
- Safety Score: 92/100 (good)
- Training Hours: 165/200 (behind target)

### Key Takeaways
Doing Well: Emissions, compliance, renewable energy
Watch: Safety training hours behind schedule
Action Needed: Accelerate training in Q2
```

---

## Example Interaction

**User:** "What does the ESG dashboard show?"

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
→ CSV with all widget data across 4 quarters
```

**Claude:**

## ESG Executive Dashboard: Analysis

### Environmental (Strong)
- **Emissions:** 1,850 t in Q1 — down 8% from Q4, 22% from last year
- **Energy:** 45% renewable, up from 38% last quarter
- **Trend:** All environmental metrics improving consistently

### Social (Mixed)
- **Safety:** Score of 92/100 — good, but down from 95 last quarter
- **Training:** 165 of 200 hours completed — 17.5% behind schedule
- **Connection:** Training shortfall may be affecting safety scores

### Governance (Strong)
- **Compliance:** 98% — excellent
- **Supplier Assessment:** Improving (+5% coverage)

### Top Actions
1. **Urgent:** Accelerate training program (165/200 hrs)
2. **Monitor:** Safety score decline (92 vs 95)
3. **Celebrate:** Emissions and renewable energy targets on track

Would you like me to render the actual dashboard, or drill into any section?

---

## Variations

### Visual dashboard

Render the actual dashboard with interactive charts:

```
apps_dashboard_render({
  dashboardTemplateId: "<dashboard-guid>",
  orgNodeId: "<root-org-guid>",
  startDate: "2024-04-01",
  endDate: "2025-03-31",
  timePeriodType: "quarter"
})
```

### Section focus

If user asks about a specific section, focus analysis there.

### Period comparison

"How does this dashboard compare to last quarter?" — Call `reporting_dashboards_getData` for both periods and show changes.

---

## Edge Cases

- **Multiple matching dashboards:** Present options with descriptions to help user choose.
- **Empty dashboard:** Template exists but no data — explain that data needs to be captured first.
- **Unknown org node:** If no root org node is obvious, ask which location to view.

---

## CLI Equivalent

[CLI: Talk to Your Dashboard](../../../cli/recipes/exploration/talk-to-your-dashboard.md)
