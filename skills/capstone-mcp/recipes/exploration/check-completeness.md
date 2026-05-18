# Recipe: Check Completeness

> Audit data completeness for a specific time period.

## When to Use

- "What's missing for Q1?"
- "Data quality audit"
- "Which sites haven't submitted?"
- "Completeness check for reporting"

## Prompt Shortcut

```
/check-completeness period="Q4 2024" orgNode="Region 1"
```

`period` is required. `orgNode` is optional (defaults to entire organization).

---

## Required Context

| What's Needed | Source | If Missing |
|--------------|--------|------------|
| Time period | User request | Ask: "Which period should I check?" |
| Org scope | User request or `capstone://model/organization` resource | Default: entire organization |
| Metrics list | `capstone://model/metrics` resource or `model_metrics_list` | Auto-loaded from resources |

---

## Workflow

### 1. Check data availability

```
data_availability({ dataInterval: 3 })
```

Confirm the target period has data. Identify the most recent period if user said "latest."

### 2. Get metrics list

```
model_metrics_list({ includeCalculations: false })
```

Get all Input metrics (data entry points). Count them — this is the "expected" dimension. Calculations don't need manual data entry.

### 3. Get org nodes

```
model_orgNodes_list()
```

Get leaf nodes (where data is entered). Cross-reference: expected data points = input metrics × leaf org nodes.

### 4. Get actual data

```
data_computedValues_list({
  templateId: "<report-template-guid>",
  timePeriodNames: "Q1 FY 25",
  dataInterval: 3,
  orgNodeIds: "<scoped-org-node-guid>"
})
```

If `orgNode` was specified, filter to that scope.

### 5. Identify gaps

Compare expected data points against actual:
- Parse response `gridRows` — rows with `isDataRow: true` that have values vs rows without
- Calculate overall completeness: `captured / expected * 100`
- Group gaps by location and by metric

### 6. Flag critical gaps

Highlight high-priority missing data:
- Metrics with active targets
- Metrics required by reporting frameworks
- Locations with the most gaps

### 7. Suggest actions

Recommend next steps to fill gaps, prioritized by impact.

---

## Present Results

```
## Data Completeness Report: Q1 2025

### Summary
- **Expected:** 450 data points
- **Captured:** 412 data points
- **Completeness:** 91.6%
- **Missing:** 38 data points

### By Location

| Location | Expected | Captured | Completeness |
|----------|----------|----------|--------------|
| Site A   | 150      | 150      | 100%         |
| Site B   | 150      | 138      | 92%          |
| Site C   | 150      | 124      | 83%          |

### By Metric

| Metric | Expected | Captured | Missing |
|--------|----------|----------|---------|
| Energy | 45       | 45       | 0       |
| Water  | 45       | 42       | 3       |
| Waste  | 45       | 38       | 7       |

### Priority Actions
1. **Site C** — 26 data points missing (68% of gaps)
2. **Waste metrics** — 7 gaps across sites
3. **Water at Site C** — 3 months missing
```

---

## Example Interaction

**User:** "What data is missing for Q1?"

**Claude:**
```
data_availability({ dataInterval: 3 })
→ Q1 FY 25 has data (partial)

model_metrics_list({ includeCalculations: false })
→ 30 input metrics

model_orgNodes_list()
→ 15 leaf sites → Expected: 30 × 15 = 450 data points

templates_spreadsheetReports_list()
→ Found: Full Report Template (rpt-abc123)

data_computedValues_list({
  templateId: "rpt-abc123",
  timePeriodNames: "Q1 FY 25",
  dataInterval: 3
})
→ 412 data points present out of 450 expected
```

**Claude:**

## Data Completeness: Q1 FY 2025

**Overall:** 91.6% complete (412 of 450 data points)

### Biggest Gaps
| Location | Missing | Completeness |
|----------|---------|--------------|
| Site C | 26 | 83% |
| Site B | 12 | 92% |
| Site A | 0 | 100% |

### Missing Metrics
- **Waste metrics:** 7 gaps (Sites B and C)
- **Water metrics:** 3 gaps (Site C only)
- **Safety metrics:** 2 gaps (Site C)

### Recommended Actions
1. Contact Site C data manager — they account for 68% of all gaps
2. Prioritize waste data entry — largest single metric gap
3. Site A is fully complete and could serve as a model for others

---

## Variations

### Validation status

Include validation workflow progress alongside data entry:
```
| Location | Entered | Validated | Approved |
|----------|---------|-----------|----------|
| Site A   | 100%    | 85%       | 60%      |
```

### Period comparison

Track completeness improvement over time: "Q1 completeness improved from 85% to 92% over the quarter."

### Discipline focus

Scope the completeness check using `disciplineNodeIds` to focus on Environmental, Social, or Governance metrics only.

---

## Edge Cases

- **No data for period:** Period exists in time period list but has zero values — report 0% complete and verify the period is correct.
- **New metrics:** Recently added metrics won't have historical data — exclude from completeness calculation or note separately.
- **Aggregated org nodes:** Checking a parent org node may show calculated values even if leaf data is missing — always check leaf nodes for true completeness.

---

## CLI Equivalent

[CLI: Check Completeness](../../../cli/recipes/exploration/check-completeness.md)
