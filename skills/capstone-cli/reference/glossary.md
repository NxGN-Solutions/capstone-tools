# Glossary

> Standard terms → Capstone terminology mapping with CLI examples.

---

## Data Modeling

### KPI / Metric / Measure / Indicator

**Capstone Term:** Metric

**Definition:** A quantifiable measure that tracks performance. In Capstone, Metric is the parent type for both Input (manual data) and Calculation (formula-derived).

**CLI:**
```bash
cap model metrics list --json          # All metrics
cap model inputs list --json           # Manual entry metrics
cap model calculations list --json     # Formula-based metrics
```


---

### Location / Site / Facility / Operation

**Capstone Term:** Org Node

**Definition:** A node in the organizational hierarchy representing a physical location, business unit, or reporting entity. Org Nodes form a tree structure for data aggregation.

**CLI:**
```bash
cap masterdata org-nodes list --json
cap masterdata org-nodes get <id> --json
```


---

### Category / Topic / Type

**Capstone Term:** Discipline

**Definition:** A hierarchical categorization of metrics (e.g., Environmental, Social, Governance). Disciplines form a tree structure.

**CLI:**
```bash
cap masterdata disciplines list --json
cap masterdata disciplines get <id> --json
```


---

### Reporting Standard / Disclosure Framework

**Capstone Term:** Framework

**Definition:** An external reporting standard that metrics can be mapped to (e.g., GRI, SASB, CDP, TCFD). Frameworks have nodes representing disclosure topics.

**CLI:**
```bash
cap masterdata frameworks list --json
cap masterdata frameworks get <id> --json
```


---

### Unit / Measurement / UoM

**Capstone Term:** Unit of Measure

**Definition:** The measurement unit for metric values (e.g., kWh, tonnes CO2e, litres, count). Each unit has an optional **symbol** and a **symbol position** (Prefix or Suffix) that controls how values display. Currency symbols use Prefix (`$100`), all other units use Suffix (`100 kg`).

**CLI:**
```bash
cap masterdata units list --json
cap masterdata units get <id> --json
cap masterdata units save --json       # Update symbol position, etc.
```


---

### Manual Data / Entered Value

**Capstone Term:** Input (Metric)

**Definition:** A metric that captures manually entered or automatically ingested data points. Inputs are the source data for calculations.

**CLI:**
```bash
cap model inputs list --json
cap model inputs create --file input.json
```


---

### Formula / Derived Metric / Calculated Value

**Capstone Term:** Calculation (Metric)

**Definition:** A metric whose values are computed from a formula referencing other metrics. Formulas use Excel-like syntax.

**CLI:**
```bash
cap model calculations list --json
cap model calculations create --file calc.json
```


---

## Data Capture

### Raw Value / Submitted Data / Data Point

**Capstone Term:** Input Value

**Definition:** An actual data entry for an Input metric at a specific Org Node and Time Period. Input Values are stored with validation status and audit trail.

**Business Key:** `metric` + `orgNode` + `timePeriodType` + `startDate`. This combination uniquely identifies an input value. The save endpoint uses business-key upsert — zero IDs resolve to existing records automatically, making saves idempotent.

**CLI:**
```bash
cap data input-values list --template <id> --data-interval week --periods "(W5) Jan 26 2026" --json
cap data input-values save --file values.json
```

> **Note:** The `list` command requires both `--template` and `--data-interval`. Use `--periods` to specify which periods to show (comma-separated period names from `time-periods list`).


---

### Time Period / Date Range / Interval

**Capstone Term:** Time Period

**Definition:** A time interval for data capture (Day, Week, Month, Quarter, Year). Each metric has a defined data interval.

**CLI:**
```bash
cap data time-periods list --data-interval month --json
cap data time-periods list --data-interval quarter --json
cap data time-periods list --data-interval year --json
```


---

### Approval / Validation / Sign-off

**Capstone Term:** Validation Level

**Definition:** Multi-level approval workflow for data quality assurance. Each Input Value progresses through validation levels before being "approved."

**CLI:**
```bash
cap data input-values validate <id> --result approve --json
cap data input-values validate <id> --result reject --comments "Reason for rejection" --json
```


---

### Edit Request / Correction Request

**Capstone Term:** Change Request

**Definition:** An audit-tracked request to modify locked or validated data. Change Requests require approval and maintain complete history.

**CLI:**
```bash
cap data change-requests list --json
cap data change-requests create --file request.json
cap data change-requests validate <id> --result approve
```


