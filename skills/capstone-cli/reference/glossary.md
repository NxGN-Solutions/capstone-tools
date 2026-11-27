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
cap data input-values validate --ids <id1>,<id2> --result approve --json
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

### Completion / Progress / Outstanding Data

**Capstone Term:** Capture or Validation completion (report template `measure`)

**Definition:** A report template with `measure` 1 (Capture completion) or 2 (Validation completion) counts values instead of showing them:

- **Expected:** one value per input or narrative, site and period where the site is on the physical structure, the item is not excluded there, capture is required there, the period is of the item's own interval and the tenant has that interval enabled, and the period starts in or after the tenant's first year. Calculations are never expected.
- **Captured:** an input holds a number (zero counts; a comment alone does not), or a narrative has text.
- **Validated:** captured and approved, or captured where no validation applies (the tenant has no validation levels, or validation is not required at that site). A rejected value is never validated.
- **Not due:** a period whose last day is not before today's UTC date. It is shown but not counted.

Counts cover only the data the caller may view; nothing is stored. Cells are coloured by `completionBands`, bands over the share done like a metric's bands; the defaults are red below 40%, amber from 40% and green from 90%.

**Who is next:** with `showWhoIsNext` on a completion template (or a progress Table widget), each outstanding value names who acts next, by the reminder rules: the capturers of a value not captured, the validators of the level it awaits, or whoever saved a rejected value. Tenant-wide holders are named only where nobody holds a direct grant; NxGN admins never are. Rows carry `whoIsNext`, cells `next`, and the response's `users` and `validationLevels` name the ids.

**CLI:**
```bash
cap reporting computed-values list --template <completion-template-id> --data-interval month --periods "Jan 2026" --json
```



---

### Chart / Visualization

**Capstone Term:** Widget

**Definition:** A visual representation of data (Info Card, Pie Chart, XY Chart, Table, TextBlock, Gauge). Widgets display computed values based on their template configuration.

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

**Definition:** Defines the structure of a data entry form—which metrics, org nodes, and time periods appear for data capture. `showRowsWithValuesFilter` lets viewers switch between all rows and rows with values (`showOnlyRowsWithValues` is the default; it always applies when the flag is false). `showPendingFilter` offers a "Show pending only" toggle on its capture and validation grids, and `showOnlyPending` opens it switched on. `showChangeRequests` marks cells whose value has a pending change request, and `allowChangeRequestsToggle` gives viewers a Change Requests switch (both default false). `showDataGroupingFilter` lets viewers regroup the rows by the levels in `allowedDataGroupingLevels`, in any order, and to move levels between rows and columns. `columnDataGroupingLevels` lists the grouping levels placed on columns (for example a column block per site, with discipline rows); empty means every level is on rows.

**CLI:**
```bash
cap templates capture-templates list --json
cap templates capture-templates get <id> --json
```

Template `list` commands return every template by default (up to 1000); `--limit` and `--offset` page on the server, and `--search` (widget and dashboard templates) filters names on the server.


---

### Reminder / Alert / Scheduled Email

**Capstone Term:** Notification Rule

**Definition:** Sends a notification template when an event happens (Event Driven) or on a schedule (Scheduled). Each template is **Written for** one event or its type's digest. An Event Driven rule needs a template written for an event, and sends it when that event happens; the template names the event in its `eventType` (for example Input Value Captured or Data Lock Applied). A Scheduled rule needs a digest (null `eventType`): Pending capture digest, Pending validation digest, Pending change requests digest or Data feed changes digest. While a rule uses a template, what the template is written for cannot change. The rule picker offers only templates marked **Available to rules** (`isActive`); turning that off does not stop rules that already use the template. A schedule is a cron expression or period-end offsets: signed days from the end of each month, fiscal quarter or fiscal year, at a fire time in the rule's time zone. Each scheduled fire sends a recipient one email listing the capture templates to open for their pending work.

A rule whose template has the notification type **Data Feed Changes** must be Scheduled. Each send lists the input values and narratives a data feed changed or cleared in the rule's scope since the previous send. It goes to the Data Owners in scope (see Data Permission) and, when `notifyReplacedValueSavers` is true (the default), once to the person whose value the feed replaced. `* * * * *` sends every minute. A schedule with more than 120 days between sends is refused, because changes are kept for 120 days.

**CLI:**
```bash
cap notifications rules list --json
cap notifications rules get <id> --json
cap notifications rules runs <id> --json
cap notifications rules recipients <id> --json
cap notifications delivery-log list --rule <id> --json
```


---

### Report Layout / Spreadsheet Report

**Capstone Term:** Report Template (Spreadsheet Template)

**Definition:** Defines the structure of a report—which metrics and computed values to display in a spreadsheet format. `showRowsWithValuesFilter` lets viewers switch between all rows and rows with values; when false, `showOnlyRowsWithValues` always applies. `showChangeRequests` marks cells with a pending change request, and `allowChangeRequestsToggle` gives viewers a Change Requests switch (both default false; completion reports refuse them). `showDataGroupingFilter` lets viewers regroup the rows by the levels in `allowedDataGroupingLevels`, in any order, and to move levels between rows and columns. `columnDataGroupingLevels` lists the grouping levels placed on columns (for example a column block per site, with discipline rows); empty means every level is on rows.

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
| 6 | TextBlock | Metric- and narrative-aware dashboard text slots |
| 7 | Gauge | One metric at one site on a dial of its ranges |

