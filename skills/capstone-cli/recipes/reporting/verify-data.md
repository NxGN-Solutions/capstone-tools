# Recipe: Verify Data

> Confirm that widgets, dashboards, and reports return expected data after configuration changes.

## When to Use

- "Does this widget template have data?"
- "Verify the dashboard shows values"
- "Check if my calculation is computing"
- "Test the widget I just created"
- "Show me what data the dashboard returns"
- After creating/updating widget templates, dashboards, or calculations

## Required Context

Before starting, Claude should know:
- [ ] What to verify — a widget template, dashboard, or report template
- [ ] Which org node to query (use `cap masterdata org-nodes list` to discover)
- [ ] Which periods to check (use `cap data time-periods list --data-interval <interval>` to discover)

**If missing:** Claude will discover these interactively.

---

## Quick Reference: Which Command to Use

| I want to... | Command | Key Flags |
|--------------|---------|-----------|
| Check a **single widget template** has data | `cap reporting widgets get-data` | `--org-node`, `--periods` (auto-detects interval) |
| Check a widget with **typed output** (JSON structure) | `cap reporting widgets info-card\|pie-chart\|xy-chart\|table` | `--org-nodes`, `--data-interval`, `--periods` |
| Check **all widgets on a dashboard** at once | `cap reporting dashboards get-data` | `--org-node`, `--data-interval`, `--periods` |
| Check **computed values** via report template filter | `cap reporting computed-values list` | `--template`, `--data-interval`, `--periods` |
| **Download** computed values as Excel | `cap reporting computed-values download-excel` | `--template`, `--data-interval`, `--periods`, `-o file.xlsx` |

---

## Step-by-Step

### Step 1: Verify a Single Widget Template

**Purpose:** Confirm a widget template returns data for the expected metrics and org nodes.

**Best command: `widgets get-data`** — auto-detects the data interval from the template, returns CSV for easy reading.

```bash
cap reporting widgets get-data <widget-template-id> \
  --org-node <org-node-id> \
  --periods "2026-01-20,2026-01-21,2026-01-22" \
  --json
```

> **Key advantage:** You don't need to specify `--data-interval` — the command reads the widget template's configured interval automatically. Just provide `--periods` with the period names matching that interval.

**What to check:**
- Metric names appear in the output
- Values are non-empty for expected periods
- Values appear for the expected org nodes (including child nodes if the metric aggregates)

**Alternative — typed output for specific widget types:**

Use these when you need the structured JSON (e.g., to inspect chart series, pie slices, card items, or Table rows/columns):

```bash
# InfoCard — returns items with values, trend indicators
cap reporting widgets info-card <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval day \
  --periods "2026-01-20,2026-01-21" \
  --json

# PieChart — returns slices with proportions
cap reporting widgets pie-chart <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval day \
  --periods "2026-01-20" \
  --json

# XYChart — returns series with data points (JSON-only by design)
cap reporting widgets xy-chart <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval day \
  --periods "2026-01-20,2026-01-21,2026-01-22" \
  --json

# Table — returns dashboard table grid rows, period columns, metadata columns, and totals
cap reporting widgets table <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 26" \
  --json
```

> **Note:** Type-specific commands use `--org-nodes` (plural, comma-separated) and require `--data-interval`. The `get-data` command uses `--org-node` (singular) and auto-detects the interval.

For Table widgets, use both paths when validating full functionality: `widgets table` verifies the dashboard render contract, while `widgets get-data` verifies the legacy CSV compatibility output.

---

### Step 2: Verify an Entire Dashboard

**Purpose:** Confirm all widgets on a dashboard return data in a single call.

```bash
cap reporting dashboards get-data <dashboard-template-id> \
  --org-node <org-node-id> \
  --data-interval day \
  --periods "2026-01-20,2026-01-21,2026-01-22" \
  --json
```

**Response:** CSV containing all metric values across all widgets on the dashboard, broken down by org node and period.

