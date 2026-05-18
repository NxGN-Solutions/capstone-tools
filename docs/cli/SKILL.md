# Capstone CLI Skill

> Teach Claude to use Capstone for data management and analysis.

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

| You Say | Capstone Calls It | CLI Domain | Notes |
|---------|-------------------|------------|-------|
| Location, site, facility, operation | Org Node | `masterdata org-nodes` | Hierarchical tree structure |
| KPI, measure, indicator, value | Metric | `model metrics` | Parent type for Input and Calculation |
| Manual data point, entered value | Input | `model inputs` | Captured via data entry forms |
| Formula-based value, derived metric | Calculation | `model calculations` | Computed from formulas |
| Narrative disclosure, text KPI, commentary | Narrative | `model narratives` | Text-based disclosure definition |
| Narrative category, text metadata | Narrative Attribute Type | `model narrative-attribute-types` | Custom attributes for narratives |
| Location-specific narrative rule | Narrative Override | `model narrative-overrides` | Per-org capture and validation flags |
| Category, type, topic | Discipline | `masterdata disciplines` | E.g., Environmental, Social, Governance |
| Reporting standard, disclosure framework | Framework | `masterdata frameworks` | GRI, SASB, CDP, TCFD, etc. |
| Unit, measurement, UoM | Unit of Measure | `masterdata units` | kWh, tonnes CO2e, litres, count |
| Time period, date range, interval | Time Period | `data time-periods` | Day, Week, Month, Quarter, Year |
| Raw data, submitted value | Input Value | `data input-values` | Actual captured data points |
| Text response, commentary value | Narrative Value | `data narrative-values` | Draft/submit/validate approved text |
| Aggregated value, result, computed | Computed Value | `reporting computed-values` | Calculated/rolled-up values |
| Dashboard chart, visualization | Widget | `reporting widgets` | Info Card, Pie Chart, XY Chart, Table |
| Chart configuration, widget setup | Widget Template | `templates widget-templates` | Defines widget data and display |
| Dashboard layout, page configuration | Dashboard Template | `templates dashboard-templates` | Organizes widgets into sections |
| Data entry form, capture sheet | Capture Template | `templates capture-templates` | Defines data entry structure |
| Report layout, spreadsheet template | Report Template | `templates report-templates` | Defines report structure |
| Alternate org view, virtual structure | Org Node Template | `templates org-node-templates` | Projects org structure differently |
| External connection, integration | Data Source | `masterdata data-sources` | MQTT, SQL, REST integrations |
| Warehouse export, outbound feed | Data Export | `masterdata data-exports` | Outbound data to external systems |
| Edit request, correction request | Change Request | `data change-requests` | Audit-tracked data modifications |
| Locked text correction request | Narrative Change Request | `data narrative-change-requests` | Governed edits for locked narrative values |
| Period freeze, lock, close period | Data Lock | `data data-lock` | Prevents changes to locked periods |
| Account, person, login | User | `security users` | System user with permissions |
| Permission set, access level | Role | `security roles` | Feature access control |
| Organization, company, client | Tenant | `auth tenants` | Isolated data environment |
| Language, translation | Localisation | `system languages` | Multi-language support |
| Override, exception, location-specific | Metric Override | `model *-overrides` | Per-location customization |
| Custom field, metadata, tag | Attribute Type | `* *-attribute-types` | Custom attributes for entities |
| Metric-to-framework link, standard mapping | Metric Framework Node | `model metric-framework-nodes` | Links metrics to reporting frameworks |

---

## Quick Start

### Platform Support

The CLI works on **Windows, macOS, and Linux**. For shell-specific syntax (environment variables, JSON input, file paths), see the **[Cross-Platform CLI Guide](./reference/platform-guide.md)**.

### CLI Invocation

| Context | Command |
|---------|---------|
| Pre-built binary (on PATH) | `cap <command>` |
| Pre-built binary (local, macOS/Linux) | `./cap <command>` |
| Pre-built binary (local, Windows) | `.\cap.exe <command>` |
| Source checkout | `dotnet run --no-build --project NxGN.Capstone.Cli -- <command>` |

### Verify Connection

```
cap auth whoami
```

Returns your username, tenant, and accessible tenants.

### Discover What's Available

**Model structure:**
```bash
cap model metrics list --json          # All metrics (inputs + calculations)
cap model inputs list --json           # Input metrics only
cap model calculations list --json     # Calculated metrics only
cap model narratives list --json       # Text narrative definitions
cap model narrative-attribute-types list --json
```