---

### Period Freeze / Lock / Close Period

**Capstone Term:** Data Lock

**Definition:** Prevents modifications to data within a specified time period and org scope. Used for audit assurance.

**CLI:**
```bash
cap data data-lock lock --data-interval year --periods "FY 2024" --org-nodes <id> --description "Close FY 2024"
cap data data-lock unlock --data-interval year --periods "FY 2024" --org-nodes <id> --description "Reopen FY 2024"
```


---

## Reporting & Analytics

### Aggregated Value / Result / Computed

**Capstone Term:** Computed Value

**Definition:** The calculated or aggregated value of a metric at a specific Org Node and Time Period. Includes both formula calculations and hierarchical rollups.

**CLI:**
```bash
# Query computed values through a report template's filter lens
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval month \
  --periods "Jan 2026" \
  --json

# Download as Excel
cap reporting computed-values download-excel \
  --template <report-template-id> \
  --data-interval month \
  --periods "Jan 2026" \
  -o output.xlsx
```



---

### Chart / Visualization

**Capstone Term:** Widget

**Definition:** A visual representation of data (Info Card, Pie Chart, XY Chart, Table, TextBlock). Widgets display computed values based on their template configuration.

**CLI:**
```bash
# Type-specific (typed JSON for rendering)
cap reporting widgets info-card <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" --json
cap reporting widgets pie-chart <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" --json
cap reporting widgets xy-chart <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" --json
cap reporting widgets table <widget-template-id> --org-nodes <id> --data-interval month --periods "Jan 25" --json

# Type-agnostic (CSV for AI analysis — auto-detects data interval)
cap reporting widgets get-data <widget-template-id> --org-node <id> --periods "FY 24, FY 25" --json
```


---

### Dashboard / Report Page

**Capstone Term:** Dashboard

**Definition:** A collection of widgets arranged in sections for data visualization. Dashboards are configured via Dashboard Templates.

**CLI:**
```bash
cap reporting dashboards get-data <dashboard-template-id> --org-node <id> --data-interval month --periods "Jan 25" --json
cap reporting dashboards get-insights <dashboard-template-id> <layout-node-id> --org-node <id> --data-interval month --periods "Jan 25" --json
```


---

### Rollup / Aggregation

**Capstone Term:** Aggregation Method

**Definition:** How metric values combine across time or org hierarchy (Sum, Average, Max, Min, Last Value, Count).

**CLI:**
```bash
cap model inputs get <id> --json   # Shows aggregation settings
```


---

## Configuration

### Data Entry Form / Capture Sheet

**Capstone Term:** Capture Template (Spreadsheet Template)

**Definition:** Defines the structure of a data entry form—which metrics, org nodes, and time periods appear for data capture.

**CLI:**
```bash
cap templates capture-templates list --json
cap templates capture-templates get <id> --json
```


---

### Report Layout / Spreadsheet Report

**Capstone Term:** Report Template (Spreadsheet Template)

**Definition:** Defines the structure of a report—which metrics and computed values to display in a spreadsheet format.

**CLI:**
```bash
cap templates report-templates list --json
cap templates report-templates get <id> --json
```


---

### Chart Configuration / Widget Setup

**Capstone Term:** Widget Template

**Definition:** Defines a widget's data source, visualization type, time range, and display properties.

**Widget Types:**
| ID | Name | Best For |
|----|------|----------|
| 0 | InfoCard | Single KPI value with optional comparison |
| 1 | PieChart | Part-to-whole relationships, breakdowns |
| 2 | XYChart | Time series, trends, multi-metric comparison |
| 4 | Table | Metric-backed dashboard table/grid |
| 6 | TextBlock | Metric-aware dashboard text slots |

