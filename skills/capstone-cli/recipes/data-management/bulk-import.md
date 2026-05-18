# Recipe: Bulk Import Data

> Upload multiple input values via Excel spreadsheet.

## When to Use

- "Import this spreadsheet"
- "Bulk upload data from Excel"
- "I have a file with multiple values"
- "Upload [filename].xlsx"

## Required Context

Before starting, Claude needs:
- [ ] **Capture Template** — Which template defines the data structure
- [ ] **Data Interval** — Interval used by the template (`day|week|month|quarter|year`)
- [ ] **Excel File** — User's file with data to import

---

## Step-by-Step

### Step 1: Identify Capture Template

**Purpose:** Determine which template defines the expected data structure.

**Command:**
```bash
cap templates capture-templates list --json
```

**Present to user:**
```
Available capture templates:

1. Monthly Energy Data
   - Metrics: Electricity, Gas, Diesel
   - Period type: Monthly

2. Quarterly Safety Report
   - Metrics: Incidents, Near Misses, Training Hours
   - Period type: Quarterly

3. Annual Emissions Summary
   - Metrics: Scope 1, Scope 2, Scope 3
   - Period type: Annual

Which template matches your data?
```

**If user unsure:**
- "What metrics are in your spreadsheet?"
- Match against template configurations

---

### Step 2: Download Template (Optional but Recommended)

**Purpose:** Get the correct format if user doesn't have it.

**Command:**
```bash
cap data input-values download-excel \
  --template <template-id> \
  --data-interval month \
  --periods "Jan 2025" \
  -o template.xlsx
```

**Explain template structure:**
```
Template downloaded: template.xlsx

Structure:
┌──────────────┬──────────────┬─────────────────────┬───────────┐
│ Period       │ Org Node     │ Electricity (kWh)   │ Gas (m³)  │
├──────────────┼──────────────┼─────────────────────┼───────────┤
│ Jan 2025     │ Site A       │                     │           │
│ Jan 2025     │ Site B       │                     │           │
│ Feb 2025     │ Site A       │                     │           │
└──────────────┴──────────────┴─────────────────────┴───────────┘

• Period column: Use exact period names from 'cap data time-periods list'
• Org Node column: Use exact org node names from 'cap masterdata org-nodes list'
• Metric columns: Enter numeric values only
```

---

### Step 3: Guide Template Filling

**Purpose:** Help user fill the template correctly.

**Key rules to explain:**

```
📋 Filling Guidelines

1. PERIODS - Use exact names
   ✅ "Jan 2025", "Q1 FY 25"
   ❌ "January", "1/2025"

   Get valid names: cap data time-periods list --data-interval month --json

2. ORG NODES - Use exact names
   ✅ "Site A", "Corporate > Region 1 > Site A"
   ❌ "site a", "SiteA"

   Get valid names: cap masterdata org-nodes list --json

3. VALUES - Numeric only
   ✅ 1500, 1500.5, 0
   ❌ "1,500", "N/A", blank (leave empty if no data)

4. ROWS - One row per data point
   Each row = one metric value at one location for one period
```

---

### Step 4: Upload the Data

**Purpose:** Import the filled spreadsheet.

**Command:**
```bash
cap data input-values upload-excel -f <user-file.xlsx> --json
```

**What happens:**
1. File is validated row-by-row
2. Valid rows are saved
3. Invalid rows are reported with errors

---

### Step 5: Handle Results

**Purpose:** Process the upload response and guide next steps.

#### Full Success

```
✅ Import successful!

Summary:
- Total rows processed: 48
- Saved: 48
- Errors: 0

All data has been imported successfully.
```

#### Partial Success

```
⚠️ Partial import completed

Summary:
- Total rows processed: 48
- Saved: 42
- Errors: 6

Successfully imported 42 values.

Failed rows:
┌─────┬────────────┬──────────────┬──────────────────────────────┐
│ Row │ Period     │ Org Node     │ Error                        │
├─────┼────────────┼──────────────┼──────────────────────────────┤
│ 12  │ Mar 2025   │ Site A       │ Period is locked             │
│ 15  │ March 25   │ Site B       │ Period not found             │
│ 23  │ Q1 FY 25   │ Site C       │ Org node not found           │
│ 31  │ Jan 2025   │ Site D       │ Permission denied            │
│ 45  │ Feb 2025   │ Site A       │ Invalid value: "N/A"         │
│ 46  │ Feb 2025   │ Site B       │ Invalid value: "1,500"       │
└─────┴────────────┴──────────────┴──────────────────────────────┘

To fix:
1. Correct the values in your Excel file
2. Remove successfully imported rows
3. Re-upload the fixed file
```

