# Recipe: Check Data Completeness

> Identify missing data that needs to be captured.

## When to Use

- "What's missing for Q1?"
- "Data quality audit"
- "Which sites haven't submitted?"
- "Completeness check for reporting"

## Required Context

Before starting, Claude needs:
- [ ] **Period** — Which time frame to check
- [ ] **Scope** — All metrics or specific discipline
- [ ] **Locations** — All or specific org nodes

---

## Step-by-Step

### Step 1: Define Expected Data Points

**Get all possible combinations:**

```bash
cap masterdata org-nodes list --json   # Leaf nodes (entry points)
cap model inputs list --json           # Input metrics
cap data time-periods list --data-interval month --json # Periods in range
```

**Calculate expected:**
```
Expected = Leaf Org Nodes × Input Metrics × Periods
```

---

### Step 2: Resolve Report Template

**Purpose:** Find a report template that covers the completeness check.

**Command:**
```bash
cap templates report-templates list --json
```

---

### Step 3: Get Actual Data Points

**Command:**
```bash
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval month \
  --periods "Jan 2025,Feb 2025,Mar 2025" \
  --json
```

---

### Step 4: Identify Gaps

**For each expected combination:**
- Check if value exists in actual data
- If missing, add to gap list

**Group by:**
- Location (which sites have gaps)
- Metric (which metrics are incomplete)
- Period (which periods are incomplete)

---

### Step 5: Present Completeness Report

**Format:**
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
| Site A   | 150      | 150      | 100% ✅      |
| Site B   | 150      | 138      | 92% 🟡       |
| Site C   | 150      | 124      | 83% 🔴       |

### By Metric

| Metric | Expected | Captured | Missing |
|--------|----------|----------|---------|
| Energy | 45       | 45       | 0       |
| Water  | 45       | 42       | 3       |
| Waste  | 45       | 38       | 7       |

### Missing Data Details

**Site C - Incomplete:**
- Water Usage: Jan, Feb, Mar (3 missing)
- Waste Generated: Jan, Feb, Mar (3 missing)
- Recycling Rate: Feb, Mar (2 missing)

**Site B - Incomplete:**
- Waste Generated: Mar (1 missing)
- Safety Training: Feb (1 missing)

### Priority Actions

1. **Site C** — 8 data points missing (26% of gaps)
   Contact: Operations Manager
2. **Waste metrics** — 7 gaps across sites
   Due date: End of Q1 reporting period
```

---

## Example Interaction

**User:** "What data is missing for Q1 reporting?"

**Claude:**
```bash
cap masterdata org-nodes list --json
# Found 15 leaf org nodes

cap model inputs list --json
# Found 10 input metrics

cap data time-periods list --data-interval month --json
# Q1 = Jan, Feb, Mar 2025

# Expected: 15 × 10 × 3 = 450 data points

cap templates report-templates list --json
# Found: ESG Report (rpt-123)

cap reporting computed-values list \
  --template rpt-123 \
  --data-interval month \
  --periods "Jan 2025,Feb 2025,Mar 2025" \
  --json
# Found 412 actual values
```

**Claude:**

## Q1 2025 Completeness: 91.6%

**Summary:** 38 data points missing out of 450 expected.

**Biggest gaps:**

| Issue | Impact |
|-------|--------|
| Site C not submitting waste data | 9 missing values |
| Safety Training incomplete | 5 missing values |

**Quick wins (easiest to fix):**
1. Site B March Waste — Just 1 value needed
2. Site A Safety Training Feb — Single entry

**Blocked:**
- Site C has no assigned data entry user
- Recommend: Assign user before follow-up

Would you like me to generate a reminder email for the missing data?

---

## Variations

### Validation Status

Include validation completeness:
```
| Location | Entered | Validated | Approved |
|----------|---------|-----------|----------|
| Site A   | 100%    | 85%       | 60%      |
| Site B   | 92%     | 70%       | 45%      |
```

### Time Series

Track completeness over time:
```
"Q1 completeness improved from 85% to 92% over the quarter."
```

### Comparison to Prior Period

Show improvement/decline:
```
"Q1: 92% complete (vs Q4: 88%)"
```

---

## Related Recipes

- [Enter Data](../data-management/enter-data.md) — Fill gaps one at a time
- [Bulk Import Data](../data-management/bulk-import.md) — Fill gaps via Excel
- [Spot Anomalies](../analysis/spot-anomalies.md) — Check data quality
