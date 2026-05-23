# Output Formats

> How the CLI formats responses in table and JSON modes.

---

## Output Modes

Every command supports two output modes:

| Mode | Flag | Stdout | Stderr | Use Case |
|------|------|--------|--------|----------|
| **Table** (default) | (none) | Formatted ASCII table | Errors, warnings | Human reading |
| **JSON** | `--json` | Structured JSON | JSON errors/warnings | AI/automation, piping |

**Tip:** Always use `--json` when the output will be processed programmatically.

---

## List Output (Collections)

### Table Mode

```
ID         | Name                  | Discipline        | Unit
-----------|-----------------------|-------------------|--------
<id>  | Electricity Usage     | Energy            | kWh
<id>  | Water Consumption     | Water             | kL
<id>  | Safety Incidents      | Health & Safety   | count
```

- Column widths auto-size to fit the longest value
- IDs are truncated to 8 characters + `...` for readability
- Columns are separated by ` | ` with a divider row

### JSON Mode

```json
{
  "items": [
    {
      "id": "<id>",
      "name": "Electricity Usage",
      "discipline": "Energy",
      "unit": "kWh"
    },
    {
      "id": "<id>",
      "name": "Water Consumption",
      "discipline": "Water",
      "unit": "kL"
    }
  ]
}
```

- Full IDs (not truncated)
- Property names in `camelCase`
- Null values omitted
- Pretty-printed with indentation

---

## Tree List Output (Hierarchical Entities)

Entities like org nodes, disciplines, and frameworks display as trees.

### Table Mode

```
ID         | Name                  | Level | Path
-----------|-----------------------|-------|------------------------------
<id>  | Corporate             | 0     | Corporate
<id>  |   Region 1            | 1     | Corporate > Region 1
<id>  |     Site A            | 2     | Corporate > Region 1 > Site A
<id>  |     Site B            | 2     | Corporate > Region 1 > Site B
<id>  |   Region 2            | 1     | Corporate > Region 2
```

- Indentation indicates hierarchy depth
- Path column shows the full breadcrumb

### JSON Mode

```json
{
  "items": [
    {
      "id": "<id>",
      "name": "Corporate",
      "level": 0,
      "path": "Corporate",
      "children": [
        {
          "id": "<id>",
          "name": "Region 1",
          "level": 1,
          "path": "Corporate > Region 1"
        }
      ]
    }
  ]
}
```

---

## Single Entity Output (Get)

### Table Mode

```
Field                    Value
------------------------ ------------------------------------------
ID                       <id>
Name                     Electricity Usage
Description              Monthly electricity consumption at site
Discipline               Environmental > Energy
Unit                     kWh
Data Interval            Month
Org Aggregation          Sum
Time Aggregation         Sum
```

- Key-value pairs with label padding for alignment

### JSON Mode

Returns the full entity DTO:

```json
{
  "tenant": "Example Tenant",
  "id": "<id>",
  "name": "Electricity Usage",
  "description": "Monthly electricity consumption at site",
  "discipline": {
    "id": "<id>",
    "name": "Energy",
    "path": "Environmental > Energy"
  },
  "unitOfMeasure": {
    "id": "<id>",
    "name": "kWh"
  },
  "dataInterval": { "id": 2, "name": "Month" },
  "orgStructureAggregationMethod": { "id": 0, "name": "Sum" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" }
}
```

---

## Create / Save / Delete Output

### Table Mode

```
Created Metric with ID: <id>
```

```
Deleted Metric with ID: <id>
```

### JSON Mode

**Create / Save:**
```json
{
  "success": true,
  "id": "<id>"
}
```

**Delete:**
```json
{
  "success": true,
  "deleted": "<id>"
}
```

---

## Narrative Output

Narrative commands follow the same table/JSON split, with API contract objects
inside CLI envelopes:

| Command | JSON Shape |
|---------|------------|
| `cap model narratives list --json` | `{ "tenant": "...", "data": [NarrativeTreeGridRow], "pagination": {...} }` |
| `cap model narratives get <id> --json` | `{ "tenant": "...", "narrative": NarrativeDTO }` |
| `cap data narrative-values list --json` | `{ "tenant": "...", "data": GetNarrativeValueTreeGridListResponse }` |
| `cap data narrative-values get <id> --json` | `{ "tenant": "...", "narrativeValue": NarrativeValueDTO }` |
| `cap data narrative-values history <id> --json` | `{ "tenant": "...", "narrativeValueId": "...", "changes": [NarrativeValueChangeDTO] }` |
| `cap data narrative-change-requests list --json` | `{ "tenant": "...", "data": [NarrativeChangeRequestGridRow], "pagination": {...} }` |
| `cap data data-lock lock|unlock --json` | `{ "tenant": "...", "success": true, "id": "...", "action": "lock|unlock", ... }` |

Use `--json` for every step in an automated narrative workflow. Table output is
for humans and may truncate IDs.

---

## Excel Upload Output

### Table Mode (Success)

```
Upload successful: 42 Metric(s) imported
```

### Table Mode (Partial Success)

```
Warning: Upload completed with errors: 40 imported, 2 error(s)
Row | Column | Error
----|--------|----------------------------------
3   | 2      | Invalid metric name format
5   | 4      | Duplicate entry
```

