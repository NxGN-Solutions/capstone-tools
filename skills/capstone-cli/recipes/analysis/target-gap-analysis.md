# Recipe: Target Gap Analysis

> Compare actual performance against targets.

## When to Use

- "Are we on track for our targets?"
- "How close are we to the emissions goal?"
- "Target status report"
- "Will we meet our commitments?"

## Required Context

Before starting, Claude needs:
- [ ] **Metrics with targets** — Which goals to evaluate
- [ ] **Period** — Time frame for comparison
- [ ] **Scope** — Location scope (default: company-wide)

---

## Step-by-Step

### Step 1: Identify Metrics with Targets

**Purpose:** Find metrics that have targets defined.

**Command:**
```bash
cap model metrics list --json
```

**Filter:** Metrics where target values exist.

---

### Step 2: Get Target Values

**Purpose:** Extract target from metric configuration.

**Note:** Targets may be annual; needs period allocation.

---

### Step 3: Resolve Report Template

**Purpose:** Find a report template that includes the target metrics.

**Command:**
```bash
cap templates report-templates list --json
```

---

### Step 4: Get Actual Values

**Purpose:** Retrieve current performance.

**Command:**
```bash
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
```

---

### Step 5: Calculate Gaps

**For each metric:**
- Actual vs Target
- Gap (absolute and percentage)
- Status determination

**Status thresholds:**
- 🟢 On Track: Within 10% of target
- 🟡 At Risk: 10-25% behind
- 🔴 Behind: >25% behind

---

### Step 6: Present Status

**Format:**
```
## Target Status Report: Q1 FY 2025

### Summary
🟢 On Track: 4 metrics
🟡 At Risk: 2 metrics
🔴 Behind: 1 metric

### Details

| Metric | Target | Actual | Gap | Status |
|--------|--------|--------|-----|--------|
| GHG Emissions | 2,000 t | 1,850 t | -7.5% | 🟢 On Track |
| Energy Use | 500 MWh | 485 MWh | -3.0% | 🟢 On Track |
| Water Use | 50 kL | 48 kL | -4.0% | 🟢 On Track |
| Recycling Rate | 75% | 72% | -4.0% | 🟢 On Track |
| Training Hours | 200 hrs | 165 hrs | -17.5% | 🟡 At Risk |
| Safety Score | 95 | 88 | -7.4% | 🟡 At Risk |
| Incidents | 0 | 3 | +300% | 🔴 Behind |

### Year-End Projections

At current rate:
- GHG Emissions: Will achieve target (projected: 1,920 t)
- Training Hours: Will miss by ~50 hours without acceleration
- Incidents: Already exceeded annual target

### Recommendations

1. **Training Hours** — Need 105 more hours in Q2-Q4 (35/quarter)
2. **Safety Score** — Investigate Q1 dip, implement corrective action
3. **Incidents** — Root cause analysis required
```

---

## Example Interaction

**User:** "Are we on track for our emission targets?"

**Claude:**
```bash
cap model metrics list --json
# Found: GHG Emissions (target: 8,000 tonnes/year)

cap templates report-templates list --json
# Found: ESG Report (rpt-123)

cap reporting computed-values list \
  --template rpt-123 \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
# Q1 Actual: 1,850 tonnes
```

**Claude:**

## Emissions Target Status: Q1 FY 2025

**Target:** 8,000 tonnes CO2e (annual)
**Q1 Budget:** 2,000 tonnes (25% of annual)
**Q1 Actual:** 1,850 tonnes

**Status:** 🟢 On Track (-7.5% under budget)

**Breakdown:**
| Quarter | Budget | Actual | Status |
|---------|--------|--------|--------|
| Q1      | 2,000  | 1,850  | ✅ -7.5% |
| Q2      | 2,000  | —      | Upcoming |
| Q3      | 2,000  | —      | Upcoming |
| Q4      | 2,000  | —      | Upcoming |

**Projection:** At current pace, annual emissions will be ~7,400 tonnes, 7.5% under the 8,000 target.

---

## Variations

### Multi-Target View

When metrics have multiple targets (YTD, YoY, absolute):
```
| Target Type | Goal | Actual | Status |
|-------------|------|--------|--------|
| Annual      | 8,000| 1,850  | 🟢     |
| YoY Change  | -10% | -12%   | 🟢     |
| Monthly Cap | 700  | 620    | 🟢     |
```

### Location-Level Targets

Show target status by location:
```
| Site | Target | Actual | Status |
|------|--------|--------|--------|
| A    | 2,000  | 1,800  | 🟢     |
| B    | 3,000  | 3,200  | 🔴     |
| C    | 3,000  | 2,850  | 🟢     |
```

---

## Related Recipes

- [Find Trends](./find-trends.md) — See trajectory toward target
- [Talk to Your Data](../exploration/talk-to-your-data.md) — Investigate gaps
- [Generate Narrative](../reporting/generate-narrative.md) — Write target summary