#### Complete Failure (Validation)

```
❌ Import failed - validation errors

No data was imported. Please fix the following:

File format errors:
- Row 1: Missing "Period" column header
- Row 1: Unknown column "Electricty" (did you mean "Electricity"?)

Fix the column headers and try again.
```

---

### Step 6: Verify Import

**Purpose:** Confirm data was saved correctly.

**Command:**
```bash
cap data input-values list --template <template-id> --data-interval month --periods "Jan 2025" --json
```

**Or spot-check specific values:**
```bash
cap data input-values get <id> --json
```

---

## Example Interaction

**User:** "I have an Excel file with monthly energy data to upload"

**Claude:** I'll help you import that data. First, which capture template should we use?

```bash
cap templates capture-templates list --json
```

Available templates:
1. **Monthly Energy Data** — Electricity, Gas, Diesel (Monthly)
2. **Quarterly Safety Report** — Incidents, Near Misses (Quarterly)

**User:** "Monthly Energy Data"

**Claude:** Great. Do you need the template format, or is your file already in the right structure?

**User:** "Show me the format"

**Claude:**
```bash
cap data input-values download-excel \
  --template abc123 \
  --data-interval month \
  --periods "Jan 2025" \
  -o energy-template.xlsx
```

Here's the expected structure:
```
│ Period     │ Org Node   │ Electricity (kWh) │ Gas (m³) │ Diesel (L) │
│ Jan 2025   │ Site A     │ [numeric value]   │ ...      │ ...        │
```

Key requirements:
- Period names must match exactly (use `cap data time-periods list --data-interval month` to see valid names)
- Org Node names must match exactly
- Values must be numeric

**User:** "My file is ready at /path/to/energy-data.xlsx"

**Claude:**
```bash
cap data input-values upload-excel -f /path/to/energy-data.xlsx --json
```

Result:
```
✅ Import successful!

- Total rows: 24
- Saved: 24
- Errors: 0
```

All data imported successfully! Would you like to verify any specific values?

---

## Variations

### Large File (> 1000 rows)

If user has a very large file:
- "Large files may take a moment to process. The upload will continue in the background."
- Consider splitting into multiple files if issues occur

### Different Period Types

If template expects different intervals:
- Monthly template → "Jan 2025", "Feb 2025"
- Quarterly template → "Q1 FY 25", "Q2 FY 25"
- Annual template → "FY 2024", "FY 2025"

### Re-importing After Fixes

If user is re-uploading fixed data:
- "Existing values will be overwritten. Is that okay?"
- Suggest removing already-successful rows to avoid duplicate processing

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| "Period not found" | Period name doesn't match | Use exact name from `time-periods list` |
| "Org node not found" | Location name doesn't match | Use exact name from `org-nodes list` |
| "Period is locked" | Data entry locked for that period | Use Change Request workflow |
| "Permission denied" | No access to that location/discipline | Contact administrator |
| "Invalid value" | Non-numeric or malformed value | Use numbers only, no commas |
| "Missing column" | Required column not in file | Check column headers match template |

---

## Alternative: JSON Batch Save

For patterned data (same rate across many nodes/periods, seed data, plan metrics), consider the **programmatic batch save** approach instead of Excel:

```bash
python3 generate_values.py | cap data input-values save --json
```

**Advantages over Excel:**
- No file to manage — generate and pipe directly
- Scriptable and repeatable
- Business-key upsert — safe to re-run (idempotent)
- Full control over JSON structure

**See:** [Enter Data > Programmatic Batch Save](./enter-data.md#programmatic-batch-save-10-values) for the pattern.

---

## Related Recipes

- [Enter Data](./enter-data.md) — Single value entry and programmatic batch save ✅
- [Review Approvals](./review-approvals.md) — Approve imported values ✅
- [Discover Tenant Structure](../exploration/discover-tenant-structure.md) — See valid org nodes ✅
