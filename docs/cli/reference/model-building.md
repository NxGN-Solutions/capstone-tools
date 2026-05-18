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
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "<narrative name>",
  "description": "<description>",
  "reference": "<optional-reference>",
  "discipline": {
    "id": "<discipline-guid>"
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
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "<metric name>",
  "description": "<description>",
  "discipline": {
    "id": "<discipline-guid>"
  },
  "unitOfMeasure": {
    "id": "<unit-guid>"
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
      "id": "00000000-0000-0000-0000-000000000000",
      "name": "Manual Capture",
      "dataSource": {
        "id": "<manual-capture-datasource-guid>"
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
{ "success": true, "id": "<new-guid>" }
```

### Create Calculation

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "<calculation name>",
  "description": "<description>",
  "discipline": {
    "id": "<discipline-guid>"
  },
  "unitOfMeasure": {
    "id": "<unit-guid>"
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

> **Formula references:** Formulas can reference metrics by name (`[Metric Name]`) or by GUID (`[4f20e84a-9b10-4d99-abe1-afe9f8f578f5]`). GUIDs are recommended for CLI usage — they're unambiguous, rename-safe, and what the API stores internally. Use `cap model calculations get <id> --json` to see existing formulas with their GUID references.

**Command:**
```bash
cat <<'EOF' | cap model calculations create --json
{ ... payload ... }
EOF
```

### Update Existing (Save)

Same structure as create, but set `id` to the existing metric's GUID. Use `save` instead of `create`:

```bash
cat <<'EOF' | cap model inputs save --json
{ "id": "<existing-guid>", ... rest of payload ... }
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
  --org-node <org-node-guid> \
  --input <metric-guid> \
  --period-type week \
  --start-date 2026-02-09 \
  --json
```

**`save`** uses JSON input (batch):
```bash
cat payload.json | cap data input-values save --json
```

> In practice, `save` with zero GUIDs handles both creation and updates. Use `create` only when you need to get the existing value ID for a specific cell.

## Save Input Values

> **Important:** The save endpoint uses **business-key upsert** semantics. Each input value is uniquely identified by its business key: `metric` + `orgNode` + `timePeriodType` + `startDate`. The `id` field can be the zero GUID (`00000000-0000-0000-0000-000000000000`) for **both** new values and updates — the API resolves existing records by business key automatically. You only need existing GUIDs if you want to be explicit, but it's not required. This makes batch saves idempotent — running the same payload again overwrites with the same values, never creating duplicates.

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
      "id": "00000000-0000-0000-0000-000000000000",
      "value": 1500,
      "metric": { "id": "<metric-guid>" },
      "orgNode": { "id": "<org-node-guid>" },
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
      "id": "00000000-0000-0000-0000-000000000000",
      "value": 0.08,
      "metric": { "id": "<metric-a-guid>" },
      "orgNode": { "id": "<site-1-guid>" },
      "timePeriodType": { "id": 2, "name": "Month" },
      "startDate": "2026-01-01T00:00:00Z"
    },
    {
      "id": "00000000-0000-0000-0000-000000000000",
      "value": 0.09,
      "metric": { "id": "<metric-a-guid>" },
      "orgNode": { "id": "<site-2-guid>" },
      "timePeriodType": { "id": 2, "name": "Month" },
      "startDate": "2026-01-01T00:00:00Z"
    },
    {
      "id": "<existing-value-guid>",
      "value": 0.10,
      "metric": { "id": "<metric-a-guid>" },
      "orgNode": { "id": "<site-3-guid>" },
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

2. **Build payload** — use the existing `id` for updates, or the zero GUID for new entries. Include the full business key on every entry.

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
      "id": "00000000-0000-0000-0000-000000000000",
      "value": 300,
      "metric": { "id": "<filler-planned-rate-guid>" },
      "orgNode": { "id": "<filler-node-guid>" },
      "timePeriodType": { "id": 1, "name": "Week" },
      "startDate": "2026-02-09T00:00:00Z"
    },
    {
      "id": "00000000-0000-0000-0000-000000000000",
      "value": 0.60,
      "metric": { "id": "<cola-plan-pct-guid>" },
      "orgNode": { "id": "<site-guid>" },
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
      "id": "<grid-row-guid>",
      "name": "Cola Plan %",
      "path": "Cola Plan %->Enterprise B->Site1",
      "isDataRow": true,
      "unitOfMeasure": { "name": "%", "id": "<uom-guid>" },
      "precision": 2,
      "groupingKey": "<org-node-guid>",
      "timePeriodColumns": [
        {
          "inputValue": {
            "id": "<value-guid-or-zero>",
            "value": 0.6,
            "validationStatus": 2,
            "isLocked": false,
            "userCanCapture": true
          }
        },
        {
          "inputValue": {
            "id": "00000000-0000-0000-0000-000000000000",
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
- A zero GUID in `inputValue.id` means no value has been entered for that cell
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

## ProveIt Tenant Reference Data

> These IDs are specific to the ProveIt tenant. Other tenants will have different IDs.

### Discipline IDs

| Discipline | ID | Parent |
|------------|----|--------|
| Finance | `019c27be-2d4b-78cc-8297-e0860d332330` | -- |
| Cost | `019c27be-2d4c-7962-9c0c-5cc6ef9d4603` | Finance |
| Cost Assumptions | `019c27be-2d4c-76d8-aacc-ab09591ec69c` | Finance |
| Revenue | `019c27be-2d4c-76b5-9e2d-148f7afbf548` | Finance |
| Revenue Assumptions | `019c27be-2d4c-7851-b657-f25e4e32cbd5` | Finance |
| Production | `019c27be-2d4c-71c7-a264-6f5e35191268` | -- |
| Metrics | `019c27be-2d4c-73be-b621-67d08778d435` | Production |
| Plan | `019c27be-2d4c-71d5-8610-9ae72ea95e38` | Production |
| Raw Metrics | `019c27be-2d4c-74e8-a958-130359d6b69c` | Production |

### Unit of Measure IDs

| Unit | Symbol | Position | ID |
|------|--------|----------|----|
| % | % | Suffix | `019c27bd-d8b6-7c62-8eed-c585d9c1dd5d` |
| Count | | Suffix | `019c27bd-d8b6-7886-8d4f-2036938c9475` |
| Factor | | Suffix | `019c27bd-d8b6-7f2b-b09d-f1f373a03d41` |
| Litres | l | Suffix | `019c27bd-d8b6-778b-a687-e95342ff8e8d` |
| Units | | Suffix | `019c27bd-d8b7-72c5-b780-cecbe151995f` |
| Units/Minute | | Suffix | `019c27bd-d8b7-7c8e-ad4f-56684e813cae` |
| USD | $ | Prefix | `019c27bd-d8b7-7e81-8a5a-28d628b1db13` |

### Data Source IDs

| Data Source | ID |
|-------------|-----|
| Manual Capture | `019c274c-1e1d-72fd-a560-1068f484e311` |
| MQTT TimescaleDB | `019c274c-1e25-7578-94b4-eca88d4470f7` |

### Enterprise & Site IDs

| Node | ID |
|------|----|
| Enterprise B | `019c27be-1601-779a-a224-14e4088ad8b6` |
| Site1 | `019c27be-1602-7339-9f88-70478f2bd0fc` |
| Site2 | `019c27be-1606-7659-b886-3c6dc5d1ea06` |
| Site3 | `019c27be-1609-7341-a7f6-4f10fdd45a2a` |

---

## Payload Templates: Unit of Measure

### Create/Save Unit of Measure

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
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

# Update (set id to existing GUID)
echo '{ "id": "<existing-guid>", ... }' | cap masterdata units save --json
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

See [Formula Language](../../features/metrics/formula-language.md) for the full reference.

---

## Workflow: Creating a New Input

1. Identify the discipline, unit, and aggregation pattern from the tables above
2. Copy the Input payload template
3. Fill in name, description, discipline ID, unit ID, aggregation IDs
4. Run `cat <<'EOF' | cap model inputs create --json`
5. Verify with `cap model inputs get <new-id> --json`

## Workflow: Creating a New Calculation

1. Identify which metrics to reference in the formula — use `cap model metrics list --json` to find GUIDs
2. Choose the calculation phase (Before/After Aggregations) — see [Calculation Phases](../../features/metrics/calculations.md#calculation-phase)
3. Choose aggregation methods (or None for recomputed ratios) — see Common Aggregation Patterns above
4. Copy the Calculation payload template
5. Fill in name, description, discipline ID, unit ID, formula using GUID references
6. Run `cat <<'EOF' | cap model calculations create --json`
7. Capture the returned GUID from the `--json` response — needed if other calculations reference this one
8. Verify with `cap model calculations get <new-id> --json`

**Important notes:**
- **Formula validation** happens at create time — the API checks syntax, verifies referenced metric GUIDs exist, and detects circular dependencies
- **Duplicate names** are allowed — the system won't prevent creating a second calculation with the same name
- **Automatic engine pickup** — a `ComputableItemChange` message is published after create, so the calculation engine processes new calculations without manual intervention
- **Computed values visibility** — to query computed values for the new calculation, ensure a Report Template exists whose filters (discipline, metric type) include it. Report templates use filter criteria to determine which metrics appear; new calculations matching those filters show up automatically. See [Report Templates](../../features/spreadsheet-templates/report-templates.md)

---

## Batch Calculation Creation

When creating calculations that **reference other new calculations**, you need to create them in dependency order because formulas use GUIDs.

### Strategy

1. **Batch 1:** Create calculations that only reference existing metrics (GUIDs already known)
2. **Batch 2:** Capture Batch 1 GUIDs from `--json` responses, use them in Batch 2 formulas
3. **Batch 3:** Capture Batch 2 GUIDs, use them in Batch 3 formulas

### Example

```bash
# Batch 1: Cost Per Unit (references existing Filler Throughput and Total Cost)
cat <<'EOF' | cap model calculations create --json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "Cost Per Unit",
  "discipline": { "id": "<cost-discipline-guid>" },
  "unitOfMeasure": { "id": "<usd-unit-guid>" },
  "dataInterval": { "id": 0, "name": "Day" },
  "precision": 2,
  "calculationPhase": { "id": 1, "name": "After Aggregations" },
  "orgStructureAggregationMethod": { "id": 3, "name": "None" },
  "timePeriodAggregationMethod": { "id": 0, "name": "None" },
  "formula": "IF [3b46e59b-7315-43bb-a667-9fbfd55f8b6c] <> 0 THEN [4b9d3b4e-327a-48de-a60b-f5fa1cf6749d] / [3b46e59b-7315-43bb-a667-9fbfd55f8b6c] ELSE 0",
  "translatedNames": [],
  "translatedDescriptions": [],
  "attributeValues": []
}
EOF
# Returns: { "success": true, "id": "8fa4d124-6807-4814-a70b-94e0a7947fc2" }
# ^^^^^ Capture this GUID for Batch 2

# Batch 2: Margin Per Unit (references Batch 1's Cost Per Unit)
cat <<'EOF' | cap model calculations create --json
{
  ...
  "formula": "[7b8d66f5-24fb-4353-83d0-aefe5373b8a8] - [8fa4d124-6807-4814-a70b-94e0a7947fc2]",
  ...
}
EOF
```

### Tips

- Always use `--json` output to capture the new GUID from each create
- Verify each batch with `cap model calculations list` before proceeding
- For large batches, consider scripting the GUID capture and substitution

---

## Related Documentation

- [Commands Reference](./commands.md) -- Quick command lookup
- [Glossary](./glossary.md) -- Term definitions
- [Create Metric Recipe](../recipes/configuration/create-metric.md) -- Interactive wizard
- [Calculations](../../features/metrics/calculations.md) -- Calculation phases and behavior
- [Formula Language](../../features/metrics/formula-language.md) -- Formula syntax
- [Aggregation Methods](../../features/metrics/aggregation-methods.md) -- Detailed aggregation rules
