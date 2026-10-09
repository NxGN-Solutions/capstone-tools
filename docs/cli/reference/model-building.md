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

For widget templates, set this at both the widget default and data-item override when a Dynamic widget collapses a selected range into one displayed value. Use None only for deliberate time-series output or explicit single-period offsets. See [Widget Time Period Aggregation](./widget-time-aggregation.md).

### Calculation Phases

| ID | Name | Description |
|----|------|-------------|
| 0 | Before Aggregations | Compute at each node for the calculation's own data interval, then aggregate. Use for conversions, emissions from activity data, per-reading counts |
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
| 6 | TextBlock | Metric-aware dashboard text with title, subtitle, description, and footnote |

### Widget Sizes

| ID | Name | Description |
|----|------|-------------|
| 0 | 25% | Quarter width (4 across) |
| 1 | 33% | Third width (3 across) |
| 2 | 50% | Half width (2 across) |
| 3 | 66% | Two-thirds width |
| 4 | 75% | Three-quarters width |
| 5 | 100% | Full width |

**Recommended defaults:** InfoCards at 50%, daily XY Charts at 100%, site-comparison XY Charts at 50%, PieCharts at 33%, TextBlocks at 100% for narrative/status text, Tables at 100% for dense grids, AI Summary at 100%. See [Build Dashboard](../recipes/configuration/build-dashboard.md#widget-sizing-guidelines) for rationale.

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

### Table Widget Enums

| Field | Values | Description |
|-------|--------|-------------|
| `dataGrouping` | `0 None`, `1 OrgNode`, `2 Discipline`, `3 Framework` | Controls rendered table row grouping. `Framework` requires Dynamic metric selection. |
| `orgNodeRowSelectionMode` | `0 Children`, `1 Descendants` | Applies when the grouping chain contains OrgNode. |
| `narrativeSelectionMode` | `0 Dynamic`, `1 Static` | Dynamic resolves narratives from Narrative Scope; Static renders explicit `narratives[]`. |

Table metric scope uses `metricTypeFilters`, `metricDisciplineFilters`, `metricFrameworkFilters`, `metricAttributeFilters`, `disciplineAttributeFilters`, and `frameworkAttributeFilters` when `metricSelectionMode` is Dynamic. The legacy/report-template-compatible `disciplineNodeAttributeFilters` and `frameworkNodeAttributeFilters` fields are also accepted by the shared DTO surface. Static Table templates use explicit `dataItems[]` unless metric filters provide the report-template fallback.

Table custom columns are controlled with `showMetricValue`, `metricPropertyColumns` (`MetricProperty` EnumDTO list; widget default `[]`), and `metricAttributeTypeIds`. Dynamic Narrative Scope uses `narrativeDisciplineFilters`, `narrativeFrameworkFilters`, `narrativeMetricAttributeFilters`, `narrativeDisciplineNodeAttributeFilters`, `narrativeFrameworkNodeAttributeFilters`, and `narrativeAttributeFilters`.

Table `styleConfiguration` uses bounded semantic HTML table slots (`panel`, `header`, `title`, `description`, `gridHeader`, `rowLabels`, `metadataCells`, `valueCells`, `totalCells`, `missingValueCells`, `emptyState`, `errorState`, `footnote`, `paginator`) plus `rowLabelHeader`, `zebraStripeColor`, `columnOverrides[]`, `rowOverrides[]`, and `categoricalColorTags[]`. Metric colour bands live on the metric (`bands`) or a static `dataItems[].bands[]` override, not on table style.

Do not author `includedDataTypes` or `orgNodeTemplateId` for new Table widgets. They are legacy compatibility/adapter fields. Arbitrary `columns[]`, expression columns, and `OrgNodeAttribute` row grouping are deferred beyond the current Table authoring contract.

---

## Common Aggregation Patterns

| Metric Type | Org Agg | Time Agg | Rationale |
|-------------|---------|----------|-----------|
| **Throughput/Volume** | Sum (0) | Sum (1) | Units add up across locations and time |
| **Cost/Revenue ($)** | Sum (0) | Sum (1) | Financial totals are additive |
| **OEE/Rates (%)** | Average (1) | Average (2) | Rates should average, not sum |
| **Assumptions** | Roll Down (5) | Average (2) | Interval at which the value changes (default Year), set once at the root, cascades to children |
| **Targets** | Roll Down (5) | Average (2) | Interval at which the target changes (default Year), set at the root, inherited by children |
| **Per-unit ratios** | None (3) | None (0) | Recomputed at each level from aggregated inputs |
| **Headcount/Balance** | Last Value (3*) | Last Value (3) | Point-in-time snapshots |

> *Note: "Last Value" is a Time Period Aggregation method (ID 3), not an Org Agg method. For org structure, use None (3) or Sum (0) depending on context.

---

## Aggregation Execution Semantics

### Org and Time Composition

Metric aggregation has two independent dimensions:

- **Org structure aggregation** flows up the org-node hierarchy from child nodes to parent nodes.
- **Time period aggregation** combines values across the selected reporting periods for the requested data interval.

When both are involved, report reads compose them for the requested view. For example, a company-level quarterly report for a monthly Sum/Sum metric rolls site values up through the org tree and combines the selected months into the quarter result.

### Cross-Interval Time Behavior

The calculation engine materializes values at every tenant-enabled interval.
The metric's time-period aggregation method controls both collapsing finer
source periods into a coarser period and expanding a coarser source period into
contained finer periods:

| Time method | Finer source -> coarser target | Coarser source -> finer target |
|-------------|---------------------------------|--------------------------------|
| None | Native interval only | No value |
| Sum | Add source values | Divide the source value evenly across contained target periods |
| Average | Average source values | Repeat the source value unchanged in each contained target period |
| Last Value | Use the most recent source value | Put the value in the final contained target period; earlier periods are zero |
| Min | Use the minimum source value | Repeat the source value unchanged |
| Max | Use the maximum source value | Repeat the source value unchanged |

Use `Average` for an assumption or rate captured at a coarser interval but
applicable unchanged throughout that interval. Use `Sum` for a coarser-period
total that should be apportioned evenly. For example, a ZAR/hour rate captured
once per year uses `dataInterval = Year` and
`timePeriodAggregationMethod = Average`, making the same hourly rate available
to monthly calculations without duplicate input capture.

### Engine Evaluation Order

The calculation engine runs two passes over every interval enabled on the tenant:

**Pass 1: Before Aggregations**

1. Input values are brought to the interval being processed with each input's time period aggregation method: finer periods collapse, coarser periods expand (see the table above).
2. Roll Down inputs are copied down the org tree. No other input is rolled up yet: each org node sees only its own captured values.
3. Each Before Aggregations calculation is evaluated at every org node, but **only for periods of its own data interval**. A Month calculation is evaluated for months only.

**Pass 2: After Aggregations**

4. Inputs and Before Aggregations results are brought to each interval with their own time period aggregation method, and rolled up the org tree with their own org structure aggregation method. A Before Aggregations result is collapsed or expanded from its data interval.
5. Each After Aggregations calculation is evaluated at every org node, for every period of every enabled interval.

What this means for formulas:

- A formula is evaluated at one interval at a time. Every reference in it, and every period offset such as `|-1|`, resolves at that same interval. A formula cannot compare a quarter value with a month value. At a month evaluation a quarterly input has already been expanded to months: repeated for Average, divided by three for Sum.
- **A Before Aggregations calculation's data interval decides where it is evaluated.** Set it to the cadence of the values the formula reads. A Month calculation over quarterly readings (Average time aggregation) sees each reading repeated in all three months, so a count or Sum over it is tripled at quarter and year. Readings with Sum time aggregation are divided by three instead.
- **An After Aggregations calculation is evaluated at every interval**, so its data interval is only the default display interval.
- Before Aggregations calculations can reference inputs and other Before Aggregations calculations **with the same data interval**. Other calculation references are not rejected on save, but they evaluate as blank.
- **Below its data interval, a Before Aggregations result is only an expansion.** A quarterly count (Sum) shows a third of the quarter in each month; a yearly Last Value result shows in the last month and quarter of the year only, and zero before that. Keep a Before Aggregations calculation at a coarser interval only when its cadence requires it (for example counts over quarterly readings), and put the interval in its name (`… | Quarter | Total Number`) so widget authors know it isn't interval-agnostic.
- **Rolling windows: use `|MONTHS(-12)+1:0|`, not `|-11:0|`.** A numeric offset counts periods of the evaluated interval, so `|-11:0|` is 12 quarters at quarter. `MONTHS(n)` converts months to periods but rounds down to whole periods (`MONTHS(-11)` is 4 quarters back, giving 15 months). `MONTHS(-12)+1` starts exactly 12 months back at month (`-11`), quarter (`-3`) and year (`0`), so an After Aggregations rolling 12-month rate such as `DIV(SUM([TRI]|MONTHS(-12)+1:0|) * [Basis], SUM([Hours]|MONTHS(-12)+1:0|), null)` is right at every interval.
- **Sums or ratios of fixed values (provisions, balances, rates) belong After Aggregations.** Average inputs captured yearly repeat at every quarter and month, so an After Aggregations `SUM()` over them is correct at every interval. As a Before Aggregations Year calculation with Last Value it would read zero for most quarters and months.

### Calculation Phases

| Phase | Use When |
|-------|----------|
| Before Aggregations | The value must be computed at each org node and period before roll-up, then summed or averaged: conversions with a factor, emissions from activity data, per-reading flags and counts. |
| After Aggregations | The value is a ratio, percentage, intensity, target achievement or period-over-period change. These must be recomputed from aggregated operands at every node and interval. |

For ratios such as emissions intensity or cost per unit, prefer After Aggregations with `orgStructureAggregationMethod = None` and `timePeriodAggregationMethod = None`, so the ratio is recomputed from aggregated numerator and denominator values. A Before Aggregations ratio or change is wrong above its own interval and node: its quarter value is the sum or average of monthly ratios, not the quarter's ratio.

## Modelling Rules

Wrong numbers almost always come from interval, phase, aggregation or formula design, not from the engine. Work through the evaluation order above before suspecting a defect.

### Org nodes are reporting units, not measurement points

The engine evaluates and aggregates every metric at every org node for every enabled period. Exclusions don't change that.
- Add an org node only where the model, or most of it, is reported, such as an operation or a site with its own reporting.
- Monitoring stations, boreholes, sample points, meters and equipment that report only a few metrics stay **metrics on their org node**, for example `Input | Monitoring | Air Quality | Air Station A10 | PM10 | µg/m³`.
- Roll them up with calculations: site average, maximum, exceedance counts and compliance.
- Never propose turning such points into org nodes.

To make dashboards work at any org node, name the measurement metrics consistently across org nodes, using the same parameter names and units. Put the limits on inputs valued per org node with Roll Down, so the same calculation and widget serve every node.

### Intervals

- An input's data interval is its capture cadence. **Changing an input's data interval deletes all of its captured values**, for every period and including locked values. Export the values first (`cap data input-values download-excel`) and reload them after the change. Never toggle an interval to inspect data.
- Set a Before Aggregations calculation's data interval to the cadence of its inputs **when you create it**. If you change it later, wait for settlement and re-check the values at month, quarter and year. If they don't match a recount, delete the calculation and create it again with the right interval.

### Fixed values: targets, limits, factors, exchange rates

Model a value that holds unchanged for a period as an input with:

| Setting | Value | Why |
|---------|-------|-----|
| Data interval | The interval at which the value changes. Default to Year: the fewest values to capture | One value per period of change. A value that changes monthly (an exchange rate) or quarterly uses that interval |
| Time period aggregation | Average (2) | Repeated unchanged in every finer period, and averaged over coarser ones |
| Org structure aggregation | Roll Down (5) | Captured once at the root and inherited by every node |

Capture it at the root org node. An org node that needs a different value captures its own, and Roll Down fills only the nodes without one. Do not use Sum, which would divide the value across the finer periods, or Last Value, which puts it in the last finer period and 0 in the others.

### Missing data stays blank

- Use `DIV(numerator, denominator, null)` for ratios and percentages so a period with no data shows nothing. `IF [D] <> 0 THEN [N] / [D] ELSE 0` and `DIV(n, d)` turn a missing or zero denominator into a real-looking zero.
- Use `COALESCE([Optional], 0)` only where a missing value genuinely means zero, such as an optional discharge line in a water balance.
- If the first `DIV` argument starts with a period-selected reference, wrap it in parentheses: `DIV(([A]|0| - [A]|-1|) * 100, [A]|-1|, null)`.

### Counting readings against a limit

- **Limits:** model them as fixed values (above) and colour each reading with a band bound to the limit metric (`lowerBoundMetric`). That needs no per-station calculation.
- **Count of exceedances** (Before Aggregations, Sum, data interval = the readings' cadence): `SUM([R1] * 0 + (IF [R1] > [Limit] THEN 1 ELSE 0), [R2] * 0 + (IF [R2] > [Limit] THEN 1 ELSE 0), ...)`. The `[R] * 0 +` term makes a station with no reading contribute nothing rather than 0.
- **Results tested** (same settings): `SUM([R1] * 0 + 1, [R2] * 0 + 1, ...)`.
- **Compliance %** (After Aggregations): `(1 - DIV(SUM([Count A], [Count B]), SUM([Tested A], [Tested B]), null)) * 100`.

At month, a quarterly count shows thirds (one exceedance in a quarter reads 0.33 per month). That is the expected Sum expansion.

### Targets

- **Target:** an input, set up as a fixed value (above) with `allowForecastedData = true`.
- **Achievement % calculation** (After Aggregations), where 100 means on target in both directions:
  - Higher is better: `DIV([Actual] * 100, [Target], null)`.
  - Lower is better: `DIV([Target] * 100, [Actual], null)`.
- **Colouring the actual:** add a band on the actual bound to the target metric (`lowerBoundMetric`).

### Forecasts

`FORECAST`, `HOLT` and `HOLTWINTERS` return a one-step-ahead forecast at each interval from the previous periods of that interval.
- `HOLT` and `HOLTWINTERS` follow the trend. A partly captured latest period drives them sharply down, even below zero.
- Use `FORECAST` (level only) until recent periods are complete.
- `HOLTWINTERS` needs at least two full seasons of history, for example 24 months for `period = 12`.

### Exclusions are for reporting only

Org-node exclusions hide a metric in capture and reports. The engine still calculates and aggregates every metric for every node and enabled period, so the value space has no gaps. Never use an exclusion to change what a calculation adds up.

### Unit conversions

- **Both directions:** define every conversion in both directions, as two conversions on the two units.
- **Fixed factor:** use one for physical units.
- **Factor metric:** use one for rates that change over time, such as exchange rates.
  - Capture the rate monthly at the root (Roll Down, Average).
  - Reference an After Aggregations calculation such as `DIV(1, [ZAR per USD], null)` for the inverse direction.
- **Saving in an override unit:** a value saved with `"unitOfMeasure"` set to the org-node override unit is converted to the metric unit only when a conversion exists **from the metric unit to the override unit**. Without it the save is rejected (`InvalidUnitOfMeasureConversion`).
- **Values captured before a conversion existed:** values saved without a unit (or in the metric unit) are stored as typed. If they were really amounts in the override unit, re-save them with `"unitOfMeasure"` set to the override unit once the conversion exists, so they are stored in the metric unit.
- **Different dimensions:** don't use a unit conversion between them (mass and volume, for example). Use a calculation with the density instead.

### Verify

After `cap data recalculation wait` reports `settled`, check one value at month, quarter and year against a manual recount from the inputs, for example with `cap reporting computed-values query`. Collapse and expand must match the aggregation rules above.

### Dense Matrix and Build Order

Report output depends on the template, metric set, org scope, time periods, and available input values. For repeatable tenant builds, treat the required data as a metric x org-node x period matrix:

1. Create org nodes, disciplines/frameworks, units, inputs, calculations, and templates.
2. Resolve selectable periods with `cap data time-periods list --data-interval <type> --json`; use the returned `startDate` values when saving input values.
3. Load leaf input values for every metric, org node, and period needed by the report or by formulas the report depends on.
4. Wait for recalculation settlement before verifying report output.

Sparse data can be valid, but it should be intentional. If a parent report row is missing or empty, first check whether the leaf cells and formula dependencies exist for the selected period names.

### Recalculation Settlement

Model changes and input-value saves can update model state asynchronously. JSON mutation responses that include `modelVersion` also include a `recommendedRecalculationWaitCommand` when the CLI can derive one.

Use these commands before reading reports:

```bash
cap data recalculation status --json
cap data recalculation wait <target-model-version> --json
```

`settled = true` means the calculation version has reached the target model version, calculation succeeded, and results were persisted. Report reads before settlement can show stale or missing computed values.

### Reporting Fetch Surfaces

Use report-template surfaces to verify saved report shape and parent rollups:

```bash
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval month \
  --periods "Jan 2026" \
  --json

cap reporting computed-values download-excel \
  --template <report-template-id> \
  --data-interval month \
  --periods "Jan 2026" \
  --output report.xlsx \
  --json
```

`reporting computed-values query` is an inline direct metric query. It is useful for targeted diagnostics, but it is not the full report-template/export lens and should not be used as the only proof that a saved report template renders correctly.

Use `cap reporting computed-values audit --metrics <metric-id> --data-interval <type> --periods <period> --include-metric-details --strict --json` to diagnose stale model state, missing metric details, no rows, or all-null values.

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
--discipline-nodes <id> --json` before capture workflows. Lock and governed edit
workflows use `cap data data-lock lock|unlock` and
the unified `cap data change-requests` surface.

### Create Input

```json
{
  "id": "<empty-id>",
  "name": "<metric name>",
  "friendlyName": "<optional business label>",
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
  "bands": [
    { "backgroundColor": "danger-subtle", "foregroundColor": "text-primary" },
    { "lowerBoundValue": 90, "backgroundColor": "warning-subtle", "foregroundColor": "text-primary" },
    { "lowerBoundValue": 95, "backgroundColor": "success-subtle", "foregroundColor": "text-primary" }
  ],
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
  "attributeValues": []
}
```

`bands` is optional. Omit the field (or send `null`) to leave stored bands unchanged on update. Send `[]` to clear them. A ladder list has an unbounded first band and later bands use exactly one of `lowerBoundValue` or `lowerBoundMetric`. An interval list may bound every row, including the first, with optional `upperBoundValue` / `upperBoundMetric`; list the in-spec interval first and leave outermost tails unbounded. Colours are registry tokens only. On an input, or an input's org-node override, `"requireComment": true` on a band asks for a comment when a value is captured in it; calculations and calculation overrides reject it. A `lowerBoundMetric`/`upperBoundMetric` must be another metric, may use any data interval, and may use another unit when a conversion exists in either direction: the bound is that metric's computed value for the same org node and period.

On an input, a band may also set `"autoValidate": true` (default `false`). When the input requires validation and a saved value lands in that band, the value is stored `Approved` instead of `ValidationRequired`, and the change history records an Approve entry by System naming the band and its limits. If a limit cannot be resolved (for example a referenced metric has no value for that org node and period yet), the value needs validation as usual. The check runs only when a value is saved; editing bands later does not re-evaluate stored values. `autoValidate` is for input bands and input overrides: calculation saves, calculation overrides and template band overrides reject it. In the Excel Bands column, a trailing ` auto` on an entry sets it.

```json
"bands": [
  { "backgroundColor": "danger-subtle", "foregroundColor": "text-primary" },
  { "lowerBoundValue": 90, "backgroundColor": "success-subtle", "foregroundColor": "text-primary", "autoValidate": true }
]
```

A `lowerBoundMetric` / `upperBoundMetric` must be another metric (not the metric itself) and may use any data interval. Its unit of measure may differ from the owner's when a unit conversion exists between the two units in either direction; every bound is converted to the metric's unit at evaluation and shown in the node's display unit. With no conversion, the save rejects the band. When a conversion uses a factor metric that has no value for a period, that point is left unpainted. The same rule applies to template rows, widget data items and org-node overrides.

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
  "friendlyName": "<optional business label>",
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
  "bands": [
    { "backgroundColor": "danger-subtle", "foregroundColor": "text-primary" },
    { "lowerBoundValue": 90, "backgroundColor": "warning-subtle", "foregroundColor": "text-primary" },
    { "lowerBoundValue": 95, "backgroundColor": "success-subtle", "foregroundColor": "text-primary" }
  ],
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
  "formula": "DIV([Numerator], [Denominator], null)",
  "attributeValues": []
}
```

`bands` is optional on calculations with the same omit/`null`/empty-list rules as inputs. Calculation bands reject `autoValidate`.

`friendlyName` is optional. Use it for user-facing labels while keeping `name`
stable for formulas, imports, and model identity. Translation arrays are edit-UI
metadata returned by entity GET endpoints; CLI create/save payloads should use
the current-language `name`, `friendlyName`, and `description` fields only.

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

### Org-Node Overrides (Inputs and Calculations)

`model input-overrides` and `model calculation-overrides` hold one row per metric and org node. A row overrides the unit of measure, precision, colour bands and, for inputs, `requireValidation` / `requireDataCapture` for that node and the nodes below it.

```json
{
  "id": "<empty-id>",
  "metric": { "id": "<metric-id>" },
  "orgNode": { "id": "<org-node-id>" },
  "unitOfMeasure": { "id": "<unit-id>" },
  "precision": 1,
  "requireValidation": true,
  "requireDataCapture": false,
  "bands": [
    { "backgroundColor": "danger-subtle", "foregroundColor": "text-primary" },
    { "lowerBoundValue": 90, "backgroundColor": "success-subtle", "foregroundColor": "text-primary" }
  ]
}
```

```bash
cat <<'EOF' | cap model input-overrides save --json
{ ... payload ... }
EOF
```

Calculation overrides use the same shape without `requireValidation` / `requireDataCapture`.

**`bands` on an override:**

- `null` (or omitted) inherits. The node uses the bands of the nearest org node at or above it whose override sets bands; rows with `null` bands are skipped. With none, the metric's `bands` apply.
- `[]` turns formatting off for the node and its descendants, until a descendant override sets a list.
- A list replaces the inherited bands, with the same structure rules as metric bands.
- Constant bounds (`lowerBoundValue` / `upperBoundValue`) are typed in the override's own `unitOfMeasure`. Values are always stored in the metric's unit, so each bound is converted to the metric unit and then shown in the node's display unit. An override with constant bounds whose unit has no conversion from the metric unit is rejected.
- A spreadsheet-template row or widget data item that sets its own `bands` still wins for colour. Inheriting from the template ("use metric default") means the node's effective bands: the override's, else the metric's. Template rows do not change `requireComment` or `autoValidate`.
- An input override may set `requireComment` and `autoValidate` on a band. They follow the list: `[]` turns both off, and `null` inherits. Constant bounds stay in the override unit and are converted to the metric unit before the check. Calculation overrides reject both flags.

`get --json` returns `bands` (absent or `null` = inherit); the table output shows `Bands: Inherited`, `None (formatting off)` or the band count. `list` uses the same three labels in a Bands column (a discipline group row shows `-`). `inherited <metric-id> <org-node-id>` prints the unit, precision, flags (inputs) and bands that node would use before its own row. The override Excel workbook has a `Bands` column in the same compact text format as metric workbooks (`danger-subtle; 90:warning-subtle; [Target]:success/white`): blank inherits, `none` turns formatting off, and a workbook without the column leaves stored bands unchanged. An input override cell accepts ` auto` and ` !comment`; a calculation override rejects both.

Capture, report and Table widget data responses carry `orgNodeBands` (metric id → org node id → bands) next to the metric-level `bands`, only for nodes whose effective bands differ; an empty list there means no formatting at that node.

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

**Units:** values are stored in the metric definition unit. `list`, `get` and the capture grid show each org node's **effective unit**: the node's unit override (or the nearest ancestor's), converted with the unit's conversion factor (a fixed factor, or the factor metric's value for that node and period).

- Send `unitOfMeasure.id` on every row to say which unit the number is in. A value in the override unit is converted to the metric unit, and a value in the metric unit is stored as sent.
- Omitting `unitOfMeasure` means the metric unit, **except** on an org node that has a unit override. There the save is rejected with `MissingUnitOfMeasure`, because a bare number copied from the screen would be stored off by the conversion factor.
- A unit with no conversion from the metric unit is rejected with `InvalidUnitOfMeasureConversion`.

```json
{
  "id": "<empty-id>",
  "metric": { "id": "<metric-id>" },
  "orgNode": { "id": "<override-org-node-id>" },
  "timePeriodType": { "id": 2, "name": "Month" },
  "startDate": "2026-08-01",
  "value": 1500,
  "unitOfMeasure": { "id": "<us-dollars-unit-id>" }
}
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

### Clear a Value

There is no separate delete command. To remove a captured value, save the same
business key (`metric` + `orgNode` + `timePeriodType` + `startDate`) with
`"value": null` — the same save the web app sends when a cell is emptied:

```bash
cat <<'EOF' | cap data input-values save --json
{
  "inputValues": [
    {
      "id": "<empty-id>",
      "value": null,
      "metric": { "id": "<metric-id>" },
      "orgNode": { "id": "<org-node-id>" },
      "timePeriodType": { "id": 4, "name": "Year" },
      "startDate": "2026-10-01T00:00:00Z"
    }
  ]
}
EOF
```

Null rows can be mixed with value rows in one batch. Clearing follows the same
rules as any save: a value in a locked period is rejected until the period is
unlocked. Wait for recalculation (`cap data recalculation wait <version>`)
before checking computed values.

### Selectable Period Validation and Diagnostics

By default, `data input-values save` submits rows after resolving each metric's
data interval and warning about non-selectable reporting starts. Add
`--strict-selectable-periods` when a seed load must reject rows whose
`startDate` is not one of the tenant's selectable reporting period starts.

Use `cap data time-periods list --data-interval <type> --json` to get valid
`startDate` values. If a date is rejected, run:

```bash
cap data time-periods diagnose <yyyy-MM-dd> --data-interval <type> --json
```

The diagnostic response includes `reportingSelectable`, `selectableRange`,
`selectablePeriod`, `nearestSelectablePeriods`, `mismatchReason`, and
`seedValidity`. Extending the tenant's configured reporting period range is the
administrative action when the requested date is outside `selectableRange`.

JSON save output groups identical period diagnostics by default. Add
`--include-period-diagnostics` to include the grouped diagnostic array, or
`--full-period-diagnostics` when you need one diagnostic per input row. The full
flag implies `--include-period-diagnostics`.

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

### Conversions

Each item in `conversions` has a `destination` unit and **exactly one** of:

- `conversionFactor` — a fixed number (destination units per one source unit: displayed = stored × factor), or
- `conversionFactorMetric` — the metric (input or calculation) whose value per org node, interval and period supplies the factor, also in destination units per source unit.

Set the unused field to `null` or leave it out. Each destination may appear only once per unit.

Fixed factor (on a `kWh` unit — 1 kWh = 0.001 MWh):

```json
"conversions": [
  {
    "id": "<empty-id>",
    "destination": { "name": "MWh" },
    "conversionFactor": 0.001
  }
]
```

Metric-referenced factor (on a `ZAR` unit — the "USD per ZAR" metric holds USD per 1 ZAR, e.g. 0.054; for a rate quoted as ZAR per USD, reference a calculation such as `1 / [ZAR per USD]`):

```json
"conversions": [
  {
    "id": "<empty-id>",
    "destination": { "name": "USD" },
    "conversionFactorMetric": { "name": "USD per ZAR" }
  }
]
```

`destination` and `conversionFactorMetric` are resolved by `id` or, when the id is empty, by `name`. `cap masterdata units get --json` shows them flattened to names (`"destination": "USD"`, `"conversionFactorMetric": "USD per ZAR"`).

> **Note:** Converted values are blank for periods where the factor metric has no value. A save in the converted unit is rejected until that period's rate has been calculated. `cap data input-values upload-excel` saves the rest of the workbook instead and lists each value without a rate as a warning: wait for `cap data recalculation wait` to report settled, then upload the same file again.

Removing a conversion that colour bands depend on is rejected, and the error names the dependent bands. A band depends on a conversion when it references a metric in a different unit, or when an org-node override in that unit has constant bounds.

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
DIV(a, b[, fallback])            Safe division; the fallback defaults to 0
SUMPRODUCT(r1, r2)               Sum of products
MEDIAN(...), MEAN(...)            Median; mean (alias of AVG)
STDEV(...), STDEVP(...)           Standard deviation: sample; population
VARIANCE(...), VARIANCEP(...)     Variance: sample; population
FORECAST(alpha, r)                Single exponential smoothing forecast
HOLT(alpha, beta, r)              Holt linear-trend forecast
HOLTWINTERS(a, b, g, period, r)   Holt-Winters additive forecast
HOLTWINTERSM(a, b, g, period, r)  Holt-Winters multiplicative forecast
```

**Safe division pattern:**
```
DIV([Numerator], [Denominator], null)
```

A missing or zero denominator stays no-data. Use the two-argument
`DIV([Numerator], [Denominator])` or `IF [Denominator] <> 0 THEN ... ELSE 0`
only when a numeric zero is the intended answer; in a reported KPI a missing or zero denominator then makes a
period without data look like a real zero.

Use `cap model formula-validation validate <calculation-name> --formula '<formula>' --json` to verify formula syntax and circular-dependency behavior before saving a named calculation. Omit `<calculation-name>` for syntax-only/dependency validation; the CLI sends an internal collision-resistant sentinel name so the validation request cannot be mistaken for a real metric.

### Formula Parser Provenance

The formula language is parser-backed. Runtime parsing and core formula behavior come from the `Formula.Parser` v1.4.3 package, implemented with F#/FParsec. This bundled CLI reference is a curated subset for common Capstone usage, not a full grammar specification. If a construct is not listed here, validate it before use rather than assuming the document is exhaustive.

Capstone registers these additional time functions:

```
NOW, YEARSTART, YEAREND, MONTHSTART, MONTHEND, DATEOFFSET,
MONTH, YEAR, DAYS, WEEKS, MONTHS, QUARTERS, YEARS, ISFUTURE
```

The authoritative pre-save check is the API validation endpoint exposed by the CLI:

```bash
cap model formula-validation validate <calculation-name> --formula '<formula>' --json
cap model formula-validation validate --formula '<formula>' --json
```

You can also pipe the formula through stdin:

```bash
echo '[A] + [B]' | cap model formula-validation validate Total --json
echo '[A] + [B]' | cap model formula-validation validate --json
```

---

## Workflow: Creating a New Input

1. Identify the discipline, unit, and aggregation pattern with `cap masterdata disciplines list --json`, `cap masterdata units list --json`, and the aggregation tables above
2. Copy the Input payload template
3. Fill in name, optional friendlyName, description, discipline ID, unit ID, aggregation IDs
4. Run `cat <<'EOF' | cap model inputs create --json`
5. Verify with `cap model inputs get <new-id> --json`

## Workflow: Creating a New Calculation

1. Identify which metrics to reference in the formula — use `cap model metrics list --json` to find the metric IDs for the target tenant
2. Choose the calculation phase (Before/After Aggregations)
3. Choose aggregation methods (or None for recomputed ratios) — see Common Aggregation Patterns above
4. Copy the Calculation payload template
5. Fill in name, optional friendlyName, description, discipline ID, unit ID, and formula using metric ID references
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
  "friendlyName": "Cost/unit",
  "discipline": { "id": "<cost-discipline-id>" },
  "unitOfMeasure": { "id": "<usd-unit-id>" },
  "dataInterval": { "id": 0, "name": "Day" },
  "precision": 2,
  "calculationPhase": { "id": 1, "name": "After Aggregations" },
  "orgStructureAggregationMethod": { "id": 3, "name": "None" },
  "timePeriodAggregationMethod": { "id": 0, "name": "None" },
  "formula": "DIV([<id>], [<id>], null)",
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