**What to check:**
- All expected metrics appear (match against the dashboard's widget templates)
- Values are present — empty cells indicate missing data or misconfigured calculations
- Org node hierarchy appears (e.g., Example Enterprise, Example Enterprise->Example Site 1, etc.)
- Calculation-derived metrics have computed values (not just input metrics)

**Optional flags:**

| Flag | Description |
|------|-------------|
| `--layout-node <id>` | Check only a specific section (get the node ID from `dashboard-templates get`) |
| `--scope peers` | Include only sibling widgets (default: `descendants` = all children) |

**Example — verify a specific tab on a tabbed dashboard:**
```bash
# First, get the dashboard structure to find the tab's node ID
cap templates dashboard-templates get <dashboard-id> --json

# Then query just that tab
cap reporting dashboards get-data <dashboard-id> \
  --layout-node <tab-node-id> \
  --org-node <org-node-id> \
  --data-interval day \
  --periods "2026-01-20" \
  --json
```

---

### Step 3: Verify Computed Values via Report Template

**Purpose:** Check calculated metric values through a report template's filter lens — useful when you don't have a dashboard but want to verify calculations are producing values.

```bash
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval day \
  --periods "2026-01-20,2026-01-21" \
  --json
```

**Optional filters:**

| Flag | Description |
|------|-------------|
| `--org-nodes <id>` | Filter to specific org node |
| `--discipline-nodes <id>` | Filter to specific discipline |
| `--framework-nodes <id>` | Filter to specific framework |
| `--metric-types input` | Show only input metrics |
| `--metric-types calculation` | Show only calculated metrics |

**Download as Excel:**
```bash
cap reporting computed-values download-excel \
  --template <report-template-id> \
  --data-interval day \
  --periods "2026-01-20,2026-01-21" \
  -o computed-values.xlsx
```

---

### Step 4: Discover Available Periods

**Purpose:** Find valid period names for the data interval you're querying.

```bash
cap data time-periods list --data-interval day --json
cap data time-periods list --data-interval month --json
cap data time-periods list --data-interval quarter --json
```

Period names are what you pass to `--periods`. For daily intervals, these are dates like `2026-01-20`. For monthly, names like `Jan 2026`. For quarterly, `Q1 FY 2026`.

---

## Example Interaction

**User:** "I just created a Cola Price Premium widget — does it have data?"

**Claude:**
```bash
# First, discover the org node and recent periods
cap masterdata org-nodes list --json
# → Example Enterprise: <id>

cap data time-periods list --data-interval day --json
# → Recent: 2026-01-20, 2026-01-21, 2026-01-22

# Verify widget data (auto-detects daily interval from template)
cap reporting widgets get-data <id> \
  --org-node <id> \
  --periods "2026-01-20,2026-01-21,2026-01-22" \
  --json
```

**Result:**
```
Org Node Path, Metric Name, ..., 2026-01-20, 2026-01-21, 2026-01-22
Example Enterprise, Cola Price Premium %, ..., 1.3, 14.1, -7.5
Example Enterprise->Example Site 1, Cola Price Premium %, ..., 3.6, 24.7, -0.7
Example Enterprise->Example Site 2, Cola Price Premium %, ..., -7.8, 4.6, -19.1
Example Enterprise->Example Site 3, Cola Price Premium %, ..., -22.4, , 5.0
```

Widget is returning data at all org levels. Example Site 3 on Jan 21 is empty — likely no Orange RPU data for that day.

---

## Troubleshooting: No Data

If values are empty:

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| All values empty for a calculation | Calculation engine hasn't processed yet | Wait 15-30 seconds and retry |
| All values empty after waiting | Formula references a non-existent metric ID | Check formula with `cap model calculations get <id> --json` and verify all `[id]` references exist in `cap model metrics list --json` |
| Values appear after simple formula but not complex | Calculation engine stuck from prior bad formula | Save a simplified formula first (remove IF/THEN), wait for values, then restore full formula |
| Input metric has values but calculation doesn't | `calculationPhase` may be wrong | Ensure `After Aggregations` (id: 1) for formulas that reference other calculations |
| Values at enterprise level but not sites | `orgStructureAggregationMethod: None` | Expected — means the metric doesn't roll up. Check if the formula references site-level inputs |
| Validator reports "Missing dependency" | False positive (known issue) | Don't rely on validator — check actual computed values instead |

---

## Related Recipes

- [Build Dashboard](../configuration/build-dashboard.md) — Create dashboards (Step 6 covers data verification)
- [Create Widget Template](../configuration/create-widget-template.md) — Create widgets
- [Talk to Your Dashboard](../exploration/talk-to-your-dashboard.md) — Analyze dashboard data in depth
- [Generate Narrative](./generate-narrative.md) — Write summaries from computed values