**Key Properties:**
- `widgetType` - Visualization type (InfoCard, PieChart, XYChart, Table, TextBlock)
- `discipline` - Categorization for organization and permissions
- `dataItems` - Array of metrics to display for metric-set widgets; TextBlock does not support data items and discovers metrics from text tokens
- `dataRangeMode` - Static (fixed) or Dynamic (relative) date range
- `dataInterval` - Monthly, Quarterly, Annual, etc.
- `pieChartWidgetTemplateType` / `centerMode` / `sliceLabelMode` - PieChart-only display options for pie vs donut, donut center content, and labels rendered on slices
- `legendValueMode` - PieChart-only stored enum behind the dashboard editor's **Show Legend Value** checkbox
- `dataGrouping`, `orgNodeRowSelectionMode`, `showOnlyRowsWithValues` - Table-only row grouping and table-wide display controls
- `metricTypeFilters`, `metricDisciplineFilters`, `metricFrameworkFilters`, and metric/discipline/framework attribute filters - Table dynamic Metric Scope controls
- `showMetricValue`, `showUnitOfMeasure`, `metricAttributeTypeIds` - Table current custom-column controls
- `narrativeSelectionMode`, `narrativeScopeConfigured`, `narratives`, and narrative scope filters - Table dynamic/static narrative row controls
- `styleConfiguration` - Table semantic-HTML style contract for slots, row/column overrides, conditional formatting, and categorical color tags
- `subtitle`, TextBlock `styleConfiguration` - TextBlock-only secondary heading and bounded text-slot style contract

**CLI:**
```bash
cap templates widget-templates list --json
cap templates widget-templates get <id> --json
cap templates widget-templates create --file widget.json --json
cap reporting widgets pie-chart <id> --org-nodes <org-node-id> --data-interval month --periods "Jan 25" --json
cap reporting widgets table <id> --org-nodes <org-node-id> --data-interval quarter --periods "Q1 FY 25" --json
```

> **Note:** The create/save commands auto-wrap JSON - you can provide just the inner object without `{"widgetTemplate": {...}}`.


---

### Dashboard Layout / Page Configuration

**Capstone Term:** Dashboard Template

**Definition:** Defines the layout of a dashboard—sections, widget placement, and navigation structure.

**CLI:**
```bash
cap templates dashboard-templates list --json
cap templates dashboard-templates get <id> --json
```


---

### Alternate Org View / Virtual Structure

**Capstone Term:** Org Node Template

**Definition:** A template that projects the org hierarchy differently for specific reporting needs without duplicating data.

**CLI:**
```bash
cap templates org-node-templates list --json
cap templates org-node-templates get <id> --json
```


---

### Override / Exception / Location-Specific

**Capstone Term:** Metric Override

**Definition:** Per-location customization of metric properties (unit, formula, description) without changing the global definition.

**CLI:**
```bash
cap model input-overrides list --org-node <id> --json
cap model calculation-overrides list --org-node <id> --json
```


---

### Custom Field / Metadata / Tag

**Capstone Term:** Attribute Type

**Definition:** A user-defined custom attribute that can be attached to entities (metrics, disciplines, frameworks, org nodes). Attribute Types extend the data model without schema changes.

**CLI:**
```bash
cap model metric-attribute-types list --json
cap masterdata discipline-attribute-types list --json
cap masterdata framework-attribute-types list --json
cap masterdata org-node-attribute-types list --json
```

---

## Administration

### Organization / Company / Client

**Capstone Term:** Tenant

**Definition:** An isolated data environment for a single organization. All data is partitioned by tenant.

**CLI:**
```bash
cap auth tenants --json              # List accessible tenants
cap auth switch-tenant <id>          # Change active tenant
```


---

### Account / Person / Login

**Capstone Term:** User

**Definition:** A system user with authentication credentials and assigned roles/permissions.

**CLI:**
```bash
cap security users download-excel -o users.xlsx
cap security users upload-excel users.xlsx --json
```


---

### Permission Set / Access Level

**Capstone Term:** Role

**Definition:** A named set of feature permissions. Users are assigned roles that control what actions they can perform.

**CLI:** (Planned)
```bash
cap security roles list --json
```


---

### Data Permission / Access Matrix

**Capstone Term:** Data Permission

**Definition:** Controls which Org Nodes and Disciplines a user can view/edit. Combines with Roles for complete access control.


---

### External Connection / Integration

**Capstone Term:** Data Source

**Definition:** Configuration for ingesting data from external systems (MQTT brokers, SQL databases, REST APIs).

**CLI:**
```bash
cap masterdata data-sources list --json
cap masterdata data-sources get <id> --json
```


---

### Warehouse Export / Outbound Feed

**Capstone Term:** Data Export

**Definition:** Configuration for exporting Capstone data to external data warehouses or analytics platforms.

**CLI:** (Planned)
```bash
cap masterdata data-exports list --json
cap masterdata data-exports get <id> --json
```


---

## See Also

- [SKILL.md](../SKILL.md) — Quick vocabulary reference
- [Commands Reference](./commands.md) — CLI command lookup
- [Output Formats](./output-formats.md) — Table vs JSON output examples
- [Recipes](../recipes/README.md) — Step-by-step workflows
