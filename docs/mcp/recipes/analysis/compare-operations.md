# Recipe: Compare Operations

> Compare performance between two or more locations.

## When to Use

- "Compare Site A vs Site B"
- "Which facility is performing best?"
- "How does Region 1 compare to Region 2?"
- "Show me a location comparison"

## Prompt Shortcut

```
/compare-operations location1="Site A" location2="Site B"
```

Both location arguments are required.

---

## Required Context

| What's Needed | Source | If Missing |
|--------------|--------|------------|
| Location names/IDs | `capstone://model/organization` resource or `model_orgNodes_list` | Ask: "Which two locations should I compare?" |
| Report template | `templates_spreadsheetReports_list` | Auto-select first template |
| Time period | User request or `data_availability` | Default: most recent period |
| Metrics to compare | User request | Default: all available metrics |

---

## Workflow

### 1. Find org nodes

```
model_orgNodes_list()
```

Match location names from the response `gridRows`. Extract the `id` for each matching org node.

**If ambiguous:** "I found 'Site A' and 'Site Alpha'. Which did you mean?"

### 2. Resolve report template

```
templates_spreadsheetReports_list()
```

Pick a template covering the comparison metrics.

### 3. Get data for location 1

```
data_computedValues_list({
  templateId: "<report-template-id>",
  timePeriodNames: "Q1 FY 25",
  dataInterval: 3,
  orgNodeIds: "<location-1-id>"
})
```

### 4. Get data for location 2

```
data_computedValues_list({
  templateId: "<report-template-id>",
  timePeriodNames: "Q1 FY 25",
  dataInterval: 3,
  orgNodeIds: "<location-2-id>"
})
```

### 5. Compare metrics

For each metric present in both responses:
- Extract values from data rows (`isDataRow: true`)
- Calculate absolute and percentage differences
- Determine which location performs better

### 6. Highlight significant differences

Flag metrics where the difference exceeds 20%.

---

## Present Results

```
## Location Comparison: Q1 2025

### Emissions
| Metric | Site A | Site B | Difference |
|--------|--------|--------|------------|
| Total GHG | 450 t | 520 t | Site A is 13.5% lower |
| Energy Use | 138,000 kWh | 125,000 kWh | Site B is 9.4% lower |

### Overall Assessment
- Best emissions: Site A (-13.5%)
- Best energy: Site B (-9.4%)
- Recommendation: Investigate Site A's energy efficiency practices
```

---

## Example Interaction

**User:** "Compare Site A vs Site B"

**Claude:**
```
model_orgNodes_list()
→ Found: Site A (org-111), Site B (org-222)

templates_spreadsheetReports_list()
→ Found: ESG Report (rpt-abc123)

data_computedValues_list({
  templateId: "rpt-abc123",
  periodType: "quarter",
  periodCount: 1,
  dataInterval: 3,
  orgNodeIds: "org-111"
})
→ Site A values: GHG=450t, Energy=138,000kWh, Water=28,000L

data_computedValues_list({
  templateId: "rpt-abc123",
  periodType: "quarter",
  periodCount: 1,
  dataInterval: 3,
  orgNodeIds: "org-222"
})
→ Site B values: GHG=520t, Energy=125,000kWh, Water=31,000L
```

**Claude:**

## Site A vs Site B: Q1 2025

| Metric | Site A | Site B | Winner | Difference |
|--------|--------|--------|--------|------------|
| GHG Emissions | 450 t | 520 t | Site A | 13.5% lower |
| Energy Use | 138,000 kWh | 125,000 kWh | Site B | 9.4% lower |
| Water Use | 28,000 L | 31,000 L | Site A | 9.7% lower |

**Summary:** Site A leads on emissions (-13.5%) and water (-9.7%). Site B leads on energy (-9.4%).

**Notable:** Site A has lower emissions despite higher energy use — suggesting a different energy mix (possibly more renewables).

---

## Variations

### Normalized comparison

For fair comparison despite size differences, suggest normalizing per employee, per unit produced, or per square meter.

### Multi-location ranking

For 3+ locations, rank by each metric rather than side-by-side comparison.

### Trend comparison

Compare how locations are changing over time by including multiple periods.

---

## Edge Cases

- **Different metric availability:** Some metrics may not apply to all locations (metric exclusions). Note which metrics are only available at one location.
- **Different org levels:** Comparing a region to a site mixes aggregated vs leaf data. Warn about this.
- **Ambiguous names:** If location name matches multiple org nodes, present options.

---

## CLI Equivalent

[CLI: Compare Operations](../../../cli/recipes/analysis/compare-operations.md)
