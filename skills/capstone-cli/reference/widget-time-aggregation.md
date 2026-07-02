# Widget Time Period Aggregation

Time period aggregation controls how widget values are combined when the selected date range contains multiple periods.

## When It Matters

Set `timePeriodAggregationMethod` when a widget needs one displayed value from multiple selected periods:

- Dynamic InfoCards
- Dynamic Pie/Donut slices
- Table cells that span more than one period
- XY charts configured as aggregated summaries

Use `{ "id": 0, "name": "None" }` only when the widget should keep periods separate, such as an XY time series with one point per period or a static period-offset card where each data item resolves a single period.

## Lookup

```bash
cap templates lookups get timePeriodAggregationMethod --json
cap meta lookups get timePeriodAggregationMethod --domain templates --json
```

| ID | Name | Use Case |
|----|------|----------|
| 0 | None | Keep each period separate |
| 1 | Sum | Additive totals such as revenue, cost, hours, consumption, counts |
| 2 | Average | Rates, percentages, utilization, averages |
| 3 | Last Value | Snapshots such as status, fleet size, headcount, balance |
| 4 | Min | Lowest value across the selected range |
| 5 | Max | Highest value across the selected range |

## Where To Set It

Set it at the widget level as the default:

```json
"timePeriodAggregationMethod": { "id": 1, "name": "Sum" }
```

Set it on a data item to override the widget default for that metric:

```json
{
  "metric": { "id": "<metric-id>", "name": "Utilization %" },
  "timePeriodAggregationMethod": { "id": 2, "name": "Average" }
}
```

## Widget Defaults

| Widget | Recommended Default |
|--------|---------------------|
| InfoCard | Sum for additive metrics, Average for rates, Last Value for snapshots |
| Pie/Donut | Sum for additive slices; Average only for rate-like slices |
| XY time series | None |
| XY aggregated summary | Sum, Average, or Last Value by metric role |
| Table | Sum for additive value cells, Average for rates, None for explicit single-period offsets |

## Common Mistakes

- Do not leave Dynamic InfoCards or Pie/Donut widgets with a null aggregation method. The render path warns and may return no values.
- Do not sum rates or percentages. Use Average unless the metric has a stronger domain-specific rule.
- Do not use None for single-value summaries that span multiple periods. It is only correct when periods must remain separate or the value field targets exactly one period.
- Keep widget-level and data-item-level methods aligned unless the widget intentionally mixes metric roles.

## Related Commands

```bash
cap templates widget-templates sample --widget-type info --json
cap templates widget-templates schema --widget-type info --json
cap templates widget-templates sample --widget-type pie --json
cap templates widget-templates schema --widget-type pie --json
cap templates widget-templates sample --widget-type xy --json
```
