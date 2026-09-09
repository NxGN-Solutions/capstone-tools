# Recipe: Configure Info Card Styles

> Build styled Info Card widget templates with the CLI. Use this when a dashboard
> needs KPI cards, period-comparison cards, alert/exceedance cards, tinted
> metric cards, left-accent cards, or full-bleed showcase cards.
>
> **Visual standard:** the normative card variants (`KPI-Accent`, `KPI-Quiet`,
> `KPI-Alert`, `KPI-Comparison`) and the type ramp live in the
> [Dashboard Design System](../../reference/dashboard-design-system.md) — start
> there; this recipe covers the mechanics and additional showcase styles.

## When to Use

- "Style this Info Card so it matches the dashboard mockup"
- "Create an alert card for exceedances"
- "Create blue, teal, purple, or warning KPI cards"
- "Show current vs previous period with a styled trend"
- "Make the card content run edge to edge inside the widget"
- "Use the CLI to demonstrate Info Card styling flexibility"

## Required Context

Before starting, know:

- [ ] The Info Card widget template ID, or enough metric/discipline context to create one
- [ ] The primary metric ID
- [ ] The discipline ID
- [ ] The data interval (`day`, `week`, `month`, `quarter`, or `year`)
- [ ] Whether the card is a single KPI, period comparison, alert, tinted metric, left-accent metric, or full-bleed showcase card
- [ ] For trends, the calculation metric ID used by the `#trend[...]` token

If any IDs are missing, discover them first:

```bash
cap model metrics list --json
cap masterdata disciplines list --json
cap templates widget-templates list --json
```

## Step-by-Step

### Step 1: Inspect the Live CLI Contract

Always start with the binary's own sample and schema. These commands are
discoverable without repository docs and include the current style fields.

```bash
cap templates widget-templates sample --widget-type info --json
cap templates widget-templates schema --widget-type info --json
```

`--widget-type info-card` is also accepted; `info` is the compact canonical
variant emitted by the sample/schema contract.

The schema lists `styleConfiguration` fields and constraints. Key fields are:

| Field | Use |
|-------|-----|
| `panel` | The card surface and internal layout |
| `title`, `value`, `unit`, `label`, `description`, `footnote` | Text slots |
| `separators` | Lines between values or sections |
| `trend` | Style for resolved `#trend[...]` text |
| `stateMessages` | No-data, unavailable, or warning messages |
| `panel.padding` | Internal spacing inside the card boundary, `0..64` |
| `panel.gap` | Internal spacing between card sections, `0..64` |
| `panel.accentSide` / `panel.accentWidth` | Panel accent edge only |

`padding`, `gap`, `accentSide`, and `accentWidth` are panel-only fields.
Borders render only when `borderColor` and positive `borderWidth` are both set.
Panel accents render only when `accentColor`, `accentSide`, and positive
`accentWidth` are all set.

### Step 2: Create or Export the Template JSON

For a new card, start from the sample and replace placeholders. The sample
command returns an envelope; extract the `payload` object before using it as a
create file.

```bash
cap templates widget-templates sample --widget-type info --json > info-card-sample.json
jq '.payload' info-card-sample.json > info-card.json
```

If `jq` is not available, copy the `payload` object from
`info-card-sample.json` into `info-card.json`.

For an existing card, export the template:

```bash
cap templates widget-templates get <widget-template-id> --json > info-card.json
```

Edit `info-card.json` and set `widgetTemplate.styleConfiguration` to one of the
style fragments below. If the exported JSON is not wrapped in `widgetTemplate`,
the CLI still accepts the inner widget object for `create` and `save`.

### Step 3: Choose a Style Pattern

Use these fragments as the value for `styleConfiguration`.

#### Quiet KPI Card

Use for plain operational KPIs where the value is the focus.

```json
{
  "panel": {
    "backgroundColor": "surface",
    "foregroundColor": "text-primary",
    "borderColor": "border",
    "borderWidth": 1,
    "borderRadius": 8,
    "padding": 20,
    "gap": 8,
    "fontFamily": "theme"
  },
  "title": {
    "foregroundColor": "text-primary",
    "fontWeight": "Semibold",
    "fontSize": 16
  },
  "value": {
    "foregroundColor": "text-primary",
    "fontWeight": "Bold",
    "fontSize": 34
  },
  "unit": {
    "foregroundColor": "text-secondary",
    "fontWeight": "Medium",
    "fontSize": 14
  },
  "label": {
    "foregroundColor": "text-secondary",
    "fontSize": 14
  }
}
```

#### Period Comparison Card

Use for current vs previous period cards. This pattern keeps the card neutral
and lets the trend direction carry the color.

