# Data Model Reference

> Enum tables, aggregation patterns, analysis techniques, and query patterns for Capstone MCP tools.

---

## Enum Tables

### Data Interval (Time Period Types)

| ID | Name | Description |
|----|------|-------------|
| 0 | Day | Daily data capture |
| 1 | Week | Weekly data capture |
| 2 | Month | Monthly data capture (most common) |
| 3 | Quarter | Quarterly data capture |
| 4 | Year | Annual data capture |

### Metric Types

| ID | Name | Description |
|----|------|-------------|
| 0 | Input | Manual data entry or automated ingestion |
| 1 | Calculation | Formula-derived from other metrics |

### Widget Types

| ID | Name | Use Case |
|----|------|----------|
| 0 | InfoCard | Single KPI value with optional comparison |
| 1 | PieChart | Part-to-whole relationships, breakdowns |
| 2 | XYChart | Time series, trends, multi-metric comparison |
| 4 | Table | Metric-backed dashboard table/grid |

### Calculation Phases

| ID | Name | Use When |
|----|------|----------|
| 0 | Before Aggregations | Compute at leaf nodes, then aggregate. Use for conversions, site-level rates |
| 1 | After Aggregations | Compute after rollup. Use for ratios of aggregated totals, company-wide rates |

> **Phase matters for accuracy:** A ratio like "cost per unit" should be calculated After Aggregations — otherwise you'd average site-level ratios instead of dividing total cost by total units.

### Org Structure Aggregation Methods

How metric values combine up the org hierarchy:

| ID | Name | Use When |
|----|------|----------|
| 0 | Sum | Values add up (consumption, cost, throughput) |
| 1 | Average | Values average across children (rates, scores, percentages) |
| 2 | Count | Count of non-null child values |
| 3 | None | Node-specific, doesn't roll up |
| 4 | Roll Up | Uses first non-null child value |
| 5 | Roll Down | Cascades parent value to children (assumptions, targets) |

### Time Period Aggregation Methods

How metric values combine across time periods:

| ID | Name | Use When |
|----|------|----------|
| 0 | None | Each period shown separately (time series) |
| 1 | Sum | Values sum across periods (cumulative metrics) |
| 2 | Average | Values average across periods (rates, scores) |
| 3 | Last Value | Most recent period only (snapshots) |
| 4 | Min | Minimum value across periods |
| 5 | Max | Maximum value across periods |

### Validation Status

| ID | Name | Description |
|----|------|-------------|
| 0 | Draft | Value entered, not yet submitted |
| 1 | Pending | Submitted, awaiting approval |
| 2 | Approved | Validated and accepted |
| 3 | Rejected | Reviewed and rejected |

---

## Common Aggregation Patterns

Understanding these patterns helps interpret computed values correctly:

| Metric Type | Org Agg | Time Agg | Example |
|-------------|---------|----------|---------|
| **Throughput/Volume** | Sum | Sum | Electricity consumption: kWh across sites and months |
| **Cost/Revenue** | Sum | Sum | Financial totals: dollars across departments |
| **Rates/Scores (%)** | Average | Average | OEE percentage, safety scores |
| **Assumptions** | Roll Down | Average | Budget assumptions: set at parent, inherited |
| **Targets** | Roll Down | Average | Annual targets: set once, cascade down |
| **Per-unit ratios** | None | None | Cost per unit: recomputed from aggregated inputs |
| **Headcount/Balance** | Sum or None | Last Value | Point-in-time snapshots |

> **Interpretation guide:** When analyzing data, check the aggregation method to understand what a value means at different org levels. A "Sum" metric at a parent node is the total of all children. An "Average" metric is the mean across children.

---

## Three-Part Query Pattern

All data queries in Capstone require three dimensions:

### 1. What (Template or Metrics)

Templates define **which metrics are visible** in a query. They act as a filter lens.

| Template Type | Tool | Purpose |
|---------------|------|---------|
| Report Template | `templates_spreadsheetReports_list` | Filter metrics for computed values |
| Capture Template | `templates_spreadsheetCaptures_list` | Filter metrics for input values |
| Dashboard Template | `templates_dashboards_list` | Organize widgets for dashboard data |
| Widget Template | `templates_widgets_list` | Define single widget data source |

### 2. Where (Org Node)

Org nodes determine the **scope** of the data:

- **Root node** = company-wide (all data rolled up)
- **Intermediate node** = regional/divisional view
- **Leaf node** = individual site/facility data

Use `model_orgNodes_list` to discover the hierarchy.

### 3. When (Time Periods)

Time periods can be specified two ways:

| Approach | Parameters | Best For |
|----------|-----------|----------|
| **By name** | `timePeriodNames="Jan 2025,Feb 2025"` | Known periods |
| **By type+count** | `periodType="quarter"`, `periodCount=4` | Most recent N periods |

