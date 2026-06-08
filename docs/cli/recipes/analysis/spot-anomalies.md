# Recipe: Spot Anomalies

> Proactively identify unusual values that may need attention.

## When to Use

- "Any unusual values last month?"
- "What looks off?"
- "Are there any outliers?"
- "Data quality check for anomalies"

## Required Context

Before starting, Claude needs:
- [ ] **Scope** — Which metrics/locations to check (default: all)
- [ ] **Period** — Which period to examine (default: most recent)
- [ ] **Baseline** — How many historical periods for comparison (default: 12)

---

## Step-by-Step

### Step 1: Resolve Report Template

**Purpose:** Find a report template for the anomaly scan.

**Command:**
```bash
cap templates report-templates list --json
```

---

### Step 2: Get Current Period Values

**Purpose:** Retrieve latest data to examine.

**Command:**
```bash
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval month \
  --periods "Mar 2025" \
  --json
```

---

### Step 3: Get Historical Baseline

**Purpose:** Establish what's "normal."

**Command:**
```bash
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval month \
  --periods "Mar 2024,Apr 2024,May 2024,Jun 2024,Jul 2024,Aug 2024,Sep 2024,Oct 2024,Nov 2024,Dec 2024,Jan 2025,Feb 2025" \
  --json
```

**Calculate for each metric/location:**
- Historical average
- Standard deviation
- Min/max range

---

### Step 4: Identify Anomalies

**Criteria:**
- More than 2 standard deviations from average
- Outside historical min/max range
- Sudden change (>50% from prior period)

**Score anomalies:** Higher score = more unusual

---

### Step 5: Present Anomalies

**Format:**
```
## Anomaly Report: March 2025

Found 3 unusual values that may need review:

### 🔴 High Priority

**1. Water Consumption at Site C**
- Current: 45,000 L
- Historical avg: 28,000 L
- Deviation: +61% (3.2 standard deviations)
- Possible cause: Leak? New equipment?

### 🟡 Medium Priority

**2. Safety Incidents at Region 2**
- Current: 8 incidents
- Historical avg: 3 incidents
- Deviation: +167% (2.5 standard deviations)
- Note: Review incident reports

### 🟡 Medium Priority

**3. Energy Usage at Site A**
- Current: 52,000 kWh
- Historical avg: 45,000 kWh
- Deviation: +16% (2.1 standard deviations)
- Possible cause: Seasonal (heating)?

---

All other values within normal ranges ✅
```

---

## Example Interaction

**User:** "Any unusual values this month?"

**Claude:** Let me check March 2025 against your historical baseline.

```bash
cap templates report-templates list --json
# Found: ESG Report (rpt-123)

cap reporting computed-values list \
  --template rpt-123 \
  --data-interval month \
  --periods "Mar 2025" \
  --json
# Current values

cap reporting computed-values list \
  --template rpt-123 \
  --data-interval month \
  --periods "Mar 2024,Apr 2024,May 2024,Jun 2024,Jul 2024,Aug 2024,Sep 2024,Oct 2024,Nov 2024,Dec 2024,Jan 2025,Feb 2025" \
  --json
# 12 months of history for baseline
```

**Claude:**

## Anomaly Scan: March 2025

Analyzed 42 metrics across 15 locations.

**Found 2 anomalies:**

| Metric | Location | Value | vs Average | Severity |
|--------|----------|-------|------------|----------|
| Water Usage | Site C | 45,000 L | +61% | 🔴 High |
| Safety Incidents | Region 2 | 8 | +167% | 🟡 Medium |

**Water Usage at Site C:**
This is unusually high. Historical average is 28,000 L. Recommend investigating for:
- Water leaks
- New water-intensive equipment
- Data entry error

**Safety Incidents at Region 2:**
8 incidents vs typical 3. Recommend:
- Review individual incident reports
- Check if related or separate events

Everything else looks normal. Would you like details on any flagged item?

---

## Variations

### Focus Area

Limit to specific discipline:
```bash
--discipline-nodes <discipline-node-id>
```

### Sensitivity Adjustment

- High sensitivity: 1.5 standard deviations
- Normal: 2 standard deviations
- Low: 3 standard deviations

### Trend Anomalies

Also flag unusual trends (not just values):
```
"Site B emissions have increased for 3 consecutive months,
counter to the company trend."
```

---

## Related Recipes

- [Find Trends](./find-trends.md) — Understand patterns over time
- [Check Data Completeness](../exploration/check-completeness.md) — Find missing data
- [Talk to Your Data](../exploration/talk-to-your-data.md) — Investigate specific values