```json
{
  "panel": {
    "backgroundColor": "surface",
    "borderColor": "border",
    "borderWidth": 1,
    "borderRadius": 8,
    "padding": 20,
    "gap": 12,
    "fontFamily": "theme"
  },
  "title": {
    "foregroundColor": "text-primary",
    "fontWeight": "Semibold",
    "fontSize": 16
  },
  "value": {
    "foregroundColor": "text-primary",
    "fontWeight": "Bold",
    "fontSize": 32
  },
  "label": {
    "foregroundColor": "text-secondary",
    "fontSize": 14
  },
  "separators": {
    "borderColor": "border",
    "borderWidth": 1
  },
  "trend": {
    "foregroundColor": "text-secondary",
    "fontWeight": "Medium",
    "fontSize": 14,
    "showLabel": true,
    "up": { "color": "danger", "icon": "arrow-up" },
    "down": { "color": "success", "icon": "arrow-down" },
    "flat": { "color": "neutral", "icon": "equals" },
    "unknown": { "color": "neutral", "icon": "none" }
  }
}
```

For air-quality and exceedance cards, an upward value is usually bad, so `up`
uses red and `down` uses green. For revenue or production cards where higher is
good, swap those two colors.

#### Alert or Exceedance Card

Use for counts, exceedances, threshold breaches, or values that need a warning
visual treatment.

```json
{
  "panel": {
    "backgroundColor": "surface-muted",
    "borderColor": "warning",
    "borderWidth": 1,
    "borderRadius": 8,
    "accentColor": "warning",
    "accentSide": "Bottom",
    "accentWidth": 6,
    "padding": 18,
    "gap": 10,
    "fontFamily": "theme"
  },
  "title": {
    "foregroundColor": "warning",
    "fontWeight": "Bold",
    "fontSize": 18
  },
  "value": {
    "foregroundColor": "warning",
    "fontWeight": "Bold",
    "fontSize": 34
  },
  "label": {
    "foregroundColor": "text-secondary",
    "fontSize": 14
  },
  "separators": {
    "borderColor": "warning",
    "borderWidth": 1
  },
  "trend": {
    "fontWeight": "Medium",
    "showLabel": true,
    "up": { "color": "danger", "icon": "caret-up" },
    "down": { "color": "success", "icon": "caret-down" },
    "flat": { "color": "neutral", "icon": "equals" },
    "unknown": { "color": "neutral", "icon": "none" }
  }
}
```

#### Tinted Rate Card

Use for rate metrics such as deposition rate, average concentration, intensity,
or other cards that should stand apart without looking like an alert.

```json
{
  "panel": {
    "backgroundColor": "surface-canvas",
    "borderColor": "info",
    "borderWidth": 1,
    "borderRadius": 8,
    "accentColor": "info",
    "accentSide": "Bottom",
    "accentWidth": 5,
    "padding": 18,
    "gap": 10,
    "fontFamily": "theme"
  },
  "title": {
    "foregroundColor": "info",
    "fontWeight": "Bold",
    "fontSize": 18
  },
  "value": {
    "foregroundColor": "info",
    "fontWeight": "Bold",
    "fontSize": 36
  },
  "unit": {
    "foregroundColor": "text-secondary",
    "fontWeight": "Medium",
    "fontSize": 14
  },
  "separators": {
    "borderColor": "info",
    "borderWidth": 1
  },
  "trend": {
    "fontWeight": "Medium",
    "showLabel": true,
    "up": { "color": "danger", "icon": "arrow-up" },
    "down": { "color": "info", "icon": "arrow-down" },
    "flat": { "color": "neutral", "icon": "equals" },
    "unknown": { "color": "neutral", "icon": "none" }
  }
}
```

#### Left Accent Metric Card

Use for pollutant or metric families where color identifies the metric type,
such as blue NO2 and purple SO2 cards.

```json
{
  "panel": {
    "backgroundColor": "surface-muted",
    "borderColor": "border",
    "borderWidth": 1,
    "borderRadius": 8,
    "accentColor": "primary",
    "accentSide": "Left",
    "accentWidth": 5,
    "padding": 18,
    "gap": 10,
    "fontFamily": "theme"
  },
  "title": {
    "foregroundColor": "primary",
    "fontWeight": "Bold",
    "fontSize": 18
  },
  "value": {
    "foregroundColor": "primary",
    "fontWeight": "Bold",
    "fontSize": 34
  },
  "unit": {
    "foregroundColor": "text-secondary",
    "fontWeight": "Medium"
  },
  "separators": {
    "borderColor": "border",
    "borderWidth": 1
  },
  "trend": {
    "fontWeight": "Medium",
    "showLabel": true,
    "up": { "color": "danger", "icon": "arrow-up" },
    "down": { "color": "success", "icon": "arrow-down" },
    "flat": { "color": "neutral", "icon": "equals" },
    "unknown": { "color": "neutral", "icon": "none" }
  }
}
```

For a blue variant, replace the purple values with:

```json
{
  "panel.backgroundColor": "surface-canvas",
  "panel.borderColor": "info",
  "panel.accentColor": "info",
  "title.foregroundColor": "info",
  "value.foregroundColor": "info",
  "separators.borderColor": "info"
}
```

