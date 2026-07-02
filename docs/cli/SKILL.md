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
| Dashboard chart, visualization | Widget | `reporting widgets` | Info Card, Pie Chart, XY Chart, TextBlock, Table |
| Chart configuration, widget setup | Widget Template | `templates widget-templates` | Defines widget data and display |
| Dashboard layout, page configuration | Dashboard Template | `templates dashboard-templates` | Organizes widgets into sections |
| Data entry form, capture sheet | Capture Template | `templates capture-templates` | Defines data entry structure |
| Report layout, spreadsheet template | Report Template | `templates report-templates` | Defines report structure |
| Alternate org view, virtual structure | Org Node Template | `templates org-node-templates` | Projects org structure differently |
| External connection, integration | Data Source | `masterdata data-sources` | MQTT, SQL, REST integrations |
| Edit request, correction request | Change Request | `data change-requests` | Audit-tracked data modifications |
| Locked text correction request | Change Request | `data change-requests` | Governed edits for locked input or narrative values |
| Period freeze, lock, close period | Data Lock | `data data-lock` | Prevents changes to locked periods |
| Account, person, login | User | `security users` | System user with permissions |
| Organization, company, client | Tenant | `auth tenants` | Isolated data environment |
| Language, translation | Localisation | `auth languages` | Multi-language support |
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

### Connect First (Auth-First On-Ramp)

**Before any discovery or data command, establish a connection.** Empty results
almost always trace back to auth/tenant, not missing data — so do this first:

```bash
cap config set api-url <URL>   # 1. Point at the Capstone server (ask the user for the URL)
cap auth login                 # 2. Log in via OAuth (opens browser)
cap auth whoami                # 3. Confirm identity, current tenant, and accessible tenants
```

