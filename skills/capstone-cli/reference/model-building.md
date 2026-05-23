# Model Building Quick Reference

> Lookup tables, payload templates, and patterns for creating metrics via CLI.
> This reference eliminates the need for repeated `model lookups get` calls.

---

## Enum Lookup Tables

### Data Interval (Time Period Types)

| ID | Name | Description |
|----|------|-------------|
| 0 | Day | Daily data capture |
| 1 | Week | Weekly data capture |
| 2 | Month | Monthly data capture (most common) |
| 3 | Quarter | Quarterly data capture |
| 4 | Year | Annual data capture |

### Org Structure Aggregation Methods

| ID | Name | Description |
|----|------|-------------|
| 0 | Sum | Values add up through hierarchy (consumption, cost, throughput) |
| 1 | Average | Values average across children (rates, scores, percentages) |
| 2 | Count | Count of non-null child values |
| 3 | None | Node-specific, doesn't roll up |
| 4 | Roll Up | Uses first non-null child value |
| 5 | Roll Down | Cascades parent value to children (assumptions, targets) |

### Time Period Aggregation Methods

| ID | Name | Description |
|----|------|-------------|
| 0 | None | Each period shown separately (time series) |
| 1 | Sum | Values sum across periods (cumulative metrics) |
| 2 | Average | Values average across periods (rates, scores) |
| 3 | Last Value | Most recent period only (snapshots) |
| 4 | Min | Minimum value across periods |
| 5 | Max | Maximum value across periods |

### Calculation Phases

| ID | Name | Description |
|----|------|-------------|
| 0 | Before Aggregations | Compute at leaf nodes, then aggregate. Use for conversions, site-level rates |
| 1 | After Aggregations | Compute after rollup. Use for ratios of aggregated totals, company-wide rates |

### Metric Types

| ID | Name |
|----|------|
| 0 | Input |
| 1 | Calculation |

### Validation Status (Input Values)

| ID | Name |
|----|------|
| 0 | Validation Required |
| 1 | Rejected |
| 2 | Approved |
| 3 | In Progress |

### Symbol Position (Units of Measure)

| ID | Name | Description |
|----|------|-------------|
| 0 | Suffix | Symbol after value (e.g., `100 %`, `50 kg`, `5 l`) |
| 1 | Prefix | Symbol before value (e.g., `$100`, `R250`, `€500`) |

> **Convention (British English):** Currency symbols are **Prefix**, all other units are **Suffix**.

### Widget Types

| ID | Name | Description |
|----|------|-------------|
| 0 | Info Card | Single KPI value with optional comparison |
| 1 | Pie Chart | Part-to-whole breakdown (Pie or Donut) |
| 2 | XY Chart | Time series, comparisons (Column/Bar/Line/Area) |
| 3 | AI Summary | AI-generated insights from sibling/descendant widgets |
| 4 | Table | Metric-backed dashboard table/grid |

### Widget Sizes

| ID | Name | Description |
|----|------|-------------|
| 0 | 25% | Quarter width (4 across) |
| 1 | 33% | Third width (3 across) |
| 2 | 50% | Half width (2 across) |
| 3 | 66% | Two-thirds width |
| 4 | 75% | Three-quarters width |
| 5 | 100% | Full width |

