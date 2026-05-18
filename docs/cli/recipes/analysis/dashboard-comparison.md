# Recipe: Dashboard Comparison

> Compare dashboard data across time periods or organizational units.

## When to Use

- "How does Q1 compare to Q4?"
- "Compare Mining vs Manufacturing dashboards"
- "What changed from last year?"
- "Compare this quarter to the same quarter last year"
- "How do our regions differ on this dashboard?"

## Required Context

Before starting, Claude needs:
- [ ] **Dashboard** — Which dashboard to compare
- [ ] **Comparison type** — Time periods or locations?
- [ ] **Comparison targets** — Which periods or org nodes to compare

---

## Step-by-Step

### Step 1: Identify Dashboard

**Purpose:** Find the dashboard to compare.

**Command:**
```bash
cap templates dashboard-templates list --json
```

**If user gave name:**
Find matching dashboard ID.

---

### Step 2: Determine Comparison Type

**Ask user if not specified:**
- **Period comparison:** Same location, different time periods
- **Location comparison:** Same time period, different locations

---

### Step 3: Get Comparison Data

**For Period Comparison:**
```bash
# Current period
cap reporting dashboards get-data <dashboard-id> \
  --org-node <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json

# Prior period
cap reporting dashboards get-data <dashboard-id> \
  --org-node <org-node-id> \
  --data-interval quarter \
  --periods "Q4 FY 24" \
  --json
```

**For Location Comparison:**
```bash
# Location A
cap reporting dashboards get-data <dashboard-id> \
  --org-node <mining-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json

# Location B
cap reporting dashboards get-data <dashboard-id> \
  --org-node <manufacturing-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
```

---

### Step 4: Analyze Differences

**For each metric/widget:**
- Calculate delta (absolute and percentage)
- Determine significance
- Note direction of change

**Significance thresholds:**
| Change | Significance |
|--------|--------------|
| < 5% | Minor |
| 5-15% | Moderate |
| > 15% | Significant |

---

### Step 5: Present Comparison

**Period Comparison Format:**
```
## Dashboard Comparison: Q1 FY 25 vs Q4 FY 24

### Overall Summary
- **Improved:** 6 metrics
- **Declined:** 2 metrics
- **Unchanged:** 1 metric

### Environmental Section

| Metric | Q4 FY 24 | Q1 FY 25 | Change | Trend |
|--------|----------|----------|--------|-------|
| GHG Emissions | 1,920 t | 1,850 t | -3.6% | Improving |
| Energy Use | 495 MWh | 485 MWh | -2.0% | Improving |
| Water Use | 45 kL | 48 kL | +6.7% | Worsening |
| Renewable % | 42% | 45% | +7.1% | Improving |

### Social Section

| Metric | Q4 FY 24 | Q1 FY 25 | Change | Trend |
|--------|----------|----------|--------|-------|
| Safety Score | 92 | 88 | -4.3% | Worsening |
| Training Hours | 180 | 165 | -8.3% | Worsening |

### Governance Section

| Metric | Q4 FY 24 | Q1 FY 25 | Change | Trend |
|--------|----------|----------|--------|-------|
| Policy Compliance | 96% | 98% | +2.1% | Improving |
| Supplier Score | 75 | 78 | +4.0% | Improving |

### Key Changes

1. **Biggest improvement:** Renewable energy percentage (+7.1%)
2. **Biggest decline:** Training hours (-8.3%)
3. **Watch area:** Water consumption trend reversal

### Insights

- Environmental momentum is strong — emissions and energy trending down
- Social metrics need attention — both training and safety declining
- Governance continues to strengthen
```

**Location Comparison Format:**
```
## Dashboard Comparison: Mining vs Manufacturing (Q1 FY 25)

### Overall Summary
- **Mining ahead:** 4 metrics
- **Manufacturing ahead:** 3 metrics
- **Similar:** 2 metrics

### Side-by-Side

| Metric | Mining | Manufacturing | Delta | Leader |
|--------|--------|---------------|-------|--------|
| GHG Emissions | 850 t | 1,000 t | +18% | Mining |
| Energy Use | 200 MWh | 285 MWh | +43% | Mining |
| Water Use | 35 kL | 13 kL | -63% | Manufacturing |
| Safety Score | 85 | 91 | +7% | Manufacturing |
| Training Hours | 85 | 80 | -6% | Mining |

### Key Differences

1. **Energy efficiency:** Mining significantly more efficient
2. **Water conservation:** Manufacturing excels (different process needs)
3. **Safety:** Manufacturing has higher safety score

### Insights

- Different operational profiles explain most variances
- Mining's energy advantage may be due to automation investments
- Manufacturing's water efficiency is a best practice to share
```

---

## Example Interaction

**User:** "How does Q1 compare to Q4 on the ESG dashboard?"

**Claude:**
```bash
cap templates dashboard-templates list --json
# Found: ESG Executive Dashboard (dash-123)

cap masterdata org-nodes list --json
# Using root org node: org-root

cap reporting dashboards get-data dash-123 \
  --org-node org-root \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
# Q1 data

cap reporting dashboards get-data dash-123 \
  --org-node org-root \
  --data-interval quarter \
  --periods "Q4 FY 24" \
  --json
# Q4 data
```

**Claude:**

## Q1 FY 25 vs Q4 FY 24 Comparison

### The Good News
- Emissions down 3.6% (1,920 → 1,850 tonnes)
- Renewable energy share up 7.1% (42% → 45%)
- Compliance improved to 98%

### Watch Areas
- Training hours dropped 8.3% — this is driving your safety score decline
- Water consumption reversed trend (+6.7%)

### Recommendation
The Q-over-Q momentum is mostly positive, but social metrics need immediate attention before they impact annual targets.

---

## Variations

### Year-over-Year Comparison

Compare to same quarter last year:
```bash
cap reporting dashboards get-data <dashboard-id> \
  --org-node <id> --data-interval quarter \
  --periods "Q1 FY 25" --json

cap reporting dashboards get-data <dashboard-id> \
  --org-node <id> --data-interval quarter \
  --periods "Q1 FY 24" --json
```

Helpful for seasonality-adjusted comparison.

### Multi-Period Trend

Compare across multiple periods:
```bash
cap reporting dashboards get-data <dashboard-id> \
  --org-node <id> --data-interval quarter \
  --periods "Q1 FY 25,Q4 FY 24,Q3 FY 24,Q2 FY 24" --json
```

Shows trend direction across the year.

### Parent-Child Location Comparison

Compare division to company overall:
```bash
# Company overall
cap reporting dashboards get-data <dashboard-id> \
  --org-node <company-root-id> \
  --data-interval quarter --json

# Specific division
cap reporting dashboards get-data <dashboard-id> \
  --org-node <division-id> \
  --data-interval quarter --json
```

Identify where a division outperforms or underperforms company averages.

### Section-Specific Comparison

Compare just one dashboard section:
```bash
cap reporting dashboards get-data <dashboard-id> \
  --layout-node <environmental-section-id> \
  --scope descendants \
  --org-node <id> \
  --data-interval quarter \
  --periods "Q1 FY 25" --json
```

Focused comparison for specific domain.

---

## Related Recipes

- [Deep Dashboard Analysis](./deep-dashboard-analysis.md) — Cross-metric pattern discovery
- [Compare Operations](./compare-operations.md) — Detailed location comparison
- [Find Trends](./find-trends.md) — Multi-period trend analysis
- [Talk to Your Dashboard](../exploration/talk-to-your-dashboard.md) — Dashboard exploration
