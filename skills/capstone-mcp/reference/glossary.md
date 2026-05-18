# Glossary

> Standard terms → Capstone terminology mapping with MCP tools.

---

## Data Modeling

### KPI / Metric / Measure / Indicator

**Capstone Term:** Metric

**Definition:** A quantifiable measure that tracks performance. In Capstone, Metric is the parent type for both Input (manual data) and Calculation (formula-derived).

**MCP Tools:**
- `model_metrics_list` — List all metrics (inputs + calculations)
- `model_metrics_get` — Get full details for one metric

**Resource:** `capstone://model/metrics` — Pre-loaded metric hierarchy (check before calling tools).

**Key fields:** `name`, `metricType` (Input/Calculation), `discipline`, `unitOfMeasure`, `dataInterval`.

---

### Location / Site / Facility / Operation

**Capstone Term:** Org Node

**Definition:** A node in the organizational hierarchy representing a physical location, business unit, or reporting entity. Org Nodes form a tree structure for data aggregation.

**MCP Tools:**
- `model_orgNodes_list` — List full hierarchy

**Resource:** `capstone://model/organization` — Pre-loaded org tree.

**Key fields:** `name`, `level`, `treePath` (e.g., "Corporate > Region 1 > Site A").

---

### Category / Topic / Type

**Capstone Term:** Discipline

**Definition:** A hierarchical categorization of metrics (e.g., Environmental, Social, Governance). Disciplines form a tree structure.

**MCP Tools:**
- `model_disciplines_list` — List all disciplines

**Resource:** `capstone://model/disciplines` — Pre-loaded discipline tree.

---

### Reporting Standard / Disclosure Framework

**Capstone Term:** Framework

**Definition:** An external reporting standard that metrics can be mapped to (e.g., GRI, SASB, CDP, TCFD). Frameworks have nodes representing disclosure topics.

**MCP Tools:**
- `model_frameworks_list` — List all frameworks

**Resource:** `capstone://model/frameworks` — Pre-loaded framework tree.

---

### Unit / Measurement / UoM

**Capstone Term:** Unit of Measure

**Definition:** The measurement unit for metric values (e.g., kWh, tonnes CO2e, litres, count).

**MCP Tools:**
- Available via `model_metrics_list` (each metric includes its unit)
- Available via `model_metrics_get` for individual metric detail

---

### Manual Data / Entered Value

**Capstone Term:** Input (Metric)

**Definition:** A metric that captures manually entered or automatically ingested data points. Inputs are the source data for calculations.

**MCP Tools:**
- `model_metrics_list` with `includeInputs: true, includeCalculations: false` — List input metrics only

---

### Formula / Derived Metric / Calculated Value

**Capstone Term:** Calculation (Metric)

**Definition:** A metric whose values are computed from a formula referencing other metrics. Formulas use Excel-like syntax with `[Metric Name]` or `[GUID]` references.

**MCP Tools:**
- `model_metrics_list` with `includeInputs: false, includeCalculations: true` — List calculations only
- `model_metrics_get` — View formula and calculation settings

**Formula syntax:**
```
[Metric Name]                     Reference another metric by name
IF condition THEN x ELSE y        Conditional logic
SUM([A], [B], [C])                Sum of values
+ - * / ^ %                       Arithmetic operators
```

---

## Data Capture

### Raw Value / Submitted Data / Data Point

**Capstone Term:** Input Value

**Definition:** An actual data entry for an Input metric at a specific Org Node and Time Period. Input Values are stored with validation status and audit trail.

**Business Key:** `metric` + `orgNode` + `timePeriodType` + `startDate`. This combination uniquely identifies an input value. The save tool uses business-key upsert — zero GUIDs resolve to existing records automatically, making saves idempotent.

**MCP Tools:**
- `data_inputValues_list` — List values for a capture template
- `data_inputValues_save` — Create or update values (batch upsert)

---

### Time Period / Date Range / Interval

**Capstone Term:** Time Period

**Definition:** A time interval for data capture (Day, Week, Month, Quarter, Year). Each metric has a defined data interval.

**MCP Tools:**
- `data_timePeriods_list` — List available periods for a given interval

**Resource:** `capstone://data/availability` — Which periods have data.

**Period name formats:**
| Interval | Example Names |
|----------|---------------|
| Day | `2026-01-15` |
| Week | `(W5) Jan 26 2026` |
| Month | `Jan 2026` |
| Quarter | `Q1 FY 25` |
| Year | `FY 2025` |

---

### Approval / Validation / Sign-off

