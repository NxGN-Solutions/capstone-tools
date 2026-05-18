# Capstone MCP Skill

> Teach Claude to use Capstone via MCP tools for data analysis and visualization.

## What is Capstone?

Capstone is a **data management platform** for organizations that need to consolidate, curate, and report on data from multiple sources. While commonly used for ESG (Environmental, Social, Governance) reporting, Capstone is domain-agnostic and supports any integrated reporting use case including operational KPIs, financial planning, and compliance monitoring.

**Key capabilities:**
- **Data consolidation** from SQL databases, MQTT brokers, REST APIs, and manual entry
- **Data curation** with multi-level validation workflows for assurance
- **Audit assurance** via data locking and change request tracking
- **Flexible modeling** with user-defined KPIs, formulas, targets, budgets, actuals, and scorecards
- **Analytics** through dashboards, widgets, and AI-powered summaries

---

## Concept Glossary

| You Say | Capstone Calls It | MCP Tool | Notes |
|---------|-------------------|----------|-------|
| Location, site, facility, operation | Org Node | `model_orgNodes_list` | Hierarchical tree structure |
| KPI, measure, indicator, value | Metric | `model_metrics_list` | Parent type for Input and Calculation |
| Manual data point, entered value | Input | `model_metrics_list` (includeCalculations: false) | Captured via data entry |
| Formula-based value, derived metric | Calculation | `model_metrics_list` (includeInputs: false) | Computed from formulas |
| Category, type, topic | Discipline | `model_disciplines_list` | E.g., Environmental, Social, Governance |
| Reporting standard, disclosure framework | Framework | `model_frameworks_list` | GRI, SASB, CDP, TCFD, etc. |
| Unit, measurement, UoM | Unit of Measure | Via `model_metrics_list` / `model_metrics_get` | kWh, tonnes CO2e, litres, count |
| Time period, date range, interval | Time Period | `data_timePeriods_list` | Day, Week, Month, Quarter, Year |
| Raw data, submitted value | Input Value | `data_inputValues_list` | Actual captured data points |
| Aggregated result, computed | Computed Value | `data_computedValues_list` | Calculated/rolled-up values |
| Dashboard widget, visualization | Widget | `apps_widget_*` | Info Card, Pie Chart, XY Chart, Table |
| Chart configuration, widget setup | Widget Template | `templates_widgets_list` | Defines widget data and display |
| Dashboard layout, page configuration | Dashboard Template | `templates_dashboards_list` | Organizes widgets into sections |
| Data entry form, capture sheet | Capture Template | `templates_spreadsheetCaptures_list` | Defines data entry structure |
| Report layout, spreadsheet template | Report Template | `templates_spreadsheetReports_list` | Defines report structure |
| Edit request, correction request | Change Request | Not in MCP v1 | Use CLI for change requests |
| Period freeze, lock, close period | Data Lock | Not in MCP v1 | Use CLI for data locks |
| Organization, company, client | Tenant | `ListTenants`, `SwitchTenant` | Isolated data environment |

> **Full glossary:** [reference/glossary.md](reference/glossary.md)

---

## How MCP Works

MCP provides three integration mechanisms:

| Mechanism | How It Works | User Experience |
|-----------|-------------|-----------------|
| **Tools** (41) | You call them directly — invisible to user | User sees results, not tool calls |
| **Resources** (7) | Auto-loaded context about the tenant | You already know the org structure, metrics, etc. |
| **Prompts** (8) | User invokes via slash commands (e.g., `/target-gap-analysis`) | Guided multi-step workflows |

**Tools** are your primary interface. Call them as needed to answer user questions.
**Resources** eliminate discovery calls — check resource data before calling tools.
**Prompts** guide complex analysis workflows when the user invokes them.

---

## Quick Start

### 1. Verify Authentication

Call `Whoami` to confirm the user is authenticated and see their tenant context.

### 2. Check Resources

Resources provide pre-loaded context:
- `capstone://model/metrics` — What metrics exist (names, types, units)
- `capstone://model/organization` — Org hierarchy (sites, regions)
- `capstone://model/disciplines` — Metric categories
- `capstone://model/frameworks` — Reporting frameworks
- `capstone://data/availability` — Which periods have data
- `capstone://user/context` — Current user and tenant

