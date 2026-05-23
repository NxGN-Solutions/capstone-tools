# Recipe: Talk to Your Data

> Answer natural language questions about specific metrics by retrieving and analyzing data.

## When to Use

- "Analyze our emissions for Q1"
- "What's our energy usage looking like?"
- "How much water did we use last year?"
- "Tell me about [metric] at [location]"

This recipe has **no prompt shortcut** — it's a direct tool workflow for answering open-ended metric questions.

---

## Required Context

| What's Needed | Source | If Missing |
|--------------|--------|------------|
| Metric(s) | `capstone://model/metrics` resource or `model_metrics_list` | Ask: "What specifically do you want to measure? (e.g., electricity, water, safety)" |
| Time range | User request or `data_availability` | Ask: "Which time period? (e.g., Q1 2025, last 6 months)" |
| Location scope | `capstone://model/organization` resource or `model_orgNodes_list` | Default: root org node (company-wide) |
| Report template | `templates_spreadsheetReports_list` | Auto-select first matching template |

---

## Workflow

### 1. Parse user question

Extract intent and parameters from natural language:
- **Metrics** mentioned (emissions, energy, water, safety)
- **Time range** (Q1, last 6 months, 2024, this year)
- **Scope** (all sites, Site A, Region 1)

**If vague about metrics:**
```
"You asked about 'emissions.' I found several emission-related metrics:
1. Scope 1 Emissions (direct)
2. Scope 2 Emissions (electricity)
3. Total GHG Emissions

Which would you like to analyze?"
```

### 2. Resolve metric IDs

Check `capstone://model/metrics` resource first. If not available, call:

```
model_metrics_list({ search: "emissions" })
```

**Match strategies:**
- Exact name match
- Keyword match (emissions → *Emissions*)
- Discipline match (environmental metrics)

### 3. Resolve location scope

If user specified a location, find the org node:

```
model_orgNodes_list()
```

Match by name from the response `gridRows`. If no location specified, use the root org node (level 0).

### 4. Resolve report template

Find a report template that includes the target metrics:

```
templates_spreadsheetReports_list()
```

Pick the template most likely to include the metric. If uncertain, call `templates_spreadsheetReports_get({ templateId: "..." })` to inspect its configuration.

**If no suitable template exists:** "I couldn't find a report template that includes [metric]. Would you like me to check input values directly instead?"

### 5. Get computed values

Retrieve the aggregated data:

```
data_computedValues_list({
  templateId: "<report-template-id>",
  periodType: "quarter",
  periodCount: 4,
  dataInterval: 3,
  orgNodeIds: "<org-node-id>"
})
```

**Or for specific periods:**
```
data_computedValues_list({
  templateId: "<report-template-id>",
  timePeriodNames: "Q1 FY 25,Q2 FY 25,Q3 FY 25,Q4 FY 25",
  dataInterval: 3,
  orgNodeIds: "<org-node-id>"
})
```

**Response parsing:** Extract data rows (`isDataRow: true`) and align values with `timePeriodColumns` by index.

### 6. Analyze data

Calculate insights beyond raw numbers:

| Analysis Type | Technique |
|---------------|-----------|
| Period-over-period change | `(current - previous) / previous * 100` |
| High/low values | Sort values across periods |
| Percentage of total | If multiple metrics, calculate shares |
| Trend direction | Increasing, decreasing, or stable based on trajectory |
| Anomalies | Values more than 2 standard deviations from mean |
| Seasonality | Recurring patterns across same periods in different years |

### 7. Synthesize answer

Present insights in conversational format with:
- **Summary** — One-line answer to the user's question
- **Key findings** — 2-3 notable observations from the data
- **Trend** — Direction and rate of change
- **Caveats** — Note any missing data or limitations

---

## Present Results

```
## Emissions Analysis: Q1 2025

**Summary:** Total GHG emissions for Q1 2025 were 2,450 tonnes CO2e,
down 8% from Q4 2024 (2,665 tonnes).

**Key Findings:**
- January was the highest month (890 tonnes), likely due to heating demand
- March showed significant reduction (720 tonnes) following the efficiency program rollout
- Site A contributed 45% of total emissions

**Trend:** Emissions are trending downward at ~3% per month

**Note:** This analysis is based on validated data only.
February data for Site C is still pending approval.
```

---

## Example Interaction

**User:** "How are our emissions doing this year?"

**Claude:** Let me analyze your emissions data for 2025.

```
model_metrics_list({ search: "emissions" })
→ Found: Total GHG Emissions (met-abc123), Scope 1 (met-def456), Scope 2 (met-ghi789)

templates_spreadsheetReports_list()
→ Found: ESG Quarterly Report (rpt-jkl012) — includes environmental metrics

data_computedValues_list({
  templateId: "rpt-jkl012",
  timePeriodNames: "Jan 2025,Feb 2025,Mar 2025",
  dataInterval: 2
})
→ Returns grid with GHG Emissions values: Jan=890, Feb=840, Mar=720
```

**Claude:**

## Emissions Overview: Q1 2025

**Total emissions:** 2,450 tonnes CO2e

| Month | Emissions | Change |
|-------|-----------|--------|
| January | 890 tonnes | — |
| February | 840 tonnes | -5.6% |
| March | 720 tonnes | -14.3% |

**Trend:** Emissions are decreasing, with a 19% reduction from January to March. At this rate, Q2 could see a further 15% reduction.

**Breakdown by source:**
- Scope 1 (direct): 1,200 tonnes (49%)
- Scope 2 (electricity): 1,250 tonnes (51%)

Would you like me to drill into a specific location or emission type?

---

## Variations

### Multiple Metrics

If user asks about multiple metrics, present each metric's analysis and compare where meaningful. Identify correlations (e.g., energy use vs emissions).

### Time Series Analysis

For trend requests, extend the period range to 12+ months using `periodType: "month"`, `periodCount: 12`. Calculate trend line and identify seasonality.

### Drill-Down

If user wants more detail after initial analysis:
- Break down by location using specific `orgNodeIds`
- Break down by sub-metric
- Show period-over-period comparison

### Visual Answer

When the user asks to "show" data, combine data retrieval with an MCP App:

```
apps_widget_xyChart({
  widgetTemplateId: "<emissions-trend-widget>",
  orgNodeIds: "<root-org-id>",
  startDate: "2025-01-01",
  endDate: "2025-03-31",
  timePeriodType: "month"
})
```

---

## Edge Cases

- **Vague metrics:** Multiple matches — present options and ask the user to choose.
- **No data for period:** Report that no data exists and suggest checking data availability or a different period.
- **No suitable template:** Suggest checking input values directly via `data_inputValues_list` instead.
- **Too many results:** Use `limit` parameter or narrow by `disciplineNodeIds` or `orgNodeIds`.

---

## Related Recipes

- [Compare Operations](../analysis/compare-operations.md) — Compare two locations
- [Find Trends](../analysis/find-trends.md) — Detailed trend analysis with projections
- [Spot Anomalies](../analysis/spot-anomalies.md) — Identify unusual values
- [Discover Structure](./discover-structure.md) — See what metrics are available
- [Talk to Dashboard](./talk-to-dashboard.md) — Dashboard-focused analysis

---

## CLI Equivalent

[CLI: Talk to Your Data](../../../cli/recipes/exploration/talk-to-your-data.md)