Use `data_timePeriods_list` to discover available period names.
Use `data_availability` to find which periods have data.

---

## Metric and Org Node Hierarchy

### Metric Hierarchy

Metrics form a tree structure:

```
Root Category (Discipline)
├── Input Metric A (manual data entry)
├── Input Metric B (manual data entry)
└── Calculation Metric C (formula: [A] + [B])
    └── Sub-calculation D (formula: [C] / [E])
```

- **Input metrics** are leaf nodes where data is captured
- **Calculation metrics** derive values from formulas referencing other metrics
- Formulas use `[Metric Name]` or `[GUID]` syntax

### Org Node Hierarchy

```
Corporate HQ (Level 0)
├── Region 1 (Level 1)
│   ├── Site A (Level 2 — leaf)
│   └── Site B (Level 2 — leaf)
└── Region 2 (Level 1)
    └── Site C (Level 2 — leaf)
```

- Data is typically entered at **leaf nodes** (individual sites)
- **Parent nodes** show aggregated values based on aggregation methods
- Different metrics may use different aggregation strategies at the same org level

---

## Analysis Techniques

When analyzing retrieved data, apply these techniques based on the user's question type:

### Quantitative Techniques

| Question Type | Technique | When to Use |
|---------------|-----------|-------------|
| "How much?" | Sum values for the period | User asks about totals |
| "Compared to?" | Calculate absolute and percentage difference | Period-over-period or location comparison |
| "Trending?" | Linear regression on time series | 6+ data points over time |
| "Highest/lowest?" | Sort and identify extremes | Ranking locations or periods |
| "On track?" | Actual vs target, gap percentage | Target tracking |
| "Normal?" | Mean + standard deviation from historical data | Anomaly detection (need 6+ periods) |
| "What's missing?" | Expected vs captured data points | Data quality audit |

### Status Classification

Use consistent thresholds when reporting status:

| Status | Threshold | Description |
|--------|-----------|-------------|
| On Track | Within 10% of target | Performing as expected |
| At Risk | 10-25% behind target | Needs attention |
| Behind | >25% behind target | Requires action |

### Anomaly Detection

Use standard deviation thresholds for sensitivity levels:

| Sensitivity | Threshold | Description |
|-------------|-----------|-------------|
| Low | 3 standard deviations | Only extreme outliers |
| Medium | 2 standard deviations | Balanced (recommended default) |
| High | 1.5 standard deviations | More sensitive detection |

### Comparison Fairness

When comparing locations or periods:
1. Check aggregation methods — Sum metrics are additive, Average metrics are not
2. Consider normalizing by production volume, headcount, or area for fair comparison
3. Check the data interval — monthly values won't match quarterly totals unless aggregated
4. Note org level differences — comparing a region (aggregated) to a site (leaf) is misleading

---

## Interpretation Guidance

### Reading Computed Values

When you retrieve computed values, the response contains:
- **Grid rows** with metric name, unit of measure, and precision
- **Time period columns** with values aligned by period
- **Data rows** (where `isDataRow: true`) contain actual values

### Reading Input Values

When you retrieve input values, each cell has:
- **value** — The numeric data point (absent when no data entered)
- **validationStatus** — Draft (0), Pending (1), Approved (2), Rejected (3)
- **isLocked** — Whether the period is locked for editing
- **id** — Zero GUID means no value exists; a real GUID means data was entered

### Comparing Values

When comparing values across periods or locations:
1. Check if metrics use **Sum** aggregation (values are additive)
2. Check if metrics use **Average** aggregation (values are averages)
3. For fair location comparison, consider normalizing by production volume or headcount
4. Check the **data interval** — monthly values won't match quarterly totals unless aggregated

### Missing Data

- A missing value means no data was captured for that metric/org/period
- This is different from a zero value (which was explicitly entered)
- Use the `/check-completeness` prompt to audit data gaps

### Formula Language Quick Reference

Formulas are used in Calculation metrics. They support:

```
[Metric Name]                     Reference another metric by name
[Metric Name]|-3:-1|              Reference last 3 periods
IF condition THEN x ELSE y        Conditional logic
SUM([A], [B], [C])                Sum of values
AVG([A], [B])                     Average of values
+ - * / ^ %                       Arithmetic operators
= <> > < >= <=                    Comparison operators
&& || !                           Logical operators
```

**Safe division pattern:**
```
IF [Denominator] <> 0 THEN [Numerator] / [Denominator] ELSE 0
```

---

## See Also

- [Tools Reference](./tools.md) — Full parameter documentation with response formats
- [Glossary](./glossary.md) — Domain term definitions with MCP tools
- [Resources](./resources.md) — Auto-loaded context data
- [CLI Model Building](../../cli/reference/model-building.md) — Detailed model creation reference (for CLI users)
