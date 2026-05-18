# Recipe: Find Trends

> Identify how metrics are changing over time.

## When to Use

- "How are emissions trending?"
- "Show the trend for energy usage"
- "Is safety improving?"
- "What's the direction of [metric]?"

## Required Context

Before starting, Claude needs:
- [ ] **Metric** — What to analyze
- [ ] **Time Range** — How far back to look (default: 12 periods)
- [ ] **Location** — Scope (default: all)

---

## Step-by-Step

### Step 1: Identify Metric

**Purpose:** Resolve metric name to ID.

**Command:**
```bash
cap model metrics list --json
```

Find matching metric.

---

### Step 2: Determine Time Range

**Purpose:** Get appropriate period coverage.

**Default:** Last 12 periods of metric's data interval

**Command:**
```bash
cap data time-periods list --data-interval month --json
```

---

### Step 3: Resolve Report Template

**Purpose:** Find a report template that includes the trend metric.

**Command:**
```bash
cap templates report-templates list --json
```

---

### Step 4: Get Time Series Data

**Purpose:** Retrieve values across periods.

**Command:**
```bash
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval month \
  --org-nodes <org-node-id> \
  --periods "Jan 2024,Feb 2024,Mar 2024,Apr 2024,May 2024,Jun 2024,Jul 2024,Aug 2024,Sep 2024,Oct 2024,Nov 2024,Dec 2024" \
  --json
```

**Required parameters:**
- `--template` — Report template ID
- `--data-interval` — Typically `month` for trend analysis

---

### Step 5: Analyze Trend

**Calculate:**
- Period-over-period changes
- Overall direction (up, down, stable)
- Rate of change (% per period)
- Volatility (variance)

**Identify:**
- Inflection points (trend reversals)
- Seasonality (if applicable)

---

### Step 6: Present Trend

**Format:**
```
## Trend Analysis: GHG Emissions (12 months)

**Direction:** ↘️ Decreasing
**Rate:** -2.5% per month
**Total Change:** -26% year-over-year

| Period | Value | Change |
|--------|-------|--------|
| Jan 24 | 890   | —      |
| Feb 24 | 850   | -4.5%  |
| Mar 24 | 820   | -3.5%  |
| ...    | ...   | ...    |
| Dec 24 | 660   | -3.0%  |

**Observations:**
- Consistent downward trend throughout the year
- Largest drop in Q3 (efficiency program launched)
- Slight uptick in Nov-Dec (seasonal heating demand)

**Projection:** At current rate, emissions will reach 580 tonnes
by Jun 2025 (on track for 600 target).
```

---

## Example Interaction

**User:** "How are our emissions trending?"

**Claude:**
```bash
cap model metrics list --json
# Found: Total GHG Emissions

cap templates report-templates list --json
# Found: ESG Report (rpt-123) - includes emissions

cap data time-periods list --data-interval month --json
# Getting last 12 months

cap reporting computed-values list \
  --template rpt-123 \
  --data-interval month \
  --periods "Jan 2024,Feb 2024,Mar 2024,Apr 2024,May 2024,Jun 2024,Jul 2024,Aug 2024,Sep 2024,Oct 2024,Nov 2024,Dec 2024" \
  --json
```

**Claude:**

## Emissions Trend: Last 12 Months

**Summary:** Emissions decreased 26% over the past year, from 890 to 660 tonnes.

```
890 ┤▓
850 ┤▓▓
820 ┤▓▓▓
780 ┤▓▓▓▓
750 ┤▓▓▓▓▓
720 ┤▓▓▓▓▓▓
700 ┤▓▓▓▓▓▓▓
680 ┤▓▓▓▓▓▓▓▓
660 ┤▓▓▓▓▓▓▓▓▓
    └──────────────────
     J F M A M J J A S O N D
```

**Trend:** Strong downward trajectory (-2.5%/month average)

**Key Events:**
- April: -8% (Solar installation completed)
- August: -6% (Process optimization)
- November: +3% (Seasonal heating)

---

## Variations

### Short-Term Trend

For recent changes:
- Last 3 months
- Week-over-week

### Trend Comparison

Compare trends across locations or metrics:
```
| Location | Trend | Rate |
|----------|-------|------|
| Site A   | ↘️    | -3%/mo |
| Site B   | →     | 0%/mo  |
| Site C   | ↗️    | +1%/mo |
```

### Seasonality Detection

Identify recurring patterns:
```
"Emissions show seasonal pattern:
- Winter (high): +15% vs average
- Summer (low): -10% vs average"
```

---

## Related Recipes

- [Spot Anomalies](./spot-anomalies.md) — Identify unusual values
- [Talk to Your Data](../exploration/talk-to-your-data.md) — Point-in-time analysis
- [Target Gap Analysis](./target-gap-analysis.md) — Compare to targets