### JSON Mode

```json
{
  "items": [
    { "id": "...", "name": "Metric A" },
    { "id": "...", "name": "Metric B" }
  ],
  "uploadErrors": [
    {
      "rowIndex": 3,
      "columnIndex": 2,
      "errorMessage": "Invalid metric name format"
    }
  ],
  "isValid": false
}
```

- `isValid: true` means all rows imported successfully
- `isValid: false` means some rows had errors (partial success)
- `items` contains successfully imported records
- `uploadErrors` lists row/column/message for each failure

---

## Excel Download Output

### Table Mode

```
Downloaded to /path/to/metrics.xlsx (2,458,624 bytes)
```

### JSON Mode

```json
{
  "filePath": "/path/to/metrics.xlsx",
  "fileSize": 2458624
}
```

---

## Widget Data Output

Widget commands have two output families:

| Command | Output Shape | Use Case |
|---------|--------------|----------|
| `cap reporting widgets get-data` | CSV string, wrapped as `{ "csv": "..." }` in JSON mode | AI/automation extraction across widget types |
| `cap reporting widgets info-card` | Typed info-card DTO | Dashboard card render validation |
| `cap reporting widgets pie-chart` | Typed pie-chart DTO | Slice/percentage validation |
| `cap reporting widgets xy-chart` | Typed XY-chart DTO, JSON-first | Series/category/value validation |
| `cap reporting widgets table` | Typed Table render DTO | Dashboard grid/table render validation |

### Table Widget JSON Mode

`cap reporting widgets table <id> --json` returns the shared dashboard render contract:

```json
{
  "id": "<id>",
  "title": "Ambient Noise by Site",
  "columns": [
    { "stableKey": "org-node", "header": "Org Node", "widthMode": { "id": 2, "name": "Fill" } },
    { "stableKey": "current-value", "header": "Current Value", "formatter": { "id": 0, "name": "Number" }, "unit": "dB(A)", "precision": 2 }
  ],
  "columnGroups": [],
  "rows": [
    {
      "rowKey": "row-1",
      "headers": { "org-node": "Site A" },
      "cells": [
        {
          "columnKey": "current-value",
          "rawValue": 61,
          "displayValue": "61.00 dB(A)",
          "hasValue": true,
          "isComputed": false
        }
      ]
    }
  ],
  "totalResolvedRowCount": 1,
  "renderedRowCount": 1,
  "renderedColumnCount": 2,
  "totalResolvedColumnCount": 2,
  "projectedCellCount": 2,
  "configuredCellBudget": 2000,
  "truncated": false,
  "warnings": []
}
```

### Table Widget Table Mode

Without `--json`, the CLI prints a readable table using the returned `columns` and `rows`, followed by row/column/cell counts. Warnings and truncation notices are written as warnings.

```text
Tenant: Example Tenant

Ambient Noise by Site

Org Node | Current Value
---------|--------------
Site A   | 61.00 dB(A)

Rows: 1/1, Columns: 2, Cells: 2
```

`widgets table` requires `--data-interval` and plural `--org-nodes`. `widgets get-data` uses singular `--org-node` and auto-detects the data interval from the widget template.

---

## Error Output

Errors are written to **stderr** (not stdout), so they don't interfere with piped JSON output.

### Table Mode

```
Error: Metric not found
  id: <id>
  resourceType: Metric
```

Auth errors include a hint:
```
Error: Authentication required
  Hint: Run 'cap auth login' to authenticate
```

### JSON Mode

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Metric not found"
  }
}
```

### Error Codes

| Code | Meaning | Resolution |
|------|---------|------------|
| `AUTH_REQUIRED` | Not logged in | Run `cap auth login` |
| `AUTH_EXPIRED` | Token expired | Re-run command (auto-refresh) or `cap auth login` |
| `TENANT_REQUIRED` | No tenant selected | Run `cap auth switch-tenant <id>` |
| `NOT_FOUND` | Entity doesn't exist | Verify ID with `list` command |
| `VALIDATION_ERROR` | Invalid input data | Check JSON structure matches DTO |
| `API_ERROR` | Server-side error | Check API status, retry |
| `NETWORK_ERROR` | Connection failed | Check network, verify API URL in config |
| `FORBIDDEN` | No permission | Check user roles and data permissions |

---

## Warning Output

Warnings are written to **stderr** in both modes.

### Table Mode

```
Warning: Upload completed with errors: 40 imported, 2 error(s)
```

### JSON Mode

```json
{
  "warning": "Upload completed with errors: 40 imported, 2 error(s)"
}
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Error (auth, API, validation, network) |

---

## Piping and Composition

JSON mode is designed for piping:

```bash
# Get metric IDs and pipe to another tool
cap model metrics list --json | jq '.items[].id'

# Chain commands
METRIC_ID=$(cap model inputs create --file input.json --json | jq -r '.id')
cap model inputs get $METRIC_ID --json
```

Table mode is designed for human reading:

```bash
# Quick scan
cap model metrics list

# Filtered view (combine with grep)
cap model metrics list | grep "Energy"
```

---

## See Also

- [Commands Reference](./commands.md) — Full command listing
- [Glossary](./glossary.md) — Term definitions
- [SKILL.md](../SKILL.md) — CLI overview