**Master data:**
```bash
cap masterdata org-nodes list --json      # Organizational hierarchy
cap masterdata disciplines list --json    # Metric categories
cap masterdata frameworks list --json     # Reporting standards
cap masterdata units list --json          # Units of measure
```

**Time periods:**
```bash
cap data time-periods list --data-interval month --json   # Monthly periods
cap data time-periods list --data-interval quarter --json # Quarterly periods
cap data time-periods list --data-interval year --json    # Annual periods
```

### Get Actual Data

```bash
# Input values (requires capture template + data-interval + periods)
cap data input-values list --template <id> --data-interval quarter --periods "Q1 FY 25" --json

# Computed values
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json

# Widget data
cap reporting widgets info-card <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json

cap reporting widgets table <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
```

---

## Recipe-First Approach

For standard tasks, use recipes. They encode best practices and reduce errors.

**See:** [recipes/README.md](./recipes/README.md) for the recipe index.

### When to Use Recipes vs Commands

| User Request | Approach |
|--------------|----------|
| "Analyze our emissions data..." | Recipe: talk-to-your-data |
| "Compare Site A vs Site B..." | Recipe: compare-operations |
| "What's trending with..." | Recipe: find-trends |
| "Create a metric for..." | Recipe: create-metric |
| "Enter this month's data..." | Recipe: enter-data |
| "List all metrics" | Direct: `cap model metrics list` |
| "Show org structure" | Direct: `cap masterdata org-nodes list` |
| "What periods are available?" | Direct: `cap data time-periods list --data-interval month` |

---

## Asking Questions

Always clarify before executing when user request is missing:

| Missing | Ask |
|---------|-----|
| Time period | "Which time period? (e.g., Q1 2025, last 6 months, FY 2024)" |
| Location/Org | "Which location or operation? (e.g., all sites, specific facility)" |
| Metric type | "What specifically do you want to measure? (e.g., electricity, water, safety incidents)" |
| Comparison basis | "Compare against what? (another site, last year, target)" |

---

## CLI Domains Overview

### status & config — Context & Configuration

**Purpose:** View current context and manage CLI settings.

| Command | Description |
|---------|-------------|
| `cap status` | Show current auth, tenant, and config context |
| `cap config get [key]` | View one or all configuration settings |
| `cap config set <key> <value>` | Set configuration value |
| `cap config unset <key>` | Reset setting to default |

### auth — Authentication & Tenant Management

**Purpose:** Login, logout, and tenant switching.

| Command | Description |
|---------|-------------|
| `cap auth login` | OAuth login or API key authentication |
| `cap auth logout` | Clear stored credentials |
| `cap auth whoami` | Show current user and tenant |
| `cap auth tenants` | List accessible tenants |
| `cap auth switch-tenant <id>` | Change active tenant |

### model — Metric Definitions

**Purpose:** Define what you want to measure (inputs, calculations, formulas).

| Command | Description |
|---------|-------------|
| `cap model metrics list` | List all metrics (inputs + calculations) |
| `cap model inputs list/get/create/save/delete/download-excel/upload-excel` | Manage input metrics |
| `cap model calculations list/get/create/save/delete/download-excel/upload-excel` | Manage calculations |
| `cap model narratives list/get/create/save/delete/delete-preview/delete-start/delete-status` | Manage text narrative definitions |
| `cap model narrative-attribute-types list/get/create/save/delete/download-excel/upload-excel` | Custom narrative attributes |
| `cap model narrative-overrides list/get/create/save/copy/download-excel/upload-excel` | Per-location narrative customization |
| `cap model metric-attribute-types list/get/create/save/delete/download-excel/upload-excel` | Custom metric attributes |
| `cap model metric-framework-nodes list/save/copy/download-excel/upload-excel` | Metric-to-framework associations |
| `cap model input-overrides *` | Per-location input customization |
| `cap model calculation-overrides *` | Per-location formula overrides |
| `cap model formula-validation validate` | Validate formula syntax |

### masterdata — Reference Data

**Purpose:** Core reference data: locations, categories, frameworks, units.