**Key Properties:**
- `widgetType` - Visualization type (InfoCard, PieChart, XYChart, Table, TextBlock, Gauge)
- `discipline` - Categorization for organization and permissions
- `dataItems` - Array of metrics to display for metric-set widgets; TextBlock does not support data items and discovers metrics and narratives from text tokens (`[metric-guid]`, `{narrative-guid}`)
- `dataRangeMode` - Static (a fixed window of period offsets), Dynamic (the viewer's periods), SelectedYear (the viewer's year up to their period; XY charts and tables) or SelectedYearToLatestData (the same, ending at the latest period with data; XY charts)
- `dataInterval` - Monthly, Quarterly, Annual, etc.
- `pieChartWidgetTemplateType` / `centerMode` / `sliceLabelMode` - PieChart-only display options for pie vs donut, donut center content (None, Static Text, Metric Value, Total, Percentage), and labels rendered on slices (label, value, percent)
- `legendValueMode` - PieChart-only stored enum behind the dashboard editor's **Legend values** dropdown: Hidden, Value, Percent, Value and Percent
- `pieShape` - PieChart-only: Full Circle (default) or Half Circle, the top half
- `otherSliceMode` / `otherSliceThreshold` - PieChart-only: group small slices into one Other slice, by keeping the largest N (Top N, 2 to 20) or grouping slices below N% of the total (Below Percent, 1 to 25)
- `dataGrouping`, `orgNodeRowSelectionMode`, `showOnlyRowsWithValues` - Table-only row grouping and table-wide display controls (at a leaf org node, Children and Descendants render the node itself)
- `showChangeRequests` / `allowChangeRequestsToggle` - Table-only, both default false: mark value cells with a pending change request, and let the dashboard's Change Requests switch hide those marks (the switch never shows marks a widget hides)
- `metricDisciplineFilters`, `metricFrameworkFilters`, and metric/discipline/framework attribute filters - Table dynamic Metric Scope controls (a table ignores `metricTypeFilters`; `includedDataTypes` decides its metric types)
- `showMetricValue`, `metricPropertyColumns`, `metricAttributeTypeIds` - Table current custom-column controls. `metricPropertyColumns` is a `MetricProperty` EnumDTO list (not `metricColumns`, which means metrics rendered as columns)
- `narrativeSelectionMode`, `narrativeScopeConfigured`, `narratives`, and narrative scope filters - Table dynamic/static narrative row controls
- `styleConfiguration` - Table semantic-HTML style contract for slots, row/column overrides, conditional formatting, and categorical color tags
- `subtitle`, TextBlock `styleConfiguration` - TextBlock-only secondary heading and bounded text-slot style contract
- `gaugeShape`, `gaugeStartAtZero`, `gaugeMinimum`, `gaugeMaximum`, `gaugeMajorStep`, `gaugeMinorTicksPerStep`, `gaugeLogarithmic`, `gaugeShowRangeName`, `gaugeShowDistance` - Gauge-only: a Half Dial, Arc Dial or Solid dial whose scale fields mean what an XY chart's value-axis fields mean; a gauge shows one metric (one `dataItems` row) at one site, with the range it reads in and the distance to the nearest range edge under the value. Its style's `rangeColours` (`strong` or `asSet`) and `rangeRoomPercent` (0 to 50, default 10) set how the metric's ranges are drawn and the room past an open outer range

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

`allowChangeRequestsToggle` (default false) shows viewers a Change Requests switch beside the formatting toggle; table widgets that follow the dashboard hide their change request marks while it is off.

**CLI:**
```bash
cap templates dashboard-templates list --json
cap templates dashboard-templates get <id> --json
```


---

### Alternate Org View / Virtual Structure

**Capstone Term:** Org Node Template

**Definition:** A template that projects the org hierarchy differently for specific reporting needs without duplicating data.

**Roll Down on top-level groups:** A top-level group that is not anchored on a canonical org node of the same name (for example Coal in a template grouped only by Commodity Type) has no parent, so Roll Down inputs (targets, factors, limits) are blank on it, and so are the ratios and bands that use them. Additive metrics still add up. To give the groups those values, add an attribute with one value named after the canonical root (for example *Reporting Group* = `Mining Group`) set on the root, group by it first and by the pivot attribute second, and use the `HierarchyBelowGroupings` visibility mode: the groups then sit below the anchored root and inherit from it.

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
cap model input-overrides list --org-nodes <id> --json
cap model calculation-overrides list --org-nodes <id> --json
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
cap security users get <id-or-email> --json
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

The kinds are View, Edit, Data Validation, Change Request Approval and **Data Owner**. Data Owner grants no access: it marks who receives Data Feed Changes notifications, and the user still needs View or higher. The scope permission **All Data (Owner)** (`nxgn.capstone.data.owner.unrestricted`) marks a user as Data Owner everywhere; Data Scope Unrestricted does not include it. API keys cannot hold either. In the user workbook (`cap security users download-excel` / `upload-excel`), marks go in the **Data Owner Permissions** column; a workbook without that column keeps existing marks.


---

### External Connection / Integration

**Capstone Term:** Data Source

**Definition:** Configuration for ingesting data from external systems (SQL databases, accounting and payroll systems, REST pushes). MQTT is not an import source; Capstone only publishes to MQTT as a data export target.

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
