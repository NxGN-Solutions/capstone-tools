# Recipe: Seed Tenant from Excel

> Set up a complete Tharisa tenant by uploading Excel fixture files in dependency order.

## When to Use

- "Seed a fresh database with Tharisa data"
- "Set up the tenant for testing"
- "Upload all the Excel fixtures"
- "I have a blank database, set it up"
- "Import the Tharisa data sheets"

## Required Context

Before starting, you need:
- [ ] **Running API** — Capstone API accessible at a known URL
- [ ] **Authentication** — Valid login session (`cap auth login`)
- [ ] **Tharisa tenant** — Selected as current tenant (`cap auth whoami`)
- [ ] **Excel fixture files** — Located in `NxGN.Capstone.Cli.E2ETests/Fixtures/`

---

## Step-by-Step

### Step 1: Verify Prerequisites

**Purpose:** Ensure API is reachable, authenticated, and on the correct tenant.

```bash
# Set API URL (check Aspire dashboard for actual port)
export CAPSTONE_API_URL=https://localhost:<port>

# Verify authentication and tenant
cap auth whoami --json
```

**Expected output:**
```json
{
  "tenantName": "Tharisa ...",
  "tenantId": "019c36c8-..."
}
```

**If not on Tharisa:**
```bash
# List available tenants
cap auth tenants --json

# Switch to the Tharisa tenant
cap auth switch-tenant <tharisa-tenant-id>
```

**If not authenticated:**
```bash
cap auth login
```

---

### Step 2: Upload Master Data (Steps 1–5)

**Purpose:** Seed foundational reference data. These have no dependencies on each other.

Master data commands accept the file as a **positional argument** (no `-f` flag needed).

```bash
FIXTURES="NxGN.Capstone.Cli.E2ETests/Fixtures"

# 1. Units of Measure
cap masterdata units upload-excel "$FIXTURES/Master Data/Tharisa - Units of Measure.xlsx" --json

# 2. Data Sources
cap masterdata data-sources upload-excel "$FIXTURES/Master Data/Tharisa - Data Sources.xlsx" --json

# 3. Org Nodes (organizational hierarchy)
cap masterdata org-nodes upload-excel "$FIXTURES/Master Data/Tharisa - Org Structure.xlsx" --json

# 4. Disciplines (metric categories)
cap masterdata disciplines upload-excel "$FIXTURES/Master Data/Tharisa - Discipline Structure.xlsx" --json

# 5. Frameworks (reporting frameworks like GRI, CDP)
cap masterdata frameworks upload-excel "$FIXTURES/Master Data/Tharisa - Framework Structure.xlsx" --json
```

**Verify:**
```bash
cap masterdata units list --json            # Should show ~36 units
cap masterdata disciplines list --json      # Should show ~80 disciplines
cap masterdata org-nodes list --json        # Should show org hierarchy
```

---

### Step 3: Upload Model (Steps 6–8)

**Purpose:** Seed metric definitions. Inputs and Calculations depend on Units and Disciplines from Step 2.

Model commands also accept the file as a **positional argument**.

```bash
# 6. Metric Attribute Types
cap model metric-attribute-types upload-excel "$FIXTURES/Model/Tharisa - Metric Attribute Types.xlsx" --json

# 7. Inputs (depends on: units, disciplines, data sources)
cap model inputs upload-excel "$FIXTURES/Model/Tharisa - Inputs.xlsx" --json

# 8. Calculations (depends on: units, disciplines, inputs for formulas)
cap model calculations upload-excel "$FIXTURES/Model/Tharisa - Calculations.xlsx" --json
```

**Verify:**
```bash
cap model inputs list --json --limit 5       # Should show ~386 inputs
cap model calculations list --json --limit 5  # Should show calculations
```

---

### Step 4: Upload Templates (Steps 9–12)

**Purpose:** Seed report, capture, widget, and dashboard templates. These depend on metrics from Step 3.

> **Important:** Template commands require the **`-f` flag** for the file path. Using a positional argument will silently fail with "Argument not recognized".

```bash
# 9. Report Templates
cap templates report-templates upload-excel -f "$FIXTURES/Templates/Tharisa - Report Templates.xlsx" --json

# 10. Capture Templates (spreadsheet capture definitions)
cap templates capture-templates upload-excel -f "$FIXTURES/Templates/Tharisa - Spreadsheet Templates.xlsx" --json

# 11. Widget Templates (depends on: metrics)
cap templates widget-templates upload-excel -f "$FIXTURES/Templates/Tharisa - Widget Templates.xlsx" --json

# 12. Dashboard Templates (depends on: widget templates)
cap templates dashboard-templates upload-excel -f "$FIXTURES/Templates/Tharisa - Dashboard Templates.xlsx" --json
```

**Verify:**
```bash
cap templates widget-templates list --json --limit 5      # Should show ~141 widget templates
cap templates dashboard-templates list --json --limit 5    # Should show dashboard templates
cap templates capture-templates list --json --limit 5      # Should show ~20 capture templates
```

---

### Step 5: Upload Input Values (Steps 13–14)

**Purpose:** Seed actual data values so that computed values and widget data are available. Upload yearly first, then monthly.

> **Important:** Input value upload also requires the **`-f` flag**.

```bash
# 13. Yearly input values (use Year data interval)
cap data input-values upload-excel -f "$FIXTURES/Data/Tharisa - Yearly Inputs.xlsx" --json

# 14. Monthly input values (use Month data interval)
cap data input-values upload-excel -f "$FIXTURES/Data/Tharisa - Monthly Inputs.xlsx" --json
```

**Verify data is available:**
```bash
# Check input values exist
cap data input-values list --json --limit 5

# Check that computed values are being calculated (may take a moment)
cap reporting widgets info-card <widget-id> \
  --org-nodes <org-node-id> \
  --data-interval year \
  --periods "FY 25" \
  --json
```

---

## Idempotent Behavior

All uploads are **idempotent** — re-uploading the same file updates existing data rather than creating duplicates. An upload that returns "0 items" means the data already exists and is up to date. This is normal and not an error.

---

## The `-f` Flag Gotcha

This is the most common pitfall when seeding:

| Command Type | File Argument | Example |
|-------------|---------------|---------|
| Master data (`masterdata *`) | **Positional** (no flag) | `cap masterdata units upload-excel file.xlsx` |
| Model (`model *`) | **Positional** (no flag) | `cap model inputs upload-excel file.xlsx` |
| Templates (`templates *`) | **`-f` flag required** | `cap templates widget-templates upload-excel -f file.xlsx` |
| Input values (`data input-values`) | **`-f` flag required** | `cap data input-values upload-excel -f file.xlsx` |

If a template upload returns no items or silently does nothing, check that you're using `-f`.

---

## Automated Seeding (E2E Tests)

The E2E test suite includes `TenantSetupFixture` which automates the entire seeding sequence. Tests that need a fully-seeded tenant use the `[Collection("TenantSetup")]` attribute:

```bash
# Build the CLI first
dotnet build NxGN.Capstone.Cli

# Run any test in the TenantSetup collection to trigger seeding
dotnet test NxGN.Capstone.Cli.E2ETests --filter "Collection=TenantSetup"
```

See [Testing Architecture > Tenant Seeding](../../designs/cli/11-testing.md#tenant-seeding-with-excel) for details on the automated fixture.

---

## Related Recipes

- [Bulk Import Data](./bulk-import.md) — Import data for a specific capture template
- [Enter Data](./enter-data.md) — Manual single-value data entry
- [Discover Tenant Structure](../exploration/discover-tenant-structure.md) — Explore what was seeded
