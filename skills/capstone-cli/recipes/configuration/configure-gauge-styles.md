# Recipe: Configure Gauge Styles

> Build a styled Gauge widget template with the CLI: one metric's value at one
> site on a dial of its ranges. Use this when a dashboard needs to show where a
> KPI sits against its good, watch and bad ranges, with the range name and the
> distance to the nearest range edge under the value.

## When to Use

- "Show TRIFR as a gauge against its safety ranges"
- "Make the gauge's scale run from 0 to 4 in steps of 0.5"
- "Draw the metric's ranges in the colours set on them, not the strong shade"
- "The top range has no width on the dial: give it more room"

## Step 1: Inspect the Live Contract

Start with the binary's sample and schema. They are the source for the current
field names and validation bounds.

```bash
cap templates widget-templates sample --widget-type gauge --json
cap templates widget-templates schema --widget-type gauge --json
cap reporting widgets gauge --help
```

| Command | JSON Root |
|---------|-----------|
| `cap templates widget-templates sample --widget-type gauge --json` | `.payload.widgetTemplate` contains the save-ready sample body. |
| `cap templates widget-templates schema --widget-type gauge --json` | `.fieldLookups[]` holds the gauge shapes and range colour values; `.optionalFields[]` holds the scale and style bounds. |
| `cap templates widget-templates get <id> --json` | `.widgetTemplate` contains the editable full DTO. |
| `cap reporting widgets gauge <id> ... --json` | Top-level render contract: `value`, `arcs`, `edges`, `distance`, `offScale`, the scale settings and `styleConfiguration`. |

The gauge shapes have no standalone lookup endpoint. They are in
`schema.fieldLookups` for `widgetTemplate.gaugeShape`: Half Dial (0), Arc Dial (1)
and Solid (2).

## Step 2: One Metric, One Site

A gauge shows **one metric** at **one site**:

- `dataItems` holds exactly one row, the metric the gauge shows. A second row is
  refused with `GaugeOneMetricOnly`.
- The gauge reads at one org node. `reporting widgets gauge` and
  `widget-templates preview` send the org nodes you name; the server refuses more
  than one with a 400, "A gauge shows one site". On a dashboard the gauge reads
  the dashboard's site.
- A gauge has nothing to rank or filter, so it takes no `valueSelectionConfig`.
- The row's `timePeriodAggregationMethod` sets how its periods combine (use Last
  Value or Average for a ratio such as TRIFR, Sum for an additive amount).

## Step 3: Set the Shape and the Scale

| Field | Meaning |
|-------|---------|
| `gaugeShape` | Half Dial (a half circle with a needle), Arc Dial (a three-quarter arc with a needle) or Solid (a three-quarter arc filled to the value, no needle). |
| `gaugeStartAtZero` | Default `true`. An automatic low end starts at 0. A negative value or range edge still widens the scale below 0, so the needle is never hidden. A set minimum and a log scale win over it. |
| `gaugeMinimum`, `gaugeMaximum` | `null` for Auto. A set end holds exactly. A value past it pins the needle at that end and the gauge says "Off the scale" (`offScale: true` in the render). |
| `gaugeMajorStep` | The distance between two labels, in the metric's unit. Above 0, and not on a log scale. The automatic ends round out to whole steps. |
| `gaugeMinorTicksPerStep` | 0 to 9 unlabelled marks between two labels when a step is set; `null` draws one. |
| `gaugeLogarithmic` | A log scale. It can't start at zero or hold a minimum or a range edge at or below 0; a value at or below 0 pins the needle at the low end. |
| `gaugeShowRangeName` | Default `true`: show the name of the range the value reads in. |
| `gaugeShowDistance` | Default `true`: show the distance to the nearest range edge, for example "0.03 above 2.00". |

These scale fields mean what an XY chart's value-axis fields mean. An automatic
end takes the value **and every range edge**, as an XY chart's value axis takes
its lines and zones, so every range is on the dial.

Scale errors name the field: `GaugeScaleRangeInvalid` (the maximum is not above
the minimum), `GaugeLogScaleNotPositive`, `GaugeStepInvalid`, `GaugeStepLogScale`
and `GaugeMinorTicksInvalid`.