| Command | Description |
|---------|-------------|
| `cap masterdata org-nodes list/get/create/save/delete/download-excel/upload-excel` | Organizational hierarchy |
| `cap masterdata disciplines list/get/create/save/delete/download-excel/upload-excel` | Metric categories |
| `cap masterdata frameworks list/get/create/save/delete/download-excel/upload-excel` | Reporting standards |
| `cap masterdata units list/get/create/save/delete/download-excel/upload-excel` | Units of measure |
| `cap masterdata data-sources list/get/create/save/delete/download-excel/upload-excel` | External integrations |
| `cap masterdata discipline-attribute-types list/get/create/save/delete/download-excel/upload-excel` | Custom discipline attributes |
| `cap masterdata framework-attribute-types list/get/create/save/delete/download-excel/upload-excel` | Custom framework attributes |
| `cap masterdata org-node-attribute-types list/get/create/save/delete/download-excel/upload-excel` | Custom org node attributes |
| `cap masterdata data-exports list/get/create/save/delete` | Outbound data feeds (Planned) |

### templates — Configuration Templates

**Purpose:** Configure how data is captured, displayed, and reported.

| Command | Description |
|---------|-------------|
| `cap templates capture-templates *` | Data entry form configuration |
| `cap templates report-templates *` | Report spreadsheet templates |
| `cap templates widget-templates *` | Widget visualization config |
| `cap templates dashboard-templates *` | Dashboard layout config |
| `cap templates org-node-templates *` | Alternate org projections |
| `cap templates lookups <name>` | Template enum values |

### data — Values & Workflows

**Purpose:** Capture data, manage approvals, control periods.

| Command | Description |
|---------|-------------|
| `cap data time-periods list <type>` | Discover available periods |
| `cap data input-values list/get/create/save/validate` | Data entry operations |
| `cap data input-values download-excel/upload-excel` | Bulk data import/export |
| `cap data change-requests list/get/create/save/delete/validate` | Edit request workflow |
| `cap data narrative-values list/get/save/submit/validate/approved-read/history` | Narrative text capture, validation, approved reads, and audit history |
| `cap data narrative-change-requests list/get/lookup/create/save/delete/validate` | Governed narrative edits while data is locked |
| `cap data data-lock lock|unlock` | Lock/unlock data periods |

### reporting — Analysis & Output

**Purpose:** Retrieve computed values and dashboard data for analysis.

| Command | Description |
|---------|-------------|
| `cap reporting computed-values list` | Period-scoped computed values (requires `--template`, `--data-interval`) |
| `cap reporting computed-values download-excel` | Export computed values to Excel |
| `cap reporting dashboards get-data <id>` | All widget data for a dashboard |
| `cap reporting dashboards get-insights <id>` | AI-generated insights for a dashboard |
| `cap reporting widgets info-card <id>` | Info card widget data (requires `--org-nodes`, `--data-interval`) |
| `cap reporting widgets pie-chart <id>` | Pie chart widget data |
| `cap reporting widgets xy-chart <id>` | Line/bar/area chart data (JSON-only) |
| `cap reporting widgets table <id>` | Table widget render data |

**Note:** Most reporting commands require `--data-interval` and `--periods`. Exception: `widgets get-data` auto-detects the data interval from the widget template and uses `--org-node` (singular). Type-specific widget commands (`info-card`, `pie-chart`, `xy-chart`, `table`) require `--org-nodes` (plural). Use `cap data time-periods list --data-interval <interval>` to discover available period names.

### security — Users & Access (Planned)

**Purpose:** User management and access control.

| Command | Description | Status |
|---------|-------------|--------|
| `cap security users list/get/create/save/delete` | User management | ⏳ Planned |
| `cap security roles list` | Role definitions | ⏳ Planned |
| `cap security version get` | API version info | ⏳ Planned |

### system — Administration (Planned)

**Purpose:** System-level configuration: languages, translations, tenants.

| Command | Description | Status |
|---------|-------------|--------|
| `cap system languages list/get/create/save/delete` | Language config | ⏳ Planned |
| `cap system translations *` | Translation management | ⏳ Planned |
| `cap system tenants list/get/create/save/delete` | Tenant CRUD | ⏳ Planned |
| `cap system model-state get` | Model state info | ⏳ Planned |

---

## Command Patterns

### Standard CRUD Pattern

Most entities follow this pattern:

```bash
cap <domain> <entity> list [--json]           # List all
cap <domain> <entity> get <id> [--json]       # Get one by ID
cap <domain> <entity> create --file input.json  # Create new
cap <domain> <entity> save --file input.json    # Update existing
cap <domain> <entity> delete <id>             # Delete by ID
```

### Excel Import/Export Pattern

```bash
cap <domain> <entity> download-excel --output file.xlsx
cap <domain> <entity> upload-excel --file file.xlsx
```