**Capstone Term:** Validation Level

**Definition:** Multi-level approval workflow for data quality assurance. Each Input Value progresses through validation levels before being "approved."

**Validation Status Values:**

| ID | Name | Description |
|----|------|-------------|
| 0 | Draft | Value entered, not yet submitted |
| 1 | Pending | Submitted, awaiting approval |
| 2 | Approved | Validated and accepted |
| 3 | Rejected | Reviewed and rejected |

---

### Edit Request / Correction Request

**Capstone Term:** Change Request

**Definition:** An audit-tracked request to modify locked or validated data. Change Requests require approval and maintain complete history.

**MCP Tools:** Not available in MCP v1 — use the CLI for change request workflows.

---

### Period Freeze / Lock / Close Period

**Capstone Term:** Data Lock

**Definition:** Prevents modifications to data within a specified time period and org scope. Used for audit assurance.

**MCP Tools:** Not available in MCP v1 — use the CLI for data lock operations.

---

## Reporting & Analytics

### Aggregated Value / Result / Computed

**Capstone Term:** Computed Value

**Definition:** The calculated or aggregated value of a metric at a specific Org Node and Time Period. Includes both formula calculations and hierarchical rollups. Computed values are read-only.

**MCP Tools:**
- `data_computedValues_list` — Query values through a report template's filter lens
- `reporting_dashboards_getData` — Get all widget data for a dashboard as CSV

**Important:** Computed values require a `templateId` (report template). The template's filters (discipline, metric type, framework) determine which metrics appear in the output.

---

### Chart / Visualization

**Capstone Term:** Widget

**Definition:** A visual representation of data. Four data widget types exist:

| Type | Best For | MCP Tool |
|------|----------|----------|
| InfoCard | Single KPI value with comparison | `apps_widget_infoCard` |
| PieChart | Part-to-whole breakdown | `apps_widget_pieChart` |
| XYChart | Time series, trends, multi-metric comparison | `apps_widget_xyChart` |
| Table | Tabular metric rows and formatted values | `apps_widget_table` |

**MCP Apps:** Use `apps_widget_*` tools to render interactive visualizations directly in the conversation.

---

### Dashboard / Report Page

**Capstone Term:** Dashboard

**Definition:** A collection of widgets arranged in sections for data visualization. Dashboards are configured via Dashboard Templates.

**MCP Tools:**
- `apps_dashboard_render` — Render full interactive dashboard in conversation
- `reporting_dashboards_getData` — Get all widget data as CSV for analysis
- `apps_widget_aiSummary` — AI-generated dashboard insights

---

### Rollup / Aggregation

**Capstone Term:** Aggregation Method

**Definition:** How metric values combine across time or org hierarchy. Each metric has both an Org Structure and Time Period aggregation method.

**Org Structure Aggregation:**

| ID | Name | Use When |
|----|------|----------|
| 0 | Sum | Values add up (consumption, cost, throughput) |
| 1 | Average | Values average across children (rates, scores, %) |
| 2 | Count | Count of non-null child values |
| 3 | None | Node-specific, doesn't roll up |
| 4 | Roll Up | Uses first non-null child value |
| 5 | Roll Down | Cascades parent value to children (assumptions, targets) |

**Time Period Aggregation:**

| ID | Name | Use When |
|----|------|----------|
| 0 | None | Each period shown separately (time series) |
| 1 | Sum | Values sum across periods (cumulative metrics) |
| 2 | Average | Values average across periods (rates, scores) |
| 3 | Last Value | Most recent period only (snapshots) |
| 4 | Min | Minimum value across periods |
| 5 | Max | Maximum value across periods |

**Common Patterns:**

| Metric Type | Org Agg | Time Agg | Example |
|-------------|---------|----------|---------|
| Throughput/Volume | Sum | Sum | Electricity kWh across sites and months |
| Cost/Revenue | Sum | Sum | Financial totals |
| Rates/Scores (%) | Average | Average | OEE, safety scores |
| Assumptions | Roll Down | Average | Budget assumptions cascade down |
| Targets | Roll Down | Average | Annual targets inherited |
| Per-unit ratios | None | None | Recomputed from aggregated inputs |
| Headcount/Balance | Sum or None | Last Value | Point-in-time snapshots |

> **Interpretation:** When analyzing data, check the aggregation method to understand what a value means at different org levels. A "Sum" metric at a parent node is the total of all children. An "Average" metric is the mean across children.

---

## Configuration

### Data Entry Form / Capture Sheet

**Capstone Term:** Capture Template (Spreadsheet Template)

