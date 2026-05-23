# Recipe: Seed Tenant From Excel

> Set up a tenant by uploading approved implementation workbooks in dependency order.

## When To Use

- "Seed a fresh tenant"
- "Set up the tenant for testing"
- "Upload all the Excel workbooks"
- "I have a blank tenant, set it up"
- "Import the implementation data sheets"

## Required Context

Before starting, you need:

- [ ] **Running API** - Capstone API accessible at a known URL
- [ ] **Authentication** - Valid login session (`cap auth login`)
- [ ] **Target tenant** - Selected as current tenant (`cap auth whoami`)
- [ ] **Approved workbooks** - Local Excel files prepared for the target tenant

Do not use another tenant's exported workbooks unless an implementation owner has confirmed they are safe to reuse.

---

## Step-By-Step

### Step 1: Verify Prerequisites

**Purpose:** Ensure API is reachable, authenticated, and using the intended tenant.

```bash
cap config set api-url <capstone-api-url>
cap auth whoami --json
```

**Expected output:**

```json
{
  "tenantName": "<tenant-name>",
  "tenantId": "<tenant-id>"
}
```

If the wrong tenant is selected:

```bash
cap auth tenants --json
cap auth switch-tenant <tenant-id>
```

If not authenticated:

```bash
cap auth login
```

---

### Step 2: Upload Master Data

**Purpose:** Seed foundational reference data. Master data commands accept the file as a positional argument.

```bash
WORKBOOKS="./workbooks"

cap masterdata units upload-excel "$WORKBOOKS/units-of-measure.xlsx" --json
cap masterdata data-sources upload-excel "$WORKBOOKS/data-sources.xlsx" --json
cap masterdata org-nodes upload-excel "$WORKBOOKS/org-structure.xlsx" --json
cap masterdata disciplines upload-excel "$WORKBOOKS/discipline-structure.xlsx" --json
cap masterdata frameworks upload-excel "$WORKBOOKS/framework-structure.xlsx" --json
```

Verify:

```bash
cap masterdata units list --json
cap masterdata disciplines list --json
cap masterdata org-nodes list --json
```

---

### Step 3: Upload Model Data

**Purpose:** Seed metric definitions. Inputs and calculations depend on units, disciplines, and data sources from the previous step.

```bash
cap model metric-attribute-types upload-excel "$WORKBOOKS/metric-attribute-types.xlsx" --json
cap model inputs upload-excel "$WORKBOOKS/inputs.xlsx" --json
cap model calculations upload-excel "$WORKBOOKS/calculations.xlsx" --json
```

Verify:

```bash
cap model inputs list --json --limit 5
cap model calculations list --json --limit 5
```

---

### Step 4: Upload Templates

**Purpose:** Seed report, capture, widget, and dashboard templates. These depend on metrics from Step 3.

Template commands require the `-f` flag for the file path.

```bash
cap templates report-templates upload-excel -f "$WORKBOOKS/report-templates.xlsx" --json
cap templates capture-templates upload-excel -f "$WORKBOOKS/capture-templates.xlsx" --json
cap templates widget-templates upload-excel -f "$WORKBOOKS/widget-templates.xlsx" --json
cap templates dashboard-templates upload-excel -f "$WORKBOOKS/dashboard-templates.xlsx" --json
```

Verify:

```bash
cap templates widget-templates list --json --limit 5
cap templates dashboard-templates list --json --limit 5
cap templates capture-templates list --json --limit 5
```

---

### Step 5: Upload Input Values

**Purpose:** Seed data values so computed values and widget data are available. Upload larger intervals first, then smaller intervals.

Input value upload also requires the `-f` flag.

```bash
cap data input-values upload-excel -f "$WORKBOOKS/yearly-input-values.xlsx" --json
cap data input-values upload-excel -f "$WORKBOOKS/monthly-input-values.xlsx" --json
```

Verify data is available:

```bash
cap data input-values list --json --limit 5

cap reporting widgets info-card <widget-id> \
  --org-nodes <org-node-id> \
  --data-interval year \
  --periods "FY 25" \
  --json
```

---

## Idempotent Behavior

Uploads are idempotent when the workbook business keys match existing data. Re-uploading the same file updates existing data rather than creating duplicates. An upload that returns zero changed items can mean the tenant is already up to date.

---

## The `-f` Flag Gotcha

| Command Type | File Argument | Example |
|-------------|---------------|---------|
| Master data (`masterdata *`) | Positional | `cap masterdata units upload-excel file.xlsx` |
| Model (`model *`) | Positional | `cap model inputs upload-excel file.xlsx` |
| Templates (`templates *`) | `-f` required | `cap templates widget-templates upload-excel -f file.xlsx` |
| Input values (`data input-values`) | `-f` required | `cap data input-values upload-excel -f file.xlsx` |

If a template upload returns no items or appears to do nothing, check that the command uses `-f`.

---

## Related Recipes

- [Bulk Import Data](./bulk-import.md) - Import data for a specific capture template
- [Enter Data](./enter-data.md) - Manual single-value data entry
- [Discover Tenant Structure](../exploration/discover-tenant-structure.md) - Explore what was seeded