### Capture Template Preflight Checklist

Use this before recommending or using a capture template in workflows.

1. Confirm period names for the intended interval:
```bash
cap data time-periods list --data-interval <day|week|month|quarter|year> --json
```
2. Pull a representative slice through the capture template:
```bash
cap data input-values list \
  --template <capture-template-id> \
  --data-interval <interval> \
  --periods "<period-name>" \
  --json > /tmp/capture-check.json
```
3. Validate capture surface area (row count):
```bash
jq '[.gridRows[] | select(.isDataRow==true)] | length' /tmp/capture-check.json
```
4. Validate metric scope (no unintended rows):
```bash
jq -r '[.gridRows[] | select(.isDataRow==true) | .name] | unique' /tmp/capture-check.json
```
5. If rows are noisy, tighten template filters in this order:
- constrain `Org Nodes`
- constrain `Disciplines`
- pin explicit `Metrics`
- keep `DataInterval` fixed for the use case

### Output Modes

| Flag | Output | Use Case |
|------|--------|----------|
| (none) | Table | Human-readable display |
| `--json` | JSON | AI/automation, parsing |

---

## Error Handling

### Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `AUTH_REQUIRED` | Not logged in | Run `cap auth login` |
| `NOT_FOUND` | Invalid ID | Verify ID with `list` command |
| `VALIDATION_ERROR` | Invalid input data | Check JSON structure matches DTO |
| `FORBIDDEN` | No permission | Check user roles and data permissions |

### Validation Errors

Validation errors return details about what failed:

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

## Feature-to-CLI Mapping

| Feature (docs/features/) | CLI Domain | Key Commands |
|--------------------------|------------|--------------|
| metrics/ | `model` | `metrics list/get`, `inputs *`, `calculations *` |
| org-nodes/ | `masterdata` | `org-nodes list/get/create/save/delete` |
| disciplines/ | `masterdata` | `disciplines list/get/create/save/delete` |
| frameworks/ | `masterdata` | `frameworks list/get/create/save/delete` |
| units-of-measure/ | `masterdata` | `units list/get/create/save/delete` |
| data-sources/ | `masterdata` | `data-sources list/get/create/save/delete` |
| data-exports/ | `masterdata` | `data-exports list/get/create/save/delete` |
| time-periods/ | `data` | `time-periods list` |
| data-management/ | `data` | `input-values *`, `change-requests *`, `data-lock` |
| reporting/ | `reporting` | `computed-values list`, `widgets get-data` |
| dashboards/ | `reporting` | `widgets info-card/pie-chart/xy-chart/table` |
| widget-templates/ | `templates` | `widget-templates list/get/create/save/delete` |
| dashboard-templates/ | `templates` | `dashboard-templates list/get/create/save/delete` |
| spreadsheet-templates/ | `templates` | `capture-templates *`, `report-templates *` |
| org-node-templates/ | `templates` | `org-node-templates list/get/create/save/delete` |
| users/ | `security` | `users list/get/create/save/delete`, `roles list` |
| tenants/ | `auth` + `system` | `auth tenants`, `system tenants *` |
| localisation/ | `system` | `languages *`, `translations *` |

---

## CLI Implementation Status

| Domain | Status | Features |
|--------|--------|----------|
| auth, model, masterdata, templates, data | ✅ Complete | Full CRUD + Excel for all core entities |
| Excel commands | ✅ Complete | download-excel, upload-excel for all entities |
| Overrides | ✅ Complete | input-overrides, calculation-overrides, formula-validation |
| Attribute types | ✅ Complete | metric, discipline, framework, org-node attribute types + metric-framework-nodes |
| Reporting | ✅ Complete | computed-values, widgets, dashboards |
| Security | Planned | users, roles, version |
| System | Planned | languages, translations, tenants CRUD |

**Impact:** Full CRUD + Excel support for all model, masterdata, and template entities. Analysis recipes work with reporting complete.

---

## See Also

- [CLI Design](../designs/cli/00-overview.md) — Full command reference and architecture
- [Recipes](./recipes/README.md) — Step-by-step workflows
- [Commands Reference](./reference/commands.md) — Quick command lookup
- [Template Selection Guide](./reference/template-selection.md) — Decide capture vs report vs widget vs dashboard workflows
- [Glossary](./reference/glossary.md) — Detailed term mapping
- [Output Formats](./reference/output-formats.md) — Table vs JSON output examples
- [Feature Documentation](../features/README.md) — Product capabilities
