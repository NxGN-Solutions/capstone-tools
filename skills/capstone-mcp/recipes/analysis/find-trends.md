# Recipe: Find Trends

> Identify how metrics are changing over time.

## When to Use

- "How are emissions trending?"
- "Show the trend for energy usage"
- "Is safety improving?"
- "What's the direction of [metric]?"

## Prompt Shortcut

```
/find-trends metricNames="GHG Emissions" periods="last 12 months"
```

Both arguments are optional — defaults to key metrics over available historical periods.

---

## Required Context

| What's Needed | Source | If Missing |
|--------------|--------|------------|
| Metric(s) | `capstone://model/metrics` resource or `model_metrics_list` | Ask: "Which metric should I analyze?" |
| Time range | User request or `data_availability` | Default: 12 most recent periods |
| Report template | `templates_spreadsheetReports_list` | Auto-select first template |
| Location scope | User request | Default: company-wide (root org node) |

---

## Workflow

### 1. Identify metrics

Check `capstone://model/metrics` resource first, or call:

```
model_metrics_list({ search: "emissions" })
```

Resolve metric names to IDs from the response `gridRows`.

### 2. Get time periods

```
data_timePeriods_list({ dataInterval: "month" })
```

Identify available periods for the metric's data interval.

### 3. Resolve report template

```
templates_spreadsheetReports_list()
```

Find a template including the target metrics.

### 4. Get time series data

```
data_computedValues_list({
  templateId: "<report-template-id>",
  periodType: "month",
  periodCount: 12,
  dataInterval: 2
})
```

Parse the response: extract the metric's values across all `timePeriodColumns`.

### 5. Analyze trend

Calculate:
- **Direction:** Increasing, Decreasing, or Stable
- **Rate of change:** % per period (compare first and last values)
- **Volatility:** Standard deviation of period-over-period changes

### 6. Identify patterns

Look for:
- **Seasonality:** Recurring peaks/troughs (e.g., winter heating)
- **Inflection points:** Where the trend direction changed
- **Acceleration:** Is the rate of change itself changing?

### 7. Project future

Estimate:
- Next period value based on current trajectory
- Time to reach target (if target exists)
- Confidence level based on data consistency

---

## Present Results

```
## Trend Analysis: GHG Emissions (12 months)

**Direction:** Decreasing
**Rate:** -2.5% per month
**Total Change:** -26% year-over-year

| Period | Value | Change |
|--------|-------|--------|
| Jan 24 | 890 t | - |
| Feb 24 | 850 t | -4.5% |
| Mar 24 | 820 t | -3.5% |
| ...    | ...   | ...    |
| Dec 24 | 660 t | -3.0% |

**Observations:**
- Consistent downward trend throughout the year
- Largest drop in Q3 (efficiency program launched)
- Slight uptick in Nov-Dec (seasonal heating demand)

**Projection:** At current rate, emissions will reach 580 t by Jun 2025.
```

---

## Example Interaction

**User:** "How are our emissions trending?"

**Claude:**
```
model_metrics_list({ search: "emissions" })
→ Found: Total GHG Emissions (met-abc123)

templates_spreadsheetReports_list()
→ Found: ESG Report (rpt-def456)

data_computedValues_list({
  templateId: "rpt-def456",
  periodType: "month",
  periodCount: 12,
  dataInterval: 2
})
→ Returns 12 months: [890, 850, 820, 810, 790, 760, 720, 700, 690, 680, 670, 660]
```

**Claude:**

## Emissions Trend: 12-Month Analysis

**Overall:** Emissions decreased 26% over the past year (890 → 660 tonnes/month).

| Quarter | Avg Monthly | Change |
|---------|------------|--------|
| Q1 2024 | 853 t | — |
| Q2 2024 | 787 t | -7.7% |
| Q3 2024 | 703 t | -10.7% |
| Q4 2024 | 670 t | -4.7% |

**Observations:**
- Strongest improvement in Q3 (-10.7%) — coincides with efficiency program launch
- Rate of improvement is slowing (Q4 only -4.7%)
- Seasonal uptick in December (heating)

**Projection:** At current pace, monthly emissions will reach ~600 tonnes by mid-2025. To maintain the 2024 improvement rate, new initiatives may be needed.

Would you like me to visualize this trend?

```
apps_widget_xyChart({
  widgetTemplateId: "<emissions-trend-widget>",
  orgNodeIds: "<root-org-id>",
  startDate: "2024-01-01",
  endDate: "2024-12-31",
  timePeriodType: "month"
})
```

---

## Variations

### Short-term trend

Focus on last 3 months or week-over-week changes.

### Trend comparison

Compare trends across multiple locations or metrics side-by-side.

### Seasonality detection

Identify recurring patterns by comparing same months across multiple years.

---

## Edge Cases

- **Sparse data:** If many periods have missing values, note low confidence in trend.
- **New metric:** Less than 3 data points — insufficient for trend analysis. Report available data and suggest waiting for more.
- **Step changes:** A sudden jump may indicate a reporting methodology change, not a real trend.

---

## CLI Equivalent

[CLI: Find Trends](../../../capstone-cli/recipes/analysis/find-trends.md)
