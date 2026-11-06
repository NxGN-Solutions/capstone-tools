# Recipe: Back Up and Restore a Tenant

> Download a tenant's configuration to Excel and upload it again, into the same tenant or another one.

## When to Use

- "Back up this tenant"
- "Copy the model and templates to another tenant"
- "Restore the configuration from last week's export"

The CLI uses the same Excel download and upload as the web app. There is no separate CLI export
format. Excel files reference other entities by name (metric names, org-node paths, narrative names),
never by id, so a backup restores into any tenant.

## Required Context

- [ ] **Tenant**: confirm the active tenant with `cap status --json` and `cap auth whoami --json` before
  downloading, and again before uploading.
- [ ] **Output folder**: where the `.xlsx` files are written.

---

## Step 1: Download

Run `download-excel` for each entity you need. Every command takes `--output <file>`:

```bash
cap masterdata units download-excel --output backup/units.xlsx --json
cap model inputs download-excel --output backup/inputs.xlsx --json
cap templates widget-templates download-excel --output backup/widget-templates.xlsx --json
```

## Step 2: Upload in dependency order

Upload in this order, so that every name a file refers to already exists in the target tenant.
Skip entities you did not download.

| Order | Entities | Why here |
|---|---|---|
| 1 | `masterdata units` | Everything with a unit needs it. |
| 2 | `masterdata data-sources` | Inputs and narrative data feeds name their data source. |
| 3 | `masterdata org-node-attribute-types`, `masterdata org-nodes` | Org nodes carry attribute values. |
| 4 | `masterdata framework-attribute-types`, `masterdata frameworks` | Frameworks carry attribute values. |
| 5 | `masterdata discipline-attribute-types`, `masterdata disciplines` | Disciplines carry attribute values. |
| 6 | `model metric-attribute-types` | Metrics carry attribute values. |
| 7 | `model inputs`, `model input-overrides` | Overrides name an input and an org node. |
| 8 | `model calculations`, `model calculation-overrides` | Formulas reference inputs. |
| 9 | `model metric-framework-nodes` | Assigns metrics to frameworks. |
| 10 | `masterdata units` **again** | Conversions whose factor is a metric are skipped on the first pass, because the metric did not exist yet. |
| 11 | `model narrative-attribute-types`, `model narratives`, `model narrative-overrides` | Narratives come after the metrics and before the templates. |
| 12 | `model metric-org-node-exclusions` | Excludes inputs and calculations, so it follows both. It does not cover narratives. |
| 13 | `model narrative-org-node-exclusions` | Sheet "Narrative Org Node Exclusions", columns `Org Node Path` and `Narrative`. Upload only adds exclusions; it removes none. |
| 14 | `templates org-node-templates`, `templates capture-templates`, `templates report-templates`, `templates widget-templates`, `templates dashboard-templates` | In that order: dashboards place widgets, and spreadsheet templates can use org node templates. |
| 15 | `data input-values` (captured data, optional) | Values need their metrics, periods and exclusions. A value at an excluded org node fails the whole upload. |
| 16 | `data input-values attachments` (evidence files, optional) | Files are added to values, so the values must exist first. Excel cannot carry files; see below. |

Second passes:

- Step 10 fills in conversions such as a currency conversion whose factor is an exchange-rate metric. The first units upload warns `Re-import units after importing metrics`. Until the second pass, values posted in the other currency fail with `No conversion is configured`.
- An inputs, calculations or narratives upload that warns `Bands not imported: referenced metric '…' not found` saved the metric without those bands. Upload that workbook again after step 10.
- A value posted in a converted unit is skipped until the conversion rate for its period has been calculated. Run `cap data recalculation wait --current --json`, then upload that values workbook again.

Read every upload result in full: warnings exit `0`.

Masterdata and model uploads take the file as an argument; template and input-value uploads need `-f`:

```bash
cap masterdata units upload-excel backup/units.xlsx --json
cap model inputs upload-excel backup/inputs.xlsx --json
cap templates widget-templates upload-excel -f backup/widget-templates.xlsx --json
```

`cap workflows show backup-restore --json` returns the same order.

### Evidence files (step 16)

The values workbook carries numbers, narrative text and comments, not the files attached to them. Copy those with the attachments commands, once per capture template and data interval you copied values for. Between environments (for example PROD to QA), point the CLI at the other API and log in between the two commands:

```bash
# source
cap data input-values attachments download --template <id> --data-interval month --periods "<names>" --output backup/files-month --json

# target (export CAPSTONE_API_URL=<target url>; cap auth login; cap auth switch-tenant <id> when it is another environment)
cap data input-values attachments upload backup/files-month --dry-run --json
cap data input-values attachments upload backup/files-month --json
```

Every file reports an outcome. Re-running the upload after fixing a failure adds only what is missing (`already-present` for the rest). Files on values at excluded org nodes, outside the template, or outside your view scope are not copied. See [Copy attachments between tenants](../../reference/commands.md#copy-attachments-between-tenants).

## Step 3: Verify

```bash
cap data recalculation wait <target-model-version> --json
cap templates dashboard-templates audit <dashboard-template-id> --strict --json
```

Then check a few dashboards with the [Verify Data](../reporting/verify-data.md) recipe.

## Notes

- Uploads match existing rows by exact name or path, so re-running an upload updates rather than duplicates.
- Captured data (input values) is downloaded through a capture template; see [Bulk Import Data](./bulk-import.md). Upload it after the templates (step 14), then its files (step 16).
- Seeding a new tenant from prepared workbooks follows the same order; see [Seed Tenant](./seed-tenant.md).
