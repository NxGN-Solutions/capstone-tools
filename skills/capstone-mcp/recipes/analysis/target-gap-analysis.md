# Recipe: Target Gap Analysis

> Compare actual performance against targets to identify gaps.

## When to Use

- "Are we on track for our targets?"
- "How close are we to the emissions goal?"
- "Target status report"
- "Will we meet our commitments?"

## Prompt Shortcut

```
/target-gap-analysis metricNames="GHG Emissions,Energy Use" period="Q1 FY 25"
```

Both arguments are optional — defaults to all metrics with targets in the most recent period.

---

## Required Context

| What's Needed | Source | If Missing |
|--------------|--------|------------|
| Metrics with targets | `capstone://model/metrics` resource or `model_metrics_list` | Ask: "Which metrics or goals should I check?" |
| Time period | User request or `data_availability` | Ask: "Which time period?" |
| Report template | `templates_spreadsheetReports_list` | Auto-select first matching template |
| Scope (location) | `capstone://model/organization` resource | Default: company-wide (root org node) |

---

## Workflow

### 1. Find metrics with targets

```
model_metrics_list()
```

Filter the response `gridRows` to identify metrics that have target values configured. Note their IDs and names.

### 2. Resolve report template

```
templates_spreadsheetReports_list()
```

Find a report template that includes the target metrics. Use the template `id` for the data query.

### 3. Get actual values

```
data_computedValues_list({
  templateId: "<report-template-id>",
  timePeriodNames: "Q1 FY 25",
  dataInterval: 3,
  orgNodeIds: "<root-org-node-id>"
})
```

Parse the response: extract values from rows where `isDataRow: true`, aligning with `timePeriodColumns` by index.

### 4. Calculate gaps

For each metric, compute:
- **Absolute gap:** Actual - Target
- **Percentage gap:** (Actual - Target) / Target * 100
- **Status:** On Track (within 10%), At Risk (10-25% behind), Behind (>25%)

### 5. Present status report

Format as a summary table with status counts and individual metric details.

---

## Present Results

```
## Target Status Report: Q1 FY 2025

### Summary
On Track: 4 metrics | At Risk: 2 metrics | Behind: 1 metric

### Details
| Metric | Target | Actual | Gap | Status |
|--------|--------|--------|-----|--------|
| GHG Emissions | 2,000 t | 1,850 t | -7.5% | On Track |
| Energy Use | 500 MWh | 485 MWh | -3.0% | On Track |
| Training Hours | 200 hrs | 165 hrs | -17.5% | At Risk |

### Year-End Projections
At current rate, which targets will be met/missed?

### Recommendations
Actionable steps to close gaps.
```

---

## Example Interaction

**User:** "Are we on track for our emission targets?"

**Claude:**
```
model_metrics_list({ search: "emissions" })
→ Found: GHG Emissions (met-abc123, target: 8,000 tonnes/year)

templates_spreadsheetReports_list()
→ Found: ESG Report (rpt-def456)

data_computedValues_list({
  templateId: "rpt-def456",
  timePeriodNames: "Q1 FY 25",
  dataInterval: 3
})
→ Q1 Actual: 1,850 tonnes
```

**Claude:**

## Emissions Target Status: Q1 FY 2025

**Target:** 8,000 tonnes CO2e (annual)
**Q1 Budget:** 2,000 tonnes (25% of annual)
**Q1 Actual:** 1,850 tonnes

**Status:** On Track (-7.5% under budget)

| Quarter | Budget | Actual | Status |
|---------|--------|--------|--------|
| Q1      | 2,000  | 1,850  | -7.5% |
| Q2      | 2,000  | —      | Upcoming |
| Q3      | 2,000  | —      | Upcoming |
| Q4      | 2,000  | —      | Upcoming |

**Projection:** At current pace, annual emissions will be ~7,400 tonnes, 7.5% under the 8,000 target.

---

## Variations

### Location-Level Targets

Show target status by location using `data_computedValues_list` with specific `orgNodeIds` for each site.

### Multi-Target View

When metrics have multiple targets (annual, YoY, monthly cap), show each in a separate column.

---

## Edge Cases

- **No targets configured:** Report which metrics lack targets and suggest setting them up.
- **Missing data:** Note which metrics have no actuals for the period.
- **Calculation metrics:** Targets on calculations require all input dependencies to have data.

---

## CLI Equivalent

[CLI: Target Gap Analysis](../../../capstone-cli/recipes/analysis/target-gap-analysis.md)