### 3. Query Data

All data queries need three things: **what** (template/metrics), **where** (org node), **when** (time periods).

```
templates_spreadsheetReports_list  → pick a report template
model_orgNodes_list                → pick org node(s)
data_timePeriods_list              → pick time period(s)
data_computedValues_list           → get the data
```

---

## Asking Questions

Always clarify before executing when the user's request is missing key context:

| Missing | Ask |
|---------|-----|
| Time period | "Which time period? (e.g., Q1 2025, last 6 months, FY 2024)" |
| Location/Org | "Which location or operation? (e.g., all sites, specific facility)" |
| Metric type | "What specifically do you want to measure? (e.g., electricity, water, safety incidents)" |
| Comparison basis | "Compare against what? (another site, last year, target)" |
| Dashboard name | "Which dashboard should I analyze?" |
| Report scope | "For what audience? (board summary, detailed management report, external disclosure)" |

**When the user is vague about metrics:**
```
"You asked about 'emissions.' I found several emission-related metrics:
1. Scope 1 Emissions (direct)
2. Scope 2 Emissions (electricity)
3. Total GHG Emissions

Which would you like to analyze?"
```

**When the user doesn't specify a period:**
Check `capstone://data/availability` or `data_availability` to find the most recent period with data, and confirm: "I'll use the most recent quarter with data (Q4 2024). Is that right?"

---

## Prompt-First Approach

When the user's request matches a prompt, use it. Prompts encode best-practice workflows.

| User Says | Prompt | Recipe |
|-----------|--------|--------|
| "Are we on track for targets?" | `/target-gap-analysis` | [Recipe](recipes/analysis/target-gap-analysis.md) |
| "Any unusual values?" | `/spot-anomalies` | [Recipe](recipes/analysis/spot-anomalies.md) |
| "Compare Site A vs Site B" | `/compare-operations` | [Recipe](recipes/analysis/compare-operations.md) |
| "How is [metric] trending?" | `/find-trends` | [Recipe](recipes/analysis/find-trends.md) |
| "Explain the [dashboard]" | `/talk-to-dashboard` | [Recipe](recipes/exploration/talk-to-dashboard.md) |
| "What data is missing?" | `/check-completeness` | [Recipe](recipes/exploration/check-completeness.md) |
| "Write a summary of..." | `/generate-narrative` | [Recipe](recipes/reporting/generate-narrative.md) |
| "What's in this tenant?" | `/discover-structure` | [Recipe](recipes/exploration/discover-structure.md) |
| "Analyze our [metric] data" | — (no prompt) | [Recipe](recipes/exploration/talk-to-data.md) |

> **Full prompt reference:** [reference/prompts.md](reference/prompts.md)
> **Detailed recipes:** [recipes/README.md](recipes/README.md)

---

## Tool Categories

| Category | Tools | Purpose |
|----------|-------|---------|
| **Auth** (6) | `Login`, `Logout`, `Whoami`, `ListTenants`, `SwitchTenant`, `Languages` | Authentication and tenant management |
| **Model** (5) | `model_metrics_list`, `model_metrics_get`, `model_orgNodes_list`, `model_frameworks_list`, `model_disciplines_list` | Read model structure |
| **Data** (5) | `data_timePeriods_list`, `data_availability`, `data_inputValues_list`, `data_inputValues_save`, `data_computedValues_list` | Query and save data |
| **Reporting** (2) | `reporting_dashboards_getData`, `reporting_widgets_getData` | Dashboard/widget data as CSV |
| **Templates** (6) | `templates_dashboards_list`, `templates_widgets_list`, `templates_spreadsheetReports_list`, `templates_spreadsheetCaptures_list`, `templates_spreadsheetReports_get`, `templates_spreadsheetCaptures_get` | Discover and inspect templates |
| **Template CRUD** (10) | `templates_widgets_create/save/delete`, `templates_reports_create/save/delete`, `templates_dashboards_get/create/save/delete` | Create, update, and delete widget/report/dashboard templates |
| **Apps** (7) | `apps_dashboard_render`, `apps_widget_infoCard`, `apps_widget_pieChart`, `apps_widget_xyChart`, `apps_widget_table`, `apps_widget_aiSummary`, `apps_chart_render` | Visual widgets rendered in conversation |
| **Status** (1) | `GetStatus` | Server status and auth check |

