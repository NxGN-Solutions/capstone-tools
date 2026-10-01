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

| Order | Entities |
|---|---|
| 1. Masterdata | `masterdata units`, `masterdata data-sources`, `masterdata org-node-attribute-types`, `masterdata discipline-attribute-types`, `masterdata framework-attribute-types`, `templates org-node-templates`, `masterdata org-nodes`, `masterdata disciplines`, `masterdata frameworks` |
| 2. Model | `model metric-attribute-types`, `model narrative-attribute-types`, `model inputs`, `model calculations`, `model narratives`, `model metric-framework-nodes`, `model metric-org-node-exclusions`, `model input-overrides`, `model calculation-overrides`, `model narrative-overrides` |
| 3. Templates | `templates capture-templates`, `templates report-templates`, `templates widget-templates`, `templates dashboard-templates` |

Masterdata and model uploads take the file as an argument; template uploads need `-f`:

```bash
cap masterdata units upload-excel backup/units.xlsx --json
cap model inputs upload-excel backup/inputs.xlsx --json
cap templates widget-templates upload-excel -f backup/widget-templates.xlsx --json
```

`cap workflows show backup-restore --json` returns the same order.

## Step 3: Verify

```bash
cap data recalculation wait <target-model-version> --json
cap templates dashboard-templates audit <dashboard-template-id> --strict --json
```

Then check a few dashboards with the [Verify Data](../reporting/verify-data.md) recipe.

## Notes

- Uploads match existing rows by exact name or path, so re-running an upload updates rather than duplicates.
- Captured data (input values) uses its own capture-template workflow; see [Bulk Import Data](./bulk-import.md).
- Seeding a new tenant from prepared workbooks follows the same order; see [Seed Tenant](./seed-tenant.md).
