# Recipe: Export to Excel

> Download entity data to Excel and re-import after editing.

## When to Use

- "Export our metrics to Excel"
- "Download the org structure as a spreadsheet"
- "I need to bulk-edit disciplines in Excel"
- "Export and re-import [entity]"
- "How do I do bulk updates?"

## Required Context

Before starting, Claude should know:
- [ ] Which entity type to export (metrics, org-nodes, units, etc.)
- [ ] Optional: Whether they plan to re-import after editing

**If missing:** Claude will ask what they want to export.

---

## Step-by-Step

### Step 1: Identify the Entity

**Purpose:** Determine which entity to export.

**Entities supporting Excel export/import:**

| Domain | Entity | Command Prefix |
|--------|--------|----------------|
| `model` | inputs | `cap model inputs` |
| `model` | calculations | `cap model calculations` |
| `model` | metric-attribute-types | `cap model metric-attribute-types` |
| `model` | metric-framework-nodes | `cap model metric-framework-nodes` |
| `masterdata` | org-nodes | `cap masterdata org-nodes` |
| `masterdata` | disciplines | `cap masterdata disciplines` |
| `masterdata` | frameworks | `cap masterdata frameworks` |
| `masterdata` | units | `cap masterdata units` |
| `masterdata` | data-sources | `cap masterdata data-sources` |
| `masterdata` | discipline-attribute-types | `cap masterdata discipline-attribute-types` |
| `masterdata` | framework-attribute-types | `cap masterdata framework-attribute-types` |
| `masterdata` | org-node-attribute-types | `cap masterdata org-node-attribute-types` |
| `templates` | capture-templates | `cap templates capture-templates` |
| `templates` | report-templates | `cap templates report-templates` |
| `templates` | widget-templates | `cap templates widget-templates` |
| `templates` | dashboard-templates | `cap templates dashboard-templates` |
| `templates` | org-node-templates | `cap templates org-node-templates` |
| `data` | input-values | `cap data input-values` (requires `--template`) |
| `reporting` | computed-values | `cap reporting computed-values` (requires `--template`, `--data-interval`, `--periods`) |

---

### Step 2: Download to Excel

**Purpose:** Export the current data.

**Command:**
```bash
cap <domain> <entity> download-excel -o <filename>.xlsx
```

**Examples:**
```bash
# Export all input metrics
cap model inputs download-excel -o inputs.xlsx

# Export org structure
cap masterdata org-nodes download-excel -o org-nodes.xlsx

# Export input values (requires capture template)
cap data input-values download-excel --template <capture-template-id> -o values.xlsx

# Export computed values (requires report template + data interval + periods)
cap reporting computed-values download-excel \
  --template <report-template-id> \
  --data-interval month \
  --periods "Jan 2026" \
  -o computed-values.xlsx
```

> **Note:** `-o` is the short flag for `--output`. If omitted, the file downloads to `{Entity}.xlsx` in the current directory.

**Present result:**
```
Downloaded to inputs.xlsx (245,862 bytes)

The file contains all input metrics with their current configuration.
You can now open it in Excel, make changes, and re-import.
```

---

### Step 3: Guide Editing (If Re-importing)

**Purpose:** Explain what can be safely edited.

**General rules:**
```
Editing Guidelines

1. DO NOT modify the ID column
   - Existing IDs identify records for updates
   - Leave ID blank for new records

2. DO NOT change column headers
   - Headers must match exactly for the upload to work

3. Safe to edit:
   - Name, Description, Reference fields
   - Numeric values (precision, sort order)
   - Enum values (use exact names from lookups)

4. Safe to add:
   - New rows with blank IDs → creates new records

5. Safe to delete:
   - Remove rows → those records are NOT deleted (just not updated)
   - To delete records, use the CLI delete command
```

---

### Step 4: Upload Changes

**Purpose:** Re-import the edited spreadsheet.

**Command:**
```bash
cap <domain> <entity> upload-excel <filename>.xlsx --json
```

**Examples:**
```bash
# Re-import edited metrics
cap model inputs upload-excel inputs.xlsx --json

# Re-import input values
cap data input-values upload-excel values.xlsx --json
```

---

### Step 5: Handle Upload Results

**Purpose:** Process the response and address any errors.

#### Full Success

```
Upload successful: 42 record(s) imported

All changes have been applied.
```

#### Partial Success

```
Warning: Upload completed with errors: 40 imported, 2 error(s)

Row | Column | Error
----|--------|----------------------------------
12  | 3      | Invalid discipline name
28  | 5      | Unit of measure not found

To fix:
1. Correct the errors in your Excel file
2. Remove successfully imported rows (optional)
3. Re-upload: cap model inputs upload-excel inputs.xlsx --json
```

**Common errors and fixes:**

| Error | Cause | Fix |
|-------|-------|-----|
| "Not found" | Invalid reference (discipline, unit, etc.) | Use exact names from `list` commands |
| "Duplicate" | Two rows with same identifying fields | Remove the duplicate row |
| "Required field" | Empty cell in a required column | Fill in the value |
| "Invalid format" | Wrong data type (text in numeric column) | Use correct format |

---

### Step 6: Verify Changes

**Purpose:** Confirm the changes were applied correctly.

**Command:**
```bash
cap <domain> <entity> list --json
```

Or spot-check specific records:
```bash
cap <domain> <entity> get <id> --json
```

---

## Example Interaction

**User:** "I need to update the descriptions on all our input metrics"

**Claude:** I'll help you bulk-edit metric descriptions via Excel. Let me export them first.

```bash
cap model inputs download-excel -o inputs.xlsx
```

Downloaded to `inputs.xlsx`. The file contains all input metrics with columns for ID, Name, Description, Discipline, Unit, etc.

**Edit the Description column** for the metrics you want to update, then save the file.

**Important:**
- Don't change the ID column
- Don't modify column headers

Ready to re-import when you're done editing.

**User:** "Done, file is updated"

**Claude:**
```bash
cap model inputs upload-excel inputs.xlsx --json
```

Result: 35 metrics updated successfully, 0 errors.

All description changes have been applied.

---

## Variations

### Export-Only (Reporting)

If the user just wants data for external use (no re-import):

```bash
# Export computed values for a report
cap reporting computed-values download-excel \
  --template <report-template-id> \
  --data-interval quarter \
  --periods "Q1 FY 2026" \
  -o q1-report.xlsx
```

### Bulk Create via Excel

To create many new records at once:
1. Download to get the column format: `cap <domain> <entity> download-excel -o template.xlsx`
2. Add new rows with blank ID column
3. Upload: `cap <domain> <entity> upload-excel template.xlsx --json`

### Round-Trip Workflow

The full export → edit → import cycle:
```bash
# 1. Export
cap masterdata units download-excel -o units.xlsx

# 2. Edit in Excel (add new units, update descriptions)

# 3. Re-import
cap masterdata units upload-excel units.xlsx --json

# 4. Verify
cap masterdata units list --json
```

---

## Related Recipes

- [Bulk Import Data](../data-management/bulk-import.md) — Import input values specifically ✅
- [Create Metric Wizard](./create-metric.md) — Create individual metrics ✅
- [Discover Tenant Structure](../exploration/discover-tenant-structure.md) — See what entities exist ✅