**Definition:** Defines the structure of a data entry form — which metrics, org nodes, and time periods appear for data capture. Required for querying input values.

**MCP Tools:**
- `templates_spreadsheetCaptures_list` — List capture templates
- `templates_spreadsheetCaptures_get` — Get template configuration

---

### Report Layout / Spreadsheet Report

**Capstone Term:** Report Template (Spreadsheet Template)

**Definition:** Defines which metrics appear in computed value queries. Acts as a filter lens — its discipline, metric type, and framework filters determine which metrics are visible.

**MCP Tools:**
- `templates_spreadsheetReports_list` — List report templates
- `templates_spreadsheetReports_get` — Get template configuration

> **Important:** New metrics automatically appear in a report template's output if they match its filter criteria. You don't need to explicitly add metrics to templates.

---

### Chart Configuration / Widget Setup

**Capstone Term:** Widget Template

**Definition:** Defines a widget's data source, visualization type, time range, and display properties.

**Widget Types:**

| ID | Name | Use Case |
|----|------|----------|
| 0 | InfoCard | Single KPI value with optional comparison |
| 1 | PieChart | Part-to-whole relationships, breakdowns |
| 2 | XYChart | Time series, trends, multi-metric comparison |

**MCP Tools:**
- `templates_widgets_list` — List widget templates

---

### Dashboard Layout / Page Configuration

**Capstone Term:** Dashboard Template

**Definition:** Defines the layout of a dashboard — sections, widget placement, and navigation structure.

**MCP Tools:**
- `templates_dashboards_list` — List dashboard templates

---

### Alternate Org View / Virtual Structure

**Capstone Term:** Org Node Template

**Definition:** A template that projects the org hierarchy differently for specific reporting needs without duplicating data.

**MCP Tools:** Not available in MCP v1 — use the CLI for org node template operations.

---

### Override / Exception / Location-Specific

**Capstone Term:** Metric Override

**Definition:** Per-location customization of metric properties (unit, formula, description) without changing the global definition.

**MCP Tools:** Not available in MCP v1 — use the CLI for metric override operations.

---

## Administration

### Organization / Company / Client

**Capstone Term:** Tenant

**Definition:** An isolated data environment for a single organization. All data is partitioned by tenant.

**MCP Tools:**
- `ListTenants` — List accessible tenants
- `SwitchTenant` — Change active tenant

**Resource:** `capstone://user/context` — Current tenant context.

---

### Account / Person / Login

**Capstone Term:** User

**Definition:** A system user with authentication credentials and assigned roles/permissions.

**MCP Tools:**
- `Whoami` — Current user identity and permissions

---

### Permission Set / Access Level

**Capstone Term:** Role

**Definition:** A named set of feature permissions. Users are assigned roles that control what actions they can perform.

**MCP Tools:** Not available in MCP v1 — use `Whoami` to see current user's roles.

---

### Data Permission / Access Matrix

**Capstone Term:** Data Permission

**Definition:** Controls which Org Nodes and Disciplines a user can view/edit. Combines with Roles for complete access control.

---

### External Connection / Integration

**Capstone Term:** Data Source

**Definition:** Configuration for ingesting data from external systems (MQTT brokers, SQL databases, REST APIs).

**MCP Tools:** Not available in MCP v1 — use the CLI for data source operations.

---

### Warehouse Export / Outbound Feed

**Capstone Term:** Data Export

**Definition:** Configuration for exporting Capstone data to external data warehouses or analytics platforms.

**MCP Tools:** Not available in MCP v1 — use the CLI for data export operations.

---

### Custom Field / Metadata / Tag

**Capstone Term:** Attribute Type

**Definition:** Custom attributes that extend entities with additional metadata. Each entity type (metric, discipline, framework, org node) has its own attribute types.

**MCP Tools:** Not available in MCP v1 — use the CLI for attribute type operations.

---

### Metric-to-Framework Link / Standard Mapping

**Capstone Term:** Metric Framework Node

**Definition:** Links metrics to reporting framework disclosure topics. Used to map internal metrics to external reporting requirements (GRI, SASB, etc.).

**MCP Tools:** Not available in MCP v1 — use the CLI for metric framework node operations.

---

## See Also

- [SKILL.md](../SKILL.md) — Quick vocabulary reference
- [Tools Reference](./tools.md) — Full tool documentation
- [Data Model](./data-model.md) — Enum tables and query patterns
- [CLI Glossary](../../cli/reference/glossary.md) — CLI-specific glossary with CLI commands
