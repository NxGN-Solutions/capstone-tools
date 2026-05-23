# Recipe: Spot Anomalies

> Detect statistical outliers that may need attention.

## When to Use

- "Any unusual values last month?"
- "What looks off?"
- "Are there any outliers?"
- "Data quality check for anomalies"

## Prompt Shortcut

```
/spot-anomalies period="last 12 months" sensitivity="medium"
```

Both arguments are optional — defaults to available data range with medium sensitivity.

---

## Required Context

| What's Needed | Source | If Missing |
|--------------|--------|------------|
| Metrics | `capstone://model/metrics` resource or `model_metrics_list` | Auto-use all numeric metrics |
| Report template | `templates_spreadsheetReports_list` | Auto-select first template |
| Historical data | `data_computedValues_list` with multiple periods | Need at least 6 periods for baseline |
| Baseline period count | User request | Default: 12 periods |

---

## Workflow

### 1. Resolve report template

```
templates_spreadsheetReports_list()
```

Pick a template covering the metrics of interest.

### 2. Get current period values

```
data_computedValues_list({
  templateId: "<report-template-id>",
  timePeriodNames: "Mar 2025",
  dataInterval: 2
})
```

### 3. Get historical baseline

```
data_computedValues_list({
  templateId: "<report-template-id>",
  periodType: "month",
  periodCount: 12,
  dataInterval: 2
})
```

Need at least 6 periods for meaningful statistics.

### 4. Calculate statistics

For each metric/location combination from the response `gridRows`:
- **Mean:** Average of historical values
- **Standard deviation:** Measure of normal variation
- **Min/Max:** Historical range

### 5. Identify anomalies

Flag values exceeding the threshold:
- **Low sensitivity:** 3 standard deviations (extreme outliers only)
- **Medium:** 2 standard deviations (balanced)
- **High:** 1.5 standard deviations (more sensitive)

### 6. Present anomalies

Prioritize by severity (highest deviation first) and suggest possible causes.

---

## Present Results

```
## Anomaly Report: March 2025

Found 3 unusual values that may need review:

### High Priority
**Water Consumption at Site C**
- Current: 45,000 L | Historical avg: 28,000 L
- Deviation: +61% (3.2 standard deviations)
- Possible cause: Leak? New equipment?

### Medium Priority
**Safety Incidents at Region 2**
- Current: 8 incidents | Historical avg: 3
- Deviation: +167% (2.5 standard deviations)

All other values within normal ranges.
```

---

## Example Interaction

**User:** "Are there any unusual values this month?"

**Claude:**
```
templates_spreadsheetReports_list()
→ Found: ESG Report (rpt-abc123)

data_computedValues_list({
  templateId: "rpt-abc123",
  periodType: "month",
  periodCount: 12,
  dataInterval: 2
})
→ Returns 12 months of data across all metrics

→ Statistical analysis: Mean, StdDev for each metric
→ March values compared to historical baseline
→ Found 2 values exceeding 2 standard deviations
```

**Claude:**

## Anomaly Report: March 2025

I analyzed 42 metrics across 12 months. **2 values** are outside normal ranges:

| Metric | Location | Current | Avg (12mo) | Deviation | Priority |
|--------|----------|---------|------------|-----------|----------|
| Water Use | Site C | 45,000 L | 28,000 L | +3.2 std dev | High |
| Incidents | Region 2 | 8 | 3 | +2.5 std dev | Medium |

**Water Use at Site C** — 61% above historical average. This is the most significant outlier. Recommend investigating for equipment issues or reporting errors.

**Safety Incidents** — Unusual spike in Region 2. Consider reviewing incident reports for common causes.

All other 40 metrics are within normal ranges.

---

## Variations

### Focus on specific discipline

Use `disciplineNodeIds` parameter to limit anomaly check to Environmental, Social, or Governance metrics.

### Trend anomalies

Also flag unusual trends (e.g., 3 consecutive months of increase when the norm is flat).

---

## Edge Cases

- **Insufficient history:** Need at least 6 data points for meaningful statistics. Fewer than 6, warn about low confidence.
- **Seasonal data:** If data has known seasonal patterns, note that apparent anomalies may be seasonal.
- **New metrics:** Metrics without history can't be analyzed — skip and note.

---

## CLI Equivalent

[CLI: Spot Anomalies](../../../capstone-cli/recipes/analysis/spot-anomalies.md)