Do not paste this dotted-key object into the CLI payload. It is a compact patch
description; apply the values to the matching nested fields in the previous
fragment.

#### Full-Bleed Showcase Card

Use when the card styling itself must control the full interior of the widget.
This is the right pattern for accent frames, zero-padding cards, or dark
showcase panels.

This showcase is intentionally outside the standard design-system palette and
uses raw hex values to demonstrate custom-brand styling. For normal dashboard
work, prefer registry tokens from `cap meta lookups get color-tokens`.

```json
{
  "panel": {
    "backgroundColor": "#250909",
    "foregroundColor": "#f8fafc",
    "borderColor": "#ef0909",
    "borderWidth": 5,
    "borderRadius": 8,
    "accentColor": "#0008ff",
    "accentSide": "All",
    "accentWidth": 5,
    "padding": 0,
    "gap": 8,
    "fontFamily": "theme"
  },
  "title": {
    "foregroundColor": "#f8fafc",
    "fontWeight": "Bold",
    "fontSize": 20
  },
  "value": {
    "foregroundColor": "#ffffff",
    "fontWeight": "Bold",
    "fontSize": 36
  },
  "label": {
    "foregroundColor": "#cbd5e1",
    "fontSize": 14
  },
  "description": {
    "foregroundColor": "#e2e8f0"
  },
  "footnote": {
    "foregroundColor": "#cbd5e1",
    "fontWeight": "Medium"
  },
  "trend": {
    "fontWeight": "Medium",
    "showLabel": false,
    "up": { "color": "#f87171", "icon": "arrow-up" },
    "down": { "color": "#4ade80", "icon": "arrow-down" },
    "flat": { "color": "#e2e8f0", "icon": "equals" },
    "unknown": { "color": "#cbd5e1", "icon": "none" }
  }
}
```

`panel.padding: 0` means no internal Info Card padding. Any space outside the
card surface is dashboard layout padding or dashboard card chrome, not
`styleConfiguration`.

### Step 4: Use Period Comparison Data Items When Needed

For current vs previous cards, use static period offsets and period tokens:

```json
{
  "dataRangeMode": { "id": 1, "name": "Static" },
  "dataInterval": { "id": 3, "name": "Quarter" },
  "footnote": "#trend[<change-calculation-metric-id>] since @[period:-1]",
  "dataItems": [
    {
      "metric": { "id": "<metric-id>", "name": "Metric to display" },
      "name": "@[period]",
      "dataPeriodStart": 0,
      "dataPeriodEnd": 0,
      "sortOrder": 1,
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 0, "name": "None" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null
    },
    {
      "metric": { "id": "<metric-id>", "name": "Metric to display" },
      "name": "@[period:-1]",
      "dataPeriodStart": -1,
      "dataPeriodEnd": -1,
      "sortOrder": 2,
      "metricPartitioningMode": { "id": 0, "name": "None" },
      "timePeriodAggregationMethod": { "id": 0, "name": "None" },
      "partitioningRankMode": { "id": 0, "name": "None" },
      "partitioningRankLimit": null
    }
  ]
}
```

Raw CLI/API JSON uses stored metric IDs in formula-like tokens. The widget
editor and Widget Template Excel accept metric names and translate them to IDs.

### Step 5: Save and Verify

Create a new widget:

```bash
cap templates widget-templates create --file info-card.json --json
```

Update an existing widget:

```bash
cap templates widget-templates save --file info-card.json --json
```

Verify the render contract with the typed Info Card command:

```bash
cap reporting widgets info-card <widget-template-id> \
  --org-nodes "<org-node-id>" \
  --data-interval quarter \
  --periods "Q3 FY 25" \
  --json
```

The JSON response includes resolved text content, data items, warnings, and the
style configuration returned to the dashboard renderer.

## Troubleshooting

| Problem | Check |
|---------|-------|
| Style fields are rejected | Run `cap templates widget-templates schema --widget-type info --json` and use only listed fields |
| Border is missing | Set both `borderColor` and `borderWidth` greater than `0` |
| Accent edge is missing | Set `panel.accentColor`, `panel.accentSide`, and `panel.accentWidth` greater than `0` |
| Padding `0` still leaves outside spacing | That spacing is dashboard layout padding or dashboard chrome, not Info Card internal padding |
| Trend shows the wrong business color | Trend direction follows numeric sign only; swap `trend.up.color` and `trend.down.color` for metrics where higher is good |
| Trend token shows a GUID in raw JSON | Raw CLI JSON stores metric IDs; verify rendered output with `cap reporting widgets info-card ... --json` |
| Dynamic card returns no value | Set a non-`None` `timePeriodAggregationMethod` for dynamic single-value cards |

## See Also

- [Create Widget Template](./create-widget-template.md)
- [Build Dashboard](./build-dashboard.md)
- [Command Quick Reference](../../reference/commands.md)
- [Template Selection Guide](../../reference/template-selection.md)
- [Widget Time Aggregation](../../reference/widget-time-aggregation.md)