> **Full tool reference:** [reference/tools.md](reference/tools.md)

---

## Tool Selection Guide

> Also available as a resource: `capstone://guide/tool-selection`

When deciding which tool to use, follow this decision tree:

| User Wants | Best Tool | Why |
|------------|-----------|-----|
| **Monthly/quarterly/yearly summary** | `data_computedValues_list` (dataInterval=2/3/4) | Flexible aggregation control |
| **Weekly trends** | `data_computedValues_list` (dataInterval=1) | Flexible aggregation control |
| **Daily detail / anomaly detection** | `data_computedValues_list` (dataInterval=0) | Flexible aggregation control |
| **Cross-site comparison** | `data_computedValues_list` (per-site orgNodeIds) | Flexible aggregation control |
| **SEE a dashboard** | `apps_dashboard_render` | Visual HTML rendering |
| **SEE a single widget** | `apps_widget_pieChart` / `apps_widget_xyChart` / `apps_widget_infoCard` / `apps_widget_table` | Visual HTML rendering |
| **Full dashboard CSV export** | `reporting_dashboards_getData` | Returns at widget native interval |
| **Check one widget's values** | `reporting_widgets_getData` | Returns at widget's configured interval |
| **Data quality / validation** | `data_inputValues_list` | Shows validation status + lock state |
| **Edit/enter data** | `data_inputValues_save` | Upsert by business key |
| **AI insights** | `apps_widget_aiSummary` | Pre-generated server insights |

**Key difference:** `reporting_dashboards_getData` and `reporting_widgets_getData` return data at the widget's *native* interval — you cannot control aggregation. For flexible granularity (daily/weekly/monthly), always use `data_computedValues_list` with the `dataInterval` parameter.

**~80% of data questions** should route to `data_computedValues_list` — it's the primary analysis tool.

---

## MCP Apps

MCP Apps render interactive visualizations directly in the conversation. Use them when users want to **see** data, not just read numbers.

| Tool | Renders | When to Use |
|------|---------|-------------|
| `apps_dashboard_render` | Full dashboard with all widgets | "Show me the ESG dashboard" |
| `apps_widget_infoCard` | Single KPI card | "What's our emissions number?" |
| `apps_widget_pieChart` | Pie/donut chart | "Show the energy breakdown" |
| `apps_widget_xyChart` | Bar/line/column chart | "Show the emissions trend" |
| `apps_widget_table` | Table widget | "Show the metric table" |
| `apps_widget_aiSummary` | AI-generated summary card | "Summarize the dashboard insights" |

**When to use Apps vs raw data:**
- Use `apps_widget_*` when the user wants a **visual** answer
- Use `data_computedValues_list` when you need to **analyze** or **compare** numbers
- Use both when you want to show a chart AND provide analysis

---

## Query Patterns

### Discovery → Retrieval → Analysis

```
1. DISCOVER: What metrics/templates/org nodes exist?
   → model_metrics_list, templates.*.list, model_orgNodes_list

2. RETRIEVE: Get the actual data
   → data_computedValues_list, reporting_dashboards_getData

3. ANALYZE: Interpret and present
   → Calculate gaps, trends, anomalies, comparisons

4. VISUALIZE (optional): Render widgets
   → apps_widget_*, apps_dashboard_render
```

### Common Query Parameters

| Parameter | Source | Used By |
|-----------|--------|---------|
| `orgNodeId` | `model_orgNodes_list` | Most data and reporting tools |
| `timePeriodNames` | `data_timePeriods_list` | Data queries (comma-separated period names) |
| `periodType` + `periodCount` | — | Alternative to timePeriodNames (e.g., "quarter" + 4) |
| `dataInterval` | Enum: Day(0), Week(1), Month(2), Quarter(3), Year(4) | All data/reporting tools |
| `dashboardTemplateId` | `templates_dashboards_list` | Dashboard tools |
| `widgetTemplateId` | `templates_widgets_list` | Widget tools |
| `templateId` | `templates_spreadsheetReports_list` | Computed value and input value queries |

