# MCP Prompts Reference

> User-invocable guided workflows that appear as slash commands in Claude.

---

## How Prompts Work

Prompts are **instruction templates** that users invoke via slash commands (e.g., `/target-gap-analysis`). When invoked, Claude receives a structured workflow with step-by-step instructions for using MCP tools.

| Behavior | Details |
|----------|---------|
| **Invoked by** | User (via slash command menu) |
| **Purpose** | Guide multi-step analysis workflows |
| **Output** | Instructions for Claude to follow |
| **Tools used** | Each prompt orchestrates multiple tool calls |

Not every workflow needs a prompt. Some workflows (like answering ad-hoc metric questions) are better served by following a [recipe](../recipes/README.md) using direct tool calls.

---

## Analysis Prompts (4)

### `/target-gap-analysis`

Compare actual performance against targets to identify gaps.

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `metricNames` | string | No | Comma-separated metric names (default: all metrics with targets) |
| `period` | string | No | Time period (e.g., "Q1 FY 25", default: most recent) |

**Orchestrates:** `model_metrics_list` → `templates_spreadsheetReports_list` → `data_computedValues_list` → gap calculation → status report

**Recipe:** [Target Gap Analysis](../recipes/analysis/target-gap-analysis.md) | **CLI:** [Target Gap Analysis](../../cli/recipes/analysis/target-gap-analysis.md)

---

### `/spot-anomalies`

Detect statistical outliers in the data.

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `period` | string | No | Time range (e.g., "last 12 months") |
| `sensitivity` | string | No | `low` (3 std dev), `medium` (2, default), `high` (1.5) |

**Orchestrates:** `model_metrics_list` → `templates_spreadsheetReports_list` → `data_computedValues_list` (current + historical) → statistical analysis → anomaly report

**Recipe:** [Spot Anomalies](../recipes/analysis/spot-anomalies.md) | **CLI:** [Spot Anomalies](../../cli/recipes/analysis/spot-anomalies.md)

---

### `/compare-operations`

Compare performance between two locations or organizational units.

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `location1` | string | Yes | First location/org node name |
| `location2` | string | Yes | Second location/org node name |

**Orchestrates:** `model_orgNodes_list` → `templates_spreadsheetReports_list` → `data_computedValues_list` (per location) → comparison table

**Recipe:** [Compare Operations](../recipes/analysis/compare-operations.md) | **CLI:** [Compare Operations](../../cli/recipes/analysis/compare-operations.md)

---

### `/find-trends`

Analyze time-series trends in the data.

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `metricNames` | string | No | Comma-separated metric names (default: key metrics) |
| `periods` | string | No | Time range (e.g., "last 12 months", "FY 2024") |

**Orchestrates:** `model_metrics_list` → `data_timePeriods_list` → `templates_spreadsheetReports_list` → `data_computedValues_list` → trend analysis → projections

**Recipe:** [Find Trends](../recipes/analysis/find-trends.md) | **CLI:** [Find Trends](../../cli/recipes/analysis/find-trends.md)

---

## Exploration Prompts (2)

### `/discover-structure`

Explore the tenant's organizational structure, metrics, disciplines, and frameworks.

No arguments.

**Orchestrates:** `model_orgNodes_list` → `model_disciplines_list` → `model_frameworks_list` → `model_metrics_list` → structured summary

**Recipe:** [Discover Structure](../recipes/exploration/discover-structure.md) | **CLI:** [Discover Tenant Structure](../../cli/recipes/exploration/discover-tenant-structure.md)

---

### `/check-completeness`

Audit data completeness for a specific time period.

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `period` | string | Yes | Time period to check (e.g., "Q4 2024", "FY 2024") |
| `orgNode` | string | No | Org node name to scope the check |

**Orchestrates:** `data_availability` → `model_metrics_list` → gap identification → completeness percentage → action items

**Recipe:** [Check Completeness](../recipes/exploration/check-completeness.md) | **CLI:** [Check Data Completeness](../../cli/recipes/exploration/check-completeness.md)

---

## Reporting Prompts (2)

### `/talk-to-dashboard`

Explore and interpret a dashboard's data and visualizations.

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `dashboardName` | string | Yes | Name of the dashboard to explore |

**Orchestrates:** `templates_dashboards_list` → `reporting_dashboards_getData` → section interpretation → cross-metric insights → key takeaways

**Recipe:** [Talk to Dashboard](../recipes/exploration/talk-to-dashboard.md) | **CLI:** [Talk to Your Dashboard](../../cli/recipes/exploration/talk-to-your-dashboard.md)

---

### `/generate-narrative`

Generate an AI-written performance summary suitable for reports.

| Argument | Type | Required | Description |
|----------|------|----------|-------------|
| `dashboardName` | string | Yes | Name of the dashboard to summarize |
| `period` | string | Yes | Time period (e.g., "Q4 2024") |

**Orchestrates:** `reporting_dashboards_getData` → `apps_widget_aiSummary` → professional narrative with executive summary, highlights, concerns, recommendations

**Recipe:** [Generate Narrative](../recipes/reporting/generate-narrative.md) | **CLI:** [Generate Narrative](../../cli/recipes/reporting/generate-narrative.md)

---

## Workflows Without Dedicated Prompts

These workflows use direct tool calls and are documented as recipes:

| Recipe | Description |
|--------|-------------|
| [Talk to Your Data](../recipes/exploration/talk-to-data.md) | Answer natural-language metric questions using direct tool calls |

---

## Prompts Not Available in MCP v1

These CLI recipes have no MCP equivalent yet:

| CLI Recipe | Why Not in MCP v1 |
|------------|-------------------|
| [Dashboard Comparison](../../cli/recipes/analysis/dashboard-comparison.md) | Requires period selection UI |
| [Deep Dashboard Analysis](../../cli/recipes/analysis/deep-dashboard-analysis.md) | Complex cross-metric correlation |
| [Create Metric Wizard](../../cli/recipes/configuration/create-metric.md) | v1 is read-only |
| [Create Widget Template](../../cli/recipes/configuration/create-widget-template.md) | v1 is read-only |
| [Enter Data](../../cli/recipes/data-management/enter-data.md) | Use `data_inputValues_save` directly |
| [Bulk Import Data](../../cli/recipes/data-management/bulk-import.md) | Excel operations deferred |
| [Review Approvals](../../cli/recipes/data-management/review-approvals.md) | Change request workflow deferred |

---

## See Also

- [SKILL.md](../SKILL.md) — Prompt routing table
- [Glossary](./glossary.md) — Term definitions with MCP tool references
- [Recipes](../recipes/README.md) — Detailed workflow guides for each prompt
- [Tools Reference](./tools.md) — Tools used by prompts