**Recommended defaults:** InfoCards at 50%, daily XY Charts at 100%, site-comparison XY Charts at 50%, PieCharts at 33%, Tables at 100% for dense grids, AI Summary at 100%. See [Build Dashboard](../recipes/configuration/build-dashboard.md#widget-sizing-guidelines) for rationale.

### AI Summary Context

| ID | Name | Description |
|----|------|-------------|
| 0 | Peers | Analyzes sibling widgets in the same section |
| 1 | Descendants | Analyzes all widgets in child sections |

### Pie Chart Types

| ID | Name |
|----|------|
| 0 | Pie |
| 1 | Donut |

### XY Chart Data Item Types

| ID | Name |
|----|------|
| 0 | Column |
| 1 | Bar |
| 2 | Line |
| 3 | Area |

**Best practice:** Always set `rotateCategoryAxisLabels: true` on XY Charts with daily intervals to prevent date label overlap. Monthly/quarterly charts can leave it `false`.

### Metric Partitioning Modes

| ID | Name | Description |
|----|------|-------------|
| 0 | None | Single series for the selected org node |
| 1 | Children | One series per immediate child org node |
| 2 | Descendants | One series per descendant org node |

### Table Row Field Source Types

| ID | Name | Description |
|----|------|-------------|
| 0 | MetricName | Metric display name |
| 1 | MetricReference | Metric reference/code |
| 2 | MetricType | Input or Calculation |
| 3 | Discipline | Metric discipline |
| 4 | Framework | Linked framework names |
| 5 | MetricAttribute | Selected metric attribute value |
| 6 | OrgNodeName | Org-node display name |
| 7 | OrgNodePath | Org-node path |
| 8 | OrgNodeAttribute | Selected org-node attribute value |
| 9 | StaticText | Configured static label |

### Table Value Binding Modes

| ID | Name | Description |
|----|------|-------------|
| 0 | RowMetricValue | Use the metric that generated the row |
| 1 | SelectedMetricValue | Use a specific selected metric for every row |
| 2 | CalculationMetricValue | Use a specific Calculation metric for every row |

### Table Formatting Enums

| Enum | Values |
|------|--------|
| Row Sort Mode | `0 Configured`, `1 DisplayText`, `2 AttributeOrder` (AttributeOrder is not supported for TableVersion 1) |
| Width Mode | `0 Auto`, `1 Fixed`, `2 Fill` |
| Alignment | `0 Left`, `1 Center`, `2 Right` |
| Formatter | `0 Number`, `1 Currency`, `2 Percent`, `3 Text` |
| Unit Mode | `0 MetricDefault`, `1 Hidden`, `2 Header`, `3 Cell` |
| Sort Target Kind | `0 RowField`, `1 ValueField`, `2 RowMetric`, `3 GeneratedPeriodColumn` (GeneratedPeriodColumn is not supported for TableVersion 1) |
| Sort Direction | `0 Ascending`, `1 Descending` |
| Null Placement | `0 Last`, `1 First` |

---

## Common Aggregation Patterns

| Metric Type | Org Agg | Time Agg | Rationale |
|-------------|---------|----------|-----------|
| **Throughput/Volume** | Sum (0) | Sum (1) | Units add up across locations and time |
| **Cost/Revenue ($)** | Sum (0) | Sum (1) | Financial totals are additive |
| **OEE/Rates (%)** | Average (1) | Average (2) | Rates should average, not sum |
| **Assumptions** | Roll Down (5) | Average (2) | Set once, cascades to children |
| **Targets** | Roll Down (5) | Average (2) | Set at parent, inherited by children |
| **Per-unit ratios** | None (3) | None (0) | Recomputed at each level from aggregated inputs |
| **Headcount/Balance** | Last Value (3*) | Last Value (3) | Point-in-time snapshots |

> *Note: "Last Value" is a Time Period Aggregation method (ID 3), not an Org Agg method. For org structure, use None (3) or Sum (0) depending on context.

---

## Payload Templates

### Create Narrative Definition

Narratives are text disclosures, not numeric metrics. Use `model narratives` for
the definition and `data narrative-values` for captured text.

```json
{
  "id": "<empty-id>",
  "name": "<narrative name>",
  "description": "<description>",
  "reference": "<optional-reference>",
  "discipline": {
    "id": "<discipline-id>"
  },
  "captureInterval": {
    "id": 3,
    "name": "Quarter"
  },
  "requireValidation": true,
  "requireDataCapture": true,
  "translatedNames": [],
  "translatedDescriptions": [],
  "frameworkNodes": []
}
```

**Command:**
```bash
cat <<'EOF' | cap model narratives create --json
{ ... payload ... }
EOF
```

Use `cap model narratives lookup --capture-interval quarter --org-nodes <id>
--disciplines <id> --json` before capture workflows. Lock and governed edit
workflows use `cap data data-lock lock|unlock` and
`cap data narrative-change-requests`.

### Create Input

```json
{
  "id": "<empty-id>",
  "name": "<metric name>",
  "description": "<description>",
  "discipline": {
    "id": "<discipline-id>"
  },
  "unitOfMeasure": {
    "id": "<unit-id>"
  },
  "dataInterval": {
    "id": 2,
    "name": "Month"
  },
  "precision": 2,
  "orgStructureAggregationMethod": {
    "id": 0,
    "name": "Sum"
  },
  "timePeriodAggregationMethod": {
    "id": 1,
    "name": "Sum"
  },
  "allowForecastedData": false,
  "requireValidation": false,
  "requireDataCapture": false,
  "inputDataFeeds": [
    {
      "id": "<empty-id>",
      "name": "Manual Capture",
      "dataSource": {
        "id": "<manual-capture-datasource-id>"
      },
      "dataSourceInstruction": "",
      "orgNodeOverrides": []
    }
  ],
  "translatedNames": [],
  "translatedDescriptions": [],
  "attributeValues": []
}
```

**Command:**
```bash
cat <<'EOF' | cap model inputs create --json
{ ... payload ... }
EOF
```

**Response:**
```json
{ "success": true, "id": "<new-id>" }
```

### Create Calculation

```json
{
  "id": "<empty-id>",
  "name": "<calculation name>",
  "description": "<description>",
  "discipline": {
    "id": "<discipline-id>"
  },
  "unitOfMeasure": {
    "id": "<unit-id>"
  },
  "dataInterval": {
    "id": 0,
    "name": "Day"
  },
  "precision": 2,
  "calculationPhase": {
    "id": 1,
    "name": "After Aggregations"
  },
  "orgStructureAggregationMethod": {
    "id": 3,
    "name": "None"
  },
  "timePeriodAggregationMethod": {
    "id": 0,
    "name": "None"
  },
  "formula": "IF [Denominator] <> 0 THEN [Numerator] / [Denominator] ELSE 0",
  "translatedNames": [],
  "translatedDescriptions": [],
  "attributeValues": []
}
```

> **Formula references:** Formulas can reference metrics by name (`[Metric Name]`) or by ID (`[<id>]`). IDs are recommended for CLI usage — they're unambiguous, rename-safe, and what the API stores internally. Use `cap model calculations get <id> --json` to see existing formulas with their ID references.

**Command:**
```bash
cat <<'EOF' | cap model calculations create --json
{ ... payload ... }
EOF
```

### Update Existing (Save)

Same structure as create, but set `id` to the existing metric's ID. Use `save` instead of `create`:

```bash
cat <<'EOF' | cap model inputs save --json
{ "id": "<existing-id>", ... rest of payload ... }
EOF
```

---

## Input Value Operations

### `create` vs `save`

| Command | Purpose | Use When |
|---------|---------|----------|
| `data input-values create` | **Get-or-create** a single input value cell | Initializing a cell to get its ID, or checking if a value exists at a specific business key |
| `data input-values save` | **Batch upsert** one or more values | Setting actual numeric values (new or update) |

**`create`** uses CLI flags (no JSON):
```bash
cap data input-values create \
  --org-node <org-node-id> \
  --input <metric-id> \
  --period-type week \
  --start-date 2026-02-09 \
  --json
```

**`save`** uses JSON input (batch):
```bash
cat payload.json | cap data input-values save --json
```

> In practice, `save` with zero IDs handles both creation and updates. Use `create` only when you need to get the existing value ID for a specific cell.

## Save Input Values

> **Important:** The save endpoint uses **business-key upsert** semantics. Each input value is uniquely identified by its business key: `metric` + `orgNode` + `timePeriodType` + `startDate`. The `id` field can be the zero ID (`<empty-id>`) for **both** new values and updates — the API resolves existing records by business key automatically. You only need existing IDs if you want to be explicit, but it's not required. This makes batch saves idempotent — running the same payload again overwrites with the same values, never creating duplicates.

### TimePeriodType Reference

| ID | Name |
|----|------|
| 0 | Day |
| 1 | Week |
| 2 | Month |
| 3 | Quarter |
| 4 | Year |

### Single Value

```bash
cat <<'EOF' | cap data input-values save --json
{
  "inputValues": [
    {
      "id": "<empty-id>",
      "value": 1500,
      "metric": { "id": "<metric-id>" },
      "orgNode": { "id": "<org-node-id>" },
      "timePeriodType": { "id": 2, "name": "Month" },
      "startDate": "2025-03-01T00:00:00Z"
    }
  ]
}
EOF
```

### Batch Save (Multiple Values)

Send multiple input values in a single request. Each entry needs its own full business key:

```bash
cat <<'EOF' | cap data input-values save --json
{
  "inputValues": [
    {
      "id": "<empty-id>",
      "value": 0.08,
      "metric": { "id": "<metric-a-id>" },
      "orgNode": { "id": "<site-1-id>" },
      "timePeriodType": { "id": 2, "name": "Month" },
      "startDate": "2026-01-01T00:00:00Z"
    },
    {
      "id": "<empty-id>",
      "value": 0.09,
      "metric": { "id": "<metric-a-id>" },
      "orgNode": { "id": "<site-2-id>" },
      "timePeriodType": { "id": 2, "name": "Month" },
      "startDate": "2026-01-01T00:00:00Z"
    },
    {
      "id": "<existing-value-id>",
      "value": 0.10,
      "metric": { "id": "<metric-a-id>" },
      "orgNode": { "id": "<site-3-id>" },
      "timePeriodType": { "id": 2, "name": "Month" },
      "startDate": "2026-01-01T00:00:00Z"
    }
  ]
}
EOF
```

### Using a File

For large payloads, save the JSON to a file and pass it with `--file`:

```bash
cap data input-values save --file /path/to/values.json --json
```

### Workflow: Updating Existing Input Values

1. **List current values** to get IDs and business keys:
   ```bash
   cap data input-values list --template <capture-template-id> --periods "Jan 2026,Feb 2026" --json
   ```

2. **Build payload** — use the existing `id` for updates, or the zero ID for new entries. Include the full business key on every entry.

3. **Save:**
   ```bash
   cat <<'EOF' | cap data input-values save --json
   { "inputValues": [ ... ] }
   EOF
   ```

4. **Verify** by re-listing:
   ```bash
   cap data input-values list --template <capture-template-id> --data-interval week --periods "(W5) Jan 26 2026,(W9) Feb 23 2026" --json
   ```

### Weekly Save Example

Weekly saves use `timePeriodType` ID 1 and `startDate` set to the first day of the week:

```bash
cat <<'EOF' | cap data input-values save --json
{
  "inputValues": [
    {
      "id": "<empty-id>",
      "value": 300,
      "metric": { "id": "<filler-planned-rate-id>" },
      "orgNode": { "id": "<filler-node-id>" },
      "timePeriodType": { "id": 1, "name": "Week" },
      "startDate": "2026-02-09T00:00:00Z"
    },
    {
      "id": "<empty-id>",
      "value": 0.60,
      "metric": { "id": "<cola-plan-pct-id>" },
      "orgNode": { "id": "<site-id>" },
      "timePeriodType": { "id": 1, "name": "Week" },
      "startDate": "2026-02-09T00:00:00Z"
    }
  ]
}
EOF
```

> **Tip:** Get valid week `startDate` values from `cap data time-periods list --data-interval week --json`. Each period's `startDate` field is the value to use.

### Programmatic Batch Save (generate + pipe)

For large batches (50+ values), generate the JSON payload programmatically and pipe it:

```bash
python3 generate_values.py | cap data input-values save --json
```

Or save to a file first (useful for review before submitting):

```bash
python3 generate_values.py > /tmp/values.json
cap data input-values save --file /tmp/values.json --json
```

This is the recommended approach when populating plan metrics, assumptions, or seed data across many org nodes and time periods. It's faster than Excel import and allows full control over the JSON structure.

### List Response Structure

The `data input-values list` response returns a **grid structure** matching the capture template layout:

```json
{
  "timePeriodColumns": [
    { "name": "(W5) Jan 26 2026" },
    { "name": "(W9) Feb 23 2026" }
  ],
  "gridRows": [
    {
      "id": "<grid-row-id>",
      "name": "Cola Plan %",
      "path": "Cola Plan %->Example Enterprise->Example Site 1",
      "isDataRow": true,
      "unitOfMeasure": { "name": "%", "id": "<uom-id>" },
      "precision": 2,
      "groupingKey": "<org-node-id>",
      "timePeriodColumns": [
        {
          "inputValue": {
            "id": "<value-id-or-zero>",
            "value": 0.6,
            "validationStatus": 2,
            "isLocked": false,
            "userCanCapture": true
          }
        },
        {
          "inputValue": {
            "id": "<empty-id>",
            "validationStatus": 0,
            "isLocked": false,
            "userCanCapture": true
          }
        }
      ]
    }
  ]
}
```

**Key points:**
- `gridRows[].timePeriodColumns` aligns with the top-level `timePeriodColumns` by index
- A zero ID in `inputValue.id` means no value has been entered for that cell
- `value` field is absent (not null) when no data exists
- `path` uses `->` separators: `MetricName->OrgPath`
- `groupingKey` is the org node ID for that row

### `--periods` Parameter Format

Specify periods as comma-separated period names (matching output from `time-periods list`):

```bash
# Daily (YYYY-MM-DD format)
--periods "2026-01-15"
--periods "2026-01-15,2026-01-16,2026-01-17"

# Weekly
--periods "(W5) Jan 26 2026"
--periods "(W5) Jan 26 2026,(W9) Feb 23 2026"

# Monthly
--periods "Jan 2026,Feb 2026,Mar 2026"

# Quarterly / Yearly
--periods "Q1 FY 25,Q2 FY 25"
--periods "FY 2025"
```

> **Required:** The `--data-interval` flag is mandatory for the `list` command. Without it, the API returns an error.

---

## Payload Templates: Unit of Measure

### Create/Save Unit of Measure

```json
{
  "id": "<empty-id>",
  "name": "USD",
  "symbol": "$",
  "symbolPosition": {
    "id": 1,
    "name": "Prefix"
  },
  "conversions": []
}
```

**Command:**
```bash
# Create
echo '{ ... }' | cap masterdata units create --json

# Update (set id to existing ID)
echo '{ "id": "<existing-id>", ... }' | cap masterdata units save --json
```

> **Note:** `symbolPosition` is an `EnumDTO` with `id` and `name`. Values: `0`/`Suffix` (default), `1`/`Prefix`. Currency symbols (USD, ZAR, EUR) should use Prefix; all other units use Suffix.

---

## Formula Language Quick Reference

```
[Metric Name]                     Reference another metric by name
[Metric Name]|-3:-1|              Reference last 3 periods
IF condition THEN x ELSE y        Conditional logic
SUM([A], [B], [C])                Sum of values
AVG([A], [B])                     Average of values
+ - * / ^ %                       Arithmetic operators
= <> > < >= <=                    Comparison operators
&& || !                           Logical operators
??                                Null coalesce operator
ISFUTURE()                        True if period is in the future
MIN(), MAX(), FIRST(), LAST()     Range aggregation
IFNULL(a, b), COALESCE(a, b, ..) Null handling
DIV(a, b), SUMPRODUCT(r1, r2)    Safe division, sum of products
```

**Safe division pattern:**
```
IF [Denominator] <> 0 THEN [Numerator] / [Denominator] ELSE 0
```

Use `cap model formula-validation validate --json` to verify formula syntax before saving a calculation.

---

## Workflow: Creating a New Input

1. Identify the discipline, unit, and aggregation pattern with `cap masterdata disciplines list --json`, `cap masterdata units list --json`, and the aggregation tables above
2. Copy the Input payload template
3. Fill in name, description, discipline ID, unit ID, aggregation IDs
4. Run `cat <<'EOF' | cap model inputs create --json`
5. Verify with `cap model inputs get <new-id> --json`

## Workflow: Creating a New Calculation

1. Identify which metrics to reference in the formula — use `cap model metrics list --json` to find the metric IDs for the target tenant
2. Choose the calculation phase (Before/After Aggregations)
3. Choose aggregation methods (or None for recomputed ratios) — see Common Aggregation Patterns above
4. Copy the Calculation payload template
5. Fill in name, description, discipline ID, unit ID, and formula using metric ID references
6. Run `cat <<'EOF' | cap model calculations create --json`
7. Capture the returned ID from the `--json` response — needed if other calculations reference this one
8. Verify with `cap model calculations get <new-id> --json`

**Important notes:**
- **Formula validation** happens at create time — the API checks syntax, verifies referenced metric IDs exist, and detects circular dependencies
- **Duplicate names** are allowed — the system won't prevent creating a second calculation with the same name
- **Automatic engine pickup** — a `ComputableItemChange` message is published after create, so the calculation engine processes new calculations without manual intervention
- **Computed values visibility** — to query computed values for the new calculation, ensure a Report Template exists whose filters include it. Report templates use filter criteria to determine which metrics appear; new calculations matching those filters show up automatically.

---

## Batch Calculation Creation

When creating calculations that **reference other new calculations**, you need to create them in dependency order because formulas use metric IDs.

### Strategy

1. **Batch 1:** Create calculations that only reference existing metrics
2. **Batch 2:** Capture Batch 1 IDs from `--json` responses, use them in Batch 2 formulas
3. **Batch 3:** Capture Batch 2 IDs, use them in Batch 3 formulas

### Example

```bash
# Batch 1: Cost Per Unit (references existing Filler Throughput and Total Cost)
cat <<'EOF' | cap model calculations create --json
{
  "id": "<empty-id>",
  "name": "Cost Per Unit",
  "discipline": { "id": "<cost-discipline-id>" },
  "unitOfMeasure": { "id": "<usd-unit-id>" },
  "dataInterval": { "id": 0, "name": "Day" },
  "precision": 2,
  "calculationPhase": { "id": 1, "name": "After Aggregations" },
  "orgStructureAggregationMethod": { "id": 3, "name": "None" },
  "timePeriodAggregationMethod": { "id": 0, "name": "None" },
  "formula": "IF [<id>] <> 0 THEN [<id>] / [<id>] ELSE 0",
  "translatedNames": [],
  "translatedDescriptions": [],
  "attributeValues": []
}
EOF
# Returns: { "success": true, "id": "<id>" }
# ^^^^^ Capture this ID for Batch 2

# Batch 2: Margin Per Unit (references Batch 1's Cost Per Unit)
cat <<'EOF' | cap model calculations create --json
{
  ...
  "formula": "[<id>] - [<id>]",
  ...
}
EOF
```

### Tips

- Always use `--json` output to capture the new ID from each create
- Verify each batch with `cap model calculations list` before proceeding
- For large batches, consider scripting the ID capture and substitution

---

## Related Documentation

- [Commands Reference](./commands.md) -- Quick command lookup
- [Glossary](./glossary.md) -- Term definitions
- [Create Metric Recipe](../recipes/configuration/create-metric.md) -- Interactive wizard
