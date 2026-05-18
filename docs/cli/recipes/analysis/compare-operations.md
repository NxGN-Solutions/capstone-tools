# Recipe: Compare Operations

> Compare performance between two or more locations.

## When to Use

- "Compare Site A vs Site B"
- "Which facility is performing best?"
- "How does Region 1 compare to Region 2?"
- "Show me a location comparison"

## Required Context

Before starting, Claude needs:
- [ ] **Locations** — At least 2 locations to compare
- [ ] **Metrics** — What to compare (default: key metrics)
- [ ] **Period** — Time range for comparison

---

## Step-by-Step

### Step 1: Identify Comparison Subjects

**Purpose:** Determine which locations to compare.

**Command:**
```bash
cap masterdata org-nodes list --json
```

**If user gave names:**
- Find matching org nodes
- Confirm if multiple matches

**If less than 2 locations:**
```
"I need at least two locations to compare. Which would you like?
- Site A
- Site B
- Site C"
```

---

### Step 2: Identify Comparison Metrics

**Purpose:** Determine what to compare.

**If user specified:**
```bash
cap model metrics list --json
```
Find matching metrics.

**If not specified:**
- Default to key metrics for the location type
- Or ask: "What would you like to compare? (emissions, energy, safety, all)"

---

### Step 3: Resolve Report Template

**Purpose:** Find a report template that includes the comparison metrics.

**Command:**
```bash
cap templates report-templates list --json
```

---

### Step 4: Get Data for Each Location

**Purpose:** Retrieve computed values for comparison.

**Command:**
```bash
# For each location - filter by org-node
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval quarter \
  --org-nodes <location-id> \
  --periods "Q1 FY 25" \
  --json
```

**Required parameters:**
- `--template` — Report template ID
- `--data-interval` — day/week/month/quarter/year
- `--org-nodes` — The location to compare

---

### Step 5: Calculate Comparisons

**Purpose:** Compute differences and rankings.

**For each metric:**
- Absolute values per location
- Percentage difference
- Ranking (1st, 2nd, etc.)

---

### Step 6: Present Comparison

**Purpose:** Display side-by-side results.

**Format:**
```
## Location Comparison: Q1 2025

### Emissions
| Location | Total (tonnes) | Rank | vs Best |
|----------|----------------|------|---------|
| Site A   | 450            | 1    | —       |
| Site B   | 520            | 2    | +15.6%  |
| Site C   | 610            | 3    | +35.6%  |

### Energy Usage
| Location | Total (kWh) | Rank | vs Best |
|----------|-------------|------|---------|
| Site B   | 125,000     | 1    | —       |
| Site A   | 138,000     | 2    | +10.4%  |
| Site C   | 142,000     | 3    | +13.6%  |

### Overall Assessment
- **Best Overall:** Site A (lowest emissions)
- **Most Improved:** Site B (best energy efficiency)
- **Needs Attention:** Site C (highest across both metrics)
```

---

## Example Interaction

**User:** "Compare Site A and Site B on emissions"

**Claude:**
```bash
cap masterdata org-nodes list --json
# Found: Site A (org-123), Site B (org-456)

cap templates report-templates list --json
# Found: ESG Report (rpt-789) - includes emissions

cap reporting computed-values list \
  --template rpt-789 \
  --data-interval quarter \
  --org-nodes org-123 \
  --periods "Q1 FY 25" \
  --json
# Site A: 450 tonnes

cap reporting computed-values list \
  --template rpt-789 \
  --data-interval quarter \
  --org-nodes org-456 \
  --periods "Q1 FY 25" \
  --json
# Site B: 520 tonnes
```

**Claude:**

## Emissions Comparison: Q1 2025

| Metric | Site A | Site B | Difference |
|--------|--------|--------|------------|
| Total GHG Emissions | 450 tonnes | 520 tonnes | Site A is 13.5% lower |

**Analysis:**
- Site A emitted 70 fewer tonnes than Site B this quarter
- Both sites improved from Q4 2024 (Site A: -8%, Site B: -5%)

**Possible factors:**
- Site A completed energy efficiency upgrades in January
- Site B has larger production volume (+22% vs Site A)

Would you like to see emissions intensity (per unit produced) for a fairer comparison?

---

## Variations

### Normalized Comparison

For fair comparison despite size differences:
- Per employee (FTE)
- Per unit produced
- Per square meter

### Trend Comparison

Compare how locations are changing:
```
| Location | Q4 2024 | Q1 2025 | Change |
|----------|---------|---------|--------|
| Site A   | 490     | 450     | -8.2%  |
| Site B   | 548     | 520     | -5.1%  |
```

### Many Locations

For 5+ locations, rank by metric:
```
Top 5 by Efficiency:
1. Site D — 380 tonnes
2. Site A — 450 tonnes
3. Site B — 520 tonnes
...
```

---

## Related Recipes

- [Talk to Your Data](../exploration/talk-to-your-data.md) — Analyze single location
- [Find Trends](./find-trends.md) — Time-based comparison
- [Discover Tenant Structure](../exploration/discover-tenant-structure.md) — See all locations