If anything looks wrong, run `cap status` and `cap auth doctor` (see
[Error Handling](#error-handling)).

### Verify Connection

```
cap auth whoami
```

Returns your username, tenant, and accessible tenants.

### Automation Error Contract

Use `--json` for machine-readable workflows and branch on exit codes, not
free-text stderr. Run `cap schema --json` for the live contract.

| Exit | Meaning |
|------|---------|
| `0` | Success |
| `1` | Local CLI failure or user cancellation |
| `2` | Validation/client-actionable failure, including not found and non-retriable API 4xx |
| `3` | Authentication or tenant failure |
| `4` | Server/API/network/rate-limit/malformed-response failure |
| `5` | Timeout |
| `6` | Partial result |

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

**Canonical option names:**

| Concept | Use |
|---------|-----|
| Reporting period interval | `--data-interval <day|week|month|quarter|year>` |
| One org node | `--org-node <id>` |
| One or more org nodes | `--org-nodes <id>[,<id>]` |
| Discipline tree filters | `--discipline-nodes <id>[,<id>]` |
| Framework tree filters | `--framework-nodes <id>[,<id>]` |
| Capture/report template | `--template <id>` |

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
| "Style/customize an Info Card..." | Recipe: configure-info-card-styles |
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

Use this section for routing. For full implemented command inventory and
status, run `cap schema --json` or read [reference/commands.md](./reference/commands.md).

| Domain | Use For | High-Value Commands |
|--------|---------|---------------------|
| `status`, `config`, `auth` | Context, login, tenants, API keys | `status`, `auth whoami`, `auth tenants`, `auth switch-tenant`, `auth apikey *` |
| `model` | Metric, input, calculation, narrative definitions | `metrics list/get`, `inputs *`, `calculations *`, `narratives *`, `*-overrides *`, `formula-validation validate` |
| `masterdata` | Org nodes, disciplines, frameworks, units, data sources | `org-nodes *`, `disciplines *`, `frameworks *`, `units *`, `data-sources *` |
| `templates` | Capture/report/widget/dashboard configuration | `capture-templates *`, `report-templates *`, `widget-templates *`, `dashboard-templates *`, `lookups get` |
| `data` | Captured values, narratives, change requests, locks | `time-periods list`, `input-values *`, `narrative-values *`, `change-requests *`, `data-lock lock|unlock` |
| `reporting` | Computed values and dashboard/widget output | `computed-values list/query/audit`, `widgets info-card/pie-chart/xy-chart/table`, `dashboards get-data/get-insights` |
| `security`, `system` | User Excel import/export and tenant administration | `security users download-excel/upload-excel`, `system tenants *`, `system tenants fiscal-config update` |
| `notifications` | Notification template administration | `notifications templates create/get/list/save/delete` |

Most reporting commands require `--data-interval` and `--periods`. Type-specific
widget commands use `--org-nodes` (plural). `widgets get-data` is the legacy CSV
compatibility path with singular `--org-node`; prefer `computed-values` or typed
widget commands for automation-safe checks.

For widget styling work, start with
`cap templates widget-templates schema --widget-type <info|pie|xy|table> --json`
and `cap templates widget-templates sample --widget-type <type> --json`. Widget
samples put the editable body under `.payload.widgetTemplate`; `get` responses
put it under `.widgetTemplate`; `save` returns an acknowledgement object, not the
full updated template. Pie/Donut enum values for center, legend-value, and
slice-label modes are in `schema.fieldLookups`, and
`cap reporting widgets pie-chart ... --json` returns a top-level render contract
with `styleConfiguration`, `center`, `legend`, `labelDisplay`, and
`dataItems[]`.

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
cap <domain> <entity> upload-excel file.xlsx
```

### Import And Upsert Identity

Use the same identity contract for JSON batch/upsert work and Excel upload
work:

| Entity shape | Match key |
|--------------|-----------|
| Flat entities | Exact full name only after trimming leading/trailing whitespace; case-sensitive |
| Tree entities | Exact full path only after trimming leading/trailing whitespace; case-sensitive |

Names are unique for flat entities; paths are unique for tree entities. Do not
use fuzzy matching, aliases, partial names, or tree-node short names when
planning imports. Read `cap schema --json` and use the `upsertIdentity` section
when an agent needs to confirm the current contract. `upsertIdentity.surfaces`
describes the two parallel ingestion paths: `json-batch-upsert` for rerunnable
JSON payloads and `excel-upload` for spreadsheet workflows that need row/cell
diagnostics.

For masterdata, model definition, and widget/dashboard template JSON imports,
prefer explicit upsert for rerunnable builds:

```bash
cap model inputs import-json --file inputs.json --upsert --json
cap model calculations validate-batch --file calculations.json --upsert --json
cap model calculations import-json --file calculations.json --upsert --json
cap masterdata org-nodes import-json --file org-nodes.json --upsert --json
cap masterdata disciplines import-json --file disciplines.json --upsert --json
cap masterdata frameworks import-json --file frameworks.json --upsert --json
cap masterdata units import-json --file units.json --upsert --json
cap templates widget-templates import-json --file widget-templates.json --upsert --json
cap templates dashboard-templates import-json --file dashboard-templates.json --upsert --json
```

Use `--dry-run --upsert` to preview `would-create` and `would-update` actions
before mutating a tenant.

Use snapshot export when you need a reviewable or versionable copy of the current
tenant's model-building surface:

```bash
cap system tenants snapshot --output snapshot --json
```

The snapshot command hydrates full DTOs for units, org nodes, disciplines,
frameworks, inputs, calculations, widget templates, and dashboard templates. It
writes JSON arrays that can be re-imported with the matching `import-json
--upsert` commands listed in `manifest.json`. Snapshot is read-only against the
API; restore is the explicit import/upsert path after review.

For restore planning, ask the CLI for the reviewed workflow instead of looking
for an automatic restore command:

```bash
cap workflows show restore-from-snapshot --json
```

Follow its dry-run first sequence, validate calculation batches before applying
them, and verify the restored model with recalculation wait plus computed-value
and dashboard-template audits.

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

### Auth & Tenant — Check This FIRST

**All commands return empty / no rows → verify auth & tenant before anything
else.** Run `cap status` and `cap auth doctor`, then confirm identity and tenant
with `cap auth whoami`. An empty result with exit code `0` can mean you are in
the wrong tenant, that an ambient `CAPSTONE_API_KEY` is shadowing your OAuth
login (it resolves to the empty bootstrap tenant), or that you lack
permissions — not just that data is missing.

```bash
cap status          # auth status, tenant, API endpoint at a glance
cap auth doctor     # diagnose authentication state + print recovery commands
cap auth whoami     # confirm username, current tenant, accessible tenants
```

`cap auth doctor` is the auth-recovery entry point; follow the recovery
commands it prints (typically `cap auth login` and/or `cap auth switch-tenant`).

### Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| Empty results / no rows (exit `0`) | Wrong tenant, ambient `CAPSTONE_API_KEY` shadowing OAuth, or no permission | Run `cap status`, `cap auth doctor`, `cap auth whoami` **before** assuming missing data |
| `AUTH_REQUIRED` | Not logged in | Run `cap auth login` (diagnose with `cap auth doctor`) |
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

## See Also

- [Recipes](./recipes/README.md) - Step-by-step workflows
- [Commands Reference](./reference/commands.md) - Quick command lookup
- [Template Selection Guide](./reference/template-selection.md) - Decide capture vs report vs widget vs dashboard workflows
- [Widget Time Aggregation](./reference/widget-time-aggregation.md) - Choose widget-level and data-item time aggregation methods
- [Glossary](./reference/glossary.md) - Detailed term mapping
- [Output Formats](./reference/output-formats.md) - Table vs JSON output examples
