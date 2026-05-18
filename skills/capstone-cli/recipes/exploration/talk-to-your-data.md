# Recipe: Talk to Your Data

> Answer natural language questions about specific metrics by retrieving and analyzing data.

## When to Use

- "Analyze our emissions for Q1"
- "What's our energy usage looking like?"
- "How much water did we use last year?"
- "Tell me about [metric] at [location]"

## Required Context

Before starting, Claude needs:
- [ ] **Metric(s)** — What to analyze (by name or topic)
- [ ] **Time Range** — Which period(s) to cover
- [ ] **Scope** — All locations or specific sites

**Claude will ask for missing information.**

---

## Step-by-Step

### Step 1: Parse User Question

**Purpose:** Extract intent and parameters from natural language.

**Look for:**
- Metrics mentioned (emissions, energy, water, safety)
- Time range (Q1, last 6 months, 2024, this year)
- Scope (all sites, Site A, Region 1)

**If vague:**
```
"You asked about 'emissions.' I found several emission metrics:
1. Scope 1 Emissions (direct)
2. Scope 2 Emissions (electricity)
3. Total GHG Emissions

Which would you like to analyze?"
```

---

### Step 2: Resolve Metric IDs

**Purpose:** Map user's terms to actual metric IDs.

**Command:**
```bash
cap model metrics list --json
```

**Match strategies:**
- Exact name match
- Keyword match (emissions → *Emissions*)
- Discipline match (environmental metrics)

---

### Step 3: Resolve Location Scope

**Purpose:** Determine which org nodes to include.

**If user specified location:**
```bash
cap masterdata org-nodes list --json
```
Find matching org node(s).

**If no location specified:**
Use root org node (company-wide aggregation).

---

### Step 4: Resolve Report Template

**Purpose:** Find a report template that includes the metric.

**Command:**
```bash
cap templates report-templates list --json
```

**Look for:** Templates whose configuration includes the target metric(s).

**If no suitable template exists:**
```
"I couldn't find a report template that includes [metric].
Would you like me to check input values directly instead?"
```

---

### Step 5: Get Computed Values

**Purpose:** Retrieve the aggregated data.

**Command:**
```bash
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval quarter \
  --org-nodes <org-node-ids> \
  --periods "Q1 FY 25" \
  --json
```

**Required parameters:**
- `--template` — Report template ID (from Step 4)
- `--data-interval` — day/week/month/quarter/year

**Optional filters:**
- `--org-nodes` — Limit to specific locations
- `--discipline-nodes` — Filter by discipline
- `--periods` — Specific period names

**If empty:**
```
"No data found for [metric] in [period].
Data may not have been entered yet, or the period may be locked."
```

---

### Step 6: Analyze Data

**Purpose:** Extract insights beyond raw numbers.

**Calculate:**
- Period-over-period change
- High/low values in range
- Percentage of total (if multiple metrics)

**Identify:**
- Trends (increasing, decreasing, stable)
- Anomalies (unusual values)
- Patterns (seasonality)

---

### Step 7: Synthesize Answer

**Purpose:** Present insights in conversational format.

**Format:**
```
## Emissions Analysis: Q1 2025

**Summary:** Total GHG emissions for Q1 2025 were 2,450 tonnes CO2e,
down 8% from Q4 2024 (2,665 tonnes).

**Key Findings:**
- January was the highest month (890 tonnes), likely due to
  heating demand
- March showed significant reduction (720 tonnes) following
  the efficiency program rollout
- Site A contributed 45% of total emissions

**Trend:** Emissions are trending downward at ~3% per month

**Note:** This analysis is based on validated data only.
February data for Site C is still pending approval.
```

---

## Example Interaction

**User:** "How are our emissions doing this year?"

**Claude:** Let me analyze your emissions data for 2025.

First, I'll identify your emission metrics and find a suitable report template.

```bash
cap model metrics list --json
# Found: Total GHG Emissions (met-abc123)

cap templates report-templates list --json
# Found: ESG Quarterly Report (rpt-def456) - includes emissions metrics

cap reporting computed-values list \
  --template rpt-def456 \
  --data-interval month \
  --periods "Jan 2025,Feb 2025,Mar 2025" \
  --json
```

**Claude:**

## Emissions Overview: Q1 2025

**Total emissions:** 2,450 tonnes CO2e

| Month | Emissions | Change |
|-------|-----------|--------|
| January | 890 tonnes | — |
| February | 840 tonnes | -5.6% |
| March | 720 tonnes | -14.3% |

**Trend:** Emissions are decreasing, with a 19% reduction from
January to March. At this rate, Q2 could see further 15% reduction.

**Breakdown by source:**
- Scope 1 (direct): 1,200 tonnes (49%)
- Scope 2 (electricity): 1,250 tonnes (51%)

Would you like me to drill into a specific location or emission type?

---

## Variations

### Multiple Metrics

If user asks about multiple metrics:
- Present each metric's analysis
- Compare metrics where meaningful
- Identify correlations (energy use vs emissions)

### Time Series Analysis

If user asks about trends:
- Extend period range to 12+ months
- Calculate trend line
- Identify seasonality

### Drill-Down

If user wants more detail:
- Break down by location
- Break down by sub-metric
- Show period-over-period comparison

---

## Analysis Techniques

| Question Type | Technique |
|---------------|-----------|
| "How much?" | Sum values for period |
| "Compared to?" | Calculate percentage difference |
| "Trending?" | Linear regression on time series |
| "Highest/lowest?" | Sort and identify extremes |
| "Why?" | Compare to historical baseline, identify outliers |

---

## Related Recipes

- [Compare Operations](../analysis/compare-operations.md) — Compare locations
- [Find Trends](../analysis/find-trends.md) — Detailed trend analysis
- [Spot Anomalies](../analysis/spot-anomalies.md) — Identify unusual values
- [Discover Tenant Structure](./discover-tenant-structure.md) — See available metrics