> **Note:** Data tools use `timePeriodNames` or `periodType+periodCount` for period selection. App tools use `startDate`/`endDate` (ISO format). See [tools reference](reference/tools.md) for details.

> **Full data model reference:** [reference/data-model.md](reference/data-model.md)

---

## Analysis Techniques

When analyzing retrieved data, use these techniques based on the user's question:

| Question Type | Technique | Example |
|---------------|-----------|---------|
| "How much?" | Sum values for period | Total emissions: 2,450 tonnes |
| "Compared to?" | Calculate percentage difference | Down 8% from Q4 |
| "Trending?" | Linear regression on time series | Decreasing at -2.5% per month |
| "Highest/lowest?" | Sort and identify extremes | Site C has highest water use |
| "On track?" | Actual vs target, gap % | 7.5% under budget — On Track |
| "Normal?" | Mean + standard deviation from history | Value is 3.2 std dev above mean |
| "Why?" | Compare to historical baseline, identify outliers | Spike coincides with new equipment |
| "What's missing?" | Expected vs captured data points | 38 of 450 data points missing |

---

## Error Handling

| Error | Cause | Recovery |
|-------|-------|----------|
| Authentication required | Not logged in | Call `Login` |
| Tenant selection required | Multiple tenants, none selected | Call `ListTenants` then `SwitchTenant` |
| Not found | Invalid ID | Verify with list tools |
| Validation error | Invalid parameters | Check parameter types and required fields |
| Server error | API issue | Suggest retry; check if API is running |

**Validation errors return details:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": {
      "Name": ["Name is required"],
      "UnitOfMeasureId": ["Invalid unit of measure"]
    }
  }
}
```

---

## MCP v1 Availability

| Feature | Available | Notes |
|---------|-----------|-------|
| Authentication (Login, Logout, Whoami, Tenants) | Yes | Full OAuth + tenant switching |
| Model read (Metrics, Org Nodes, Disciplines, Frameworks) | Yes | Read-only |
| Data read (Time Periods, Availability, Input Values, Computed Values) | Yes | Full query support |
| Data write (Input Values save) | Yes | Business-key upsert |
| Reporting (Dashboard data, Widget data) | Yes | CSV + visualization |
| MCP Apps (Dashboard render, Widget charts, AI Summary) | Yes | Interactive HTML in conversation |
| Templates (List + Get for all template types) | Yes | Read-only |
| Resources (6 auto-loaded contexts) | Yes | Cached model + data availability |
| Prompts (8 guided workflows) | Yes | Slash command invocation |
| Excel import/export | No | Use CLI |
| Change requests | No | Use CLI |
| Data locks | No | Use CLI |
| Metric/entity creation | No | Use CLI |
| User/role management | No | Use CLI |
| Attribute types | No | Use CLI |
| Metric overrides | No | Use CLI |

---

## Feature-to-Tool Mapping

| Feature (docs/features/) | MCP Tools |
|--------------------------|-----------|
| metrics/ | `model_metrics_list`, `model_metrics_get` |
| org-nodes/ | `model_orgNodes_list` |
| disciplines/ | `model_disciplines_list` |
| frameworks/ | `model_frameworks_list` |
| time-periods/ | `data_timePeriods_list` |
| data-management/ | `data_inputValues_list`, `data_inputValues_save` |
| reporting/ | `data_computedValues_list`, `reporting.dashboards.*` |
| dashboards/ | `apps_dashboard_render`, `apps_widget_*` |
| widget-templates/ | `templates_widgets_list` |
| dashboard-templates/ | `templates_dashboards_list` |
| spreadsheet-templates/ | `templates_spreadsheetReports_list`, `templates_spreadsheetCaptures_list` |

---

## See Also

- [CLI Skill](../cli/SKILL.md) — CLI interface (for Claude Code with Bash access)
- [Recipes](./recipes/README.md) — Step-by-step workflow guides
- [Tool Reference](./reference/tools.md) — All 41 tools with parameters and response formats
- [Glossary](./reference/glossary.md) — Detailed term definitions with MCP tools
- [Data Model](./reference/data-model.md) — Enums, aggregation, query patterns
- [Setup Guide](./setup.md) — Claude Desktop configuration (human-facing)