## Step 4: Choose the Ranges and Their Colours

The arcs are the value's ranges:

- **Own ranges.** Set `dataItems[0].bands` to give the gauge its own ranges. They
  are drawn in the colours chosen on them, always.
- **The metric's (or the site's) ranges.** Leave `dataItems[0].bands` as `null`.
  How they are drawn is the style's `rangeColours`:
  - `strong` (the default when unset): each range in the strong shade of its
    meaning, so a pale red range is drawn red;
  - `asSet`: each range in the colours set on it, as table cells paint them.

The range name under the value keeps the range's own colours either way. With
conditional formatting off (`showConditionalFormatting: false`, or the viewer's
switch when `allowConditionalFormattingToggle` lets them), the arcs are grey.

## Step 5: Leave Room Past an Open Outer Range

An outer range is open when it has no end, such as "2.00 and above". When the
scale end beside it is automatic, the dial leaves room past the last edge, so
that range has width. `rangeRoomPercent` sets the room as a percent of the
span: 0 to 50, **10 by default**. 0 ends the scale at the edge exactly, as an XY
chart does. A set minimum or maximum gets no room: it holds.

## Step 6: Style the Parts

`styleConfiguration` has these slots:

| Slot | Properties |
|------|------------|
| `panel` | The panel's surface, as a pie's panel. |
| `title`, `description`, `value`, `unit`, `rangeName`, `distance`, `footnote`, `stateMessages` | Text parts, as a pie's text boxes. |
| `arc` | `backgroundColor` (the track, and the plain arc when there are no ranges) and `opacity`. |
| `needle` | `backgroundColor` only. |

Beside the slots sit `rangeColours` (`strong` or `asSet`) and `rangeRoomPercent`
(0 to 50). Colours are registry tokens from
`cap meta lookups get color-tokens` or `#RGB` / `#RRGGBB` / `#RRGGBBAA` hex.

```json
{
  "styleConfiguration": {
    "panel": { "backgroundColor": "surface", "padding": 16, "gap": 8 },
    "title": { "foregroundColor": "text-primary", "fontWeight": "Bold", "fontSize": 18 },
    "value": { "foregroundColor": "text-primary", "fontWeight": "Bold", "fontSize": 32 },
    "rangeName": { "foregroundColor": "text-secondary", "fontSize": 14 },
    "arc": { "backgroundColor": "border" },
    "needle": { "backgroundColor": "text-primary" },
    "rangeColours": "strong",
    "rangeRoomPercent": 10
  }
}
```

Errors: `GaugeStyleRangeColoursInvalid` for a value other than `strong` or
`asSet`, `GaugeStyleRangeRoomInvalid` outside 0 to 50.

## Step 7: Preview, Save and Verify

```bash
cap templates widget-templates sample --widget-type gauge --save-ready > gauge-widget.json
# Replace the placeholder metric and discipline ids, then preview at one site:
cap templates widget-templates preview --file gauge-widget.json \
  --org-nodes <org-node-id> --periods "Mar 2026"
cap templates widget-templates create --file gauge-widget.json --json
cap reporting widgets gauge <widget-template-id> \
  --org-nodes <org-node-id> --periods "Mar 2026" --json
```

Without `--json`, the preview and `reporting widgets gauge` print the value
(marked "off the scale" when it is), the range name, the distance, the shape,
the scale settings and the ranges' arcs. With `--json`, read:

| Field | Meaning |
|-------|---------|
| `value` | `label`, the formatted `value` (or the state's word: no data, not applicable, restricted), `rawValue`, `rangeName` and the range's colours. |
| `arcs[]` | Each range, `from` to `to`, in the order drawn; a `null` end runs to the scale's end. `color` is the token or hex to draw. |
| `edges[]` | The ranges' finite edges, low to high. |
| `distance` | `value`, `edge`, `direction` (Above or Below) and `text`. |
| `offScale` | `true` when the value is past a set end, or at or below 0 on a log scale. |

## Unsupported Payloads

Gauge styling rejects raw CSS, HTML, JavaScript, unknown slots and unknown
properties. The needle takes a colour only. Use only the schema and sample
fields.
