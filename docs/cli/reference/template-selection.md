# Template Selection Guide

> Choose the right template type quickly, then run the matching CLI flow.

---

## Runtime Setup

```bash
cap config set api-url <capstone-api-url>
cap auth whoami
```

---

## Decision Matrix

| If you need to... | Use | Primary CLI |
|---|---|---|
| Enter raw values | **Capture Template** | `templates capture-templates`, `data input-values` |
| Analyze calculated/aggregated values in spreadsheet form | **Report Template** | `templates report-templates`, `reporting computed-values` |
| Define reusable chart/card behavior | **Widget Template** | `templates widget-templates`, `reporting widgets *` |
| Compose sections/tabs of widgets for end users | **Dashboard Template** | `templates dashboard-templates`, `reporting dashboards *` |
| Change reporting hierarchy lens | **Org Node Template** (used by report/dashboard templates) | `templates org-node-templates` |

---

## End-to-End Flows

### 1. Data Entry Flow (Capture Templates)

Use when users must submit raw `InputValues`.

```bash
# Discover templates
cap templates capture-templates list --json

# Discover periods for the intended interval
cap data time-periods list --data-interval month --json

# List entered values through a capture template lens
cap data input-values list \
  --template <capture-template-id> \
  --data-interval month \
  --periods "Jan 2026" \
  --json
```

#### Capture Template Design Patterns (for lean data entry)

Use these patterns when the goal is fast, low-noise manual capture.

| Capture Goal | Recommended Template Shape | Why |
|---|---|---|
| Single-location weekly entry | `DataGrouping=OrgNode`, `ShowOrgNodeFilter=true`, `AllowMultiOrgNodeSelect=false`, fixed `DataInterval` | Keeps scope to one selected location and one interval |
| Multi-location side-by-side entry | `DataGrouping=OrgNode`, `ShowOrgNodeFilter=true`, `AllowMultiOrgNodeSelect=true` | Supports consistent entry across multiple locations |
| Focused input set (avoid dense model noise) | Constrain by `Disciplines` and/or explicit `Metrics` | Prevents large grids from unrelated metrics |
| Stable import/export behavior | Prefer row-based layout first (`ShowMetricsInColumns=false`) | Easier to validate and troubleshoot |

Minimal verification loop before adopting a capture template:

```bash
# 1) Pull the template as users will use it
cap data input-values list \
  --template <capture-template-id> \
  --data-interval week \
  --periods "(W6) Feb 2 2026" \
  --json > /tmp/capture-check.json

# 2) Count data rows (capture surface area)
jq '[.gridRows[] | select(.isDataRow==true)] | length' /tmp/capture-check.json

# 3) Confirm expected metrics are present
jq -r '[.gridRows[] | select(.isDataRow==true) | .name] | unique' /tmp/capture-check.json
```

If the row count is much larger than expected, tighten template filters (org nodes, disciplines, metrics) before using it in production or demos.

### 2. Spreadsheet Analysis Flow (Report Templates)

Use when users need computed/aggregated numbers for review or export.

```bash
# Discover report templates
cap templates report-templates list --json

# Query computed values through report template filters
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval month \
  --periods "Jan 2026" \
  --json

# Optional: export computed values
cap reporting computed-values download-excel \
  --template <report-template-id> \
  --data-interval month \
  -o computed-values.xlsx
```

### 3. Visualization Flow (Widget Templates)

Use when building reusable chart/card definitions.

```bash
# Discover widget templates
cap templates widget-templates list --json

# Verify rendered widget data (type-agnostic, CSV payload)
cap reporting widgets get-data <widget-template-id> \
  --org-node <org-node-id> \
  --periods "Jan 2026" \
  --json

# Type-specific data (typed JSON payload)
cap reporting widgets info-card <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval month \
  --periods "Jan 2026" \
  --json

cap reporting widgets table <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 26" \
  --json
```

Use `widgets table` when validating the dashboard render shape (`columns`, `columnGroups`, `rows`, budgets, truncation, and warnings). Use `widgets get-data` when an agent needs the generic CSV extraction payload and does not need dashboard table-render metadata.

### 4. Dashboard Composition Flow (Dashboard Templates)

Use when assembling multiple widgets into tabs/sections.

```bash
# Discover dashboard templates
cap templates dashboard-templates list --json

# Validate full dashboard data output
cap reporting dashboards get-data <dashboard-template-id> \
  --org-node <org-node-id> \
  --data-interval month \
  --periods "Jan 2026" \
  --json
```

---

## Common Agent Mistakes

| Mistake | Correct Pattern |
|---|---|
| Using `capture-templates` to inspect calculated values | Use `reporting computed-values list --template <report-template-id>` |
| Expecting `report-templates` to support raw data entry workflow | Use `data input-values *` with a `capture-template` |
| Mixing `--org-node` and `--org-nodes` for widgets | `widgets get-data` uses `--org-node`; type-specific widgets use `--org-nodes` |
| Forgetting period discovery | Run `cap data time-periods list --data-interval <interval>` first |
| Running docs examples before installing or extracting the CLI | Download the release package and use `cap` or `.\cap.exe` |

---

## Related Docs

- [Commands Reference](./commands.md)
- [Glossary](./glossary.md)
