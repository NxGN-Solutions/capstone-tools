# Recipe: Configure Pie Chart Styles

> Build styled Pie and Donut widget templates with the CLI. Use this when a
> dashboard needs custom Pie surfaces, labels, legends, donut center styling,
> data-item presentation, number formatting, or value selection alongside style.

## When to Use

- "Style this donut chart to match the dashboard mockup"
- "Format Pie legend values in thousands"
- "Style a Donut center metric value"
- "Keep a Pie to the Top 5 values and style the output"
- "Show the same styled Pie through browser, CLI, and MCP"

## Step 1: Inspect the Live Contract

Always start with the binary's sample and schema. They are the source for the
current field names and validation bounds.

```bash
cap templates widget-templates sample --widget-type pie --json
cap templates widget-templates schema --widget-type pie --json
cap reporting widgets pie-chart --help
```

`--widget-type pie-chart` is also accepted. The compact `pie` variant is what
the CLI samples emit.

Use these roots when scripting:

| Command | JSON Root |
|---------|-----------|
| `cap templates widget-templates list --json` | `.data[]` contains compact grid rows. |
| `cap templates widget-templates sample --widget-type pie --json` | `.payload.widgetTemplate` contains the save-ready sample body. |
| `cap templates widget-templates schema --widget-type pie --json` | `.fieldLookups[]` contains enum values and `.optionalFields[]` contains style/format constraints. |
| `cap templates widget-templates get <id> --json` | `.widgetTemplate` contains the editable full DTO. |
| `cap templates widget-templates save --file pie-widget.json --json` | Top-level acknowledgement with `success`, `id`, child data-item counts, and `warnings`; run `get` for the full updated DTO. |
| `cap reporting widgets pie-chart <id> ... --json` | Top-level render contract with `id`, `title`, `pieChartWidgetType`, `styleConfiguration`, `center`, `legend`, `labelDisplay`, and `dataItems[]`. |

Pie-specific modes that are not exposed as standalone lookup endpoints are still
discoverable in `schema.fieldLookups`: `widgetTemplate.centerMode`,
`widgetTemplate.legendValueMode`, and `widgetTemplate.sliceLabelMode`.

## Step 2: Keep Styling Separate From Value Selection

Pie styling fields are visual:

| Field | Purpose |
|-------|---------|
| `styleConfiguration` | Chart-wide ("all slices") base for every element category: panel, title, chart area, slices, slice borders, slice labels, ticks, tooltip, legend, legend markers/labels/values, donut center, and state messages |
| `centerNumberFormat` | Donut Metric Value center formatting |
| `dataItems[].presentation` | Per-slice (per-metric) overrides of the chart-wide base, keyed by the same element categories (`slice`, `sliceBorders`, `sliceLabels`, `tooltip`, `legendMarker`, `legendLabel`, `legendValue`). Slice fill lives at `presentation.slice.backgroundColor`. |
| `dataItems[].numberFormat` | Source data-item value formatting |

`valueSelectionConfig` is not visual. It controls filtering, ordering, Top/Bottom
ranking, and diagnostics. Do not put style fields inside
`valueSelectionConfig`; the API, Excel import, CLI, MCP, and browser validator
reject that shape.

The CLI can use the same styling primitives as the API and web editor: Pie vs
Donut type, center mode and center number format, legend value mode, slice label
mode, chart-wide style slots, per-slice presentation overrides, explicit slice
colours, per-slice number formats, and value selection. Generate sparse style
objects from the sample/schema allowlist: include the supported properties you
intend to set and omit unsupported properties instead of sending placeholder
nulls.

### How styling cascades

Each element category has two tiers. `styleConfiguration.<slot>` sets the
chart-wide ("all slices") base; `dataItems[].presentation.<slot>` overrides it
for one slice. Resolution order per property is:

```
dataItems[].presentation.<slot>.<prop>   (per slice)
  → styleConfiguration.<slot>.<prop>     (all slices)
    → theme / auto-palette               (default)
```

Omit a property at any tier to inherit the tier below it.

Two details are important for fills and legend markers:

- **`slices`** can set a chart-wide base fill, and the per-slice fill override is
  `dataItems[].presentation.slice.backgroundColor`. The chart-wide
  `slices.backgroundColor` is an optional
  **"mute" base** that overrides the auto-palette for *every* slice. Set it, then
  give one data item a `presentation.slice.backgroundColor`, to grey everything
  and spotlight one slice:

  ```json
  {
    "styleConfiguration": { "slices": { "backgroundColor": "neutral" } },
    "dataItems": [
      { "presentation": { "slice": { "backgroundColor": "primary" } } },
      {},
      {}
    ]
  }
  ```

- **`legendMarkers`** accepts `borderColor`, `borderWidth`, `borderRadius`, and
  `opacity` only. A marker's colour always follows its slice, so there is no
  chart-wide marker fill and no `presentation.legendMarker.backgroundColor`.
  Setting either is rejected by the API, Excel import, CLI, MCP, and browser
  validator.

## Step 3: Configure a Styled Donut

Use the sample command and replace IDs with real metric and discipline IDs:

```bash
cap templates widget-templates sample --widget-type pie --json > pie-sample.json
jq '.payload' pie-sample.json > pie-widget.json
```

Then edit `pie-widget.json`. This fragment shows the styling fields to keep:

```json
{
  "styleConfiguration": {
    "panel": {
      "backgroundColor": "surface",
      "borderColor": "border",
      "borderWidth": 1,
      "borderRadius": 8,
      "padding": 20
    },
    "title": {
      "foregroundColor": "text-primary",
      "fontWeight": "Semibold",
      "fontSize": 18
    },
    "chartArea": {
      "backgroundColor": "surface",
      "padding": 8
    },
    "slices": {
      "opacity": 0.9
    },
    "sliceBorders": {
      "borderColor": "surface",
      "borderWidth": 2
    },
    "ticks": {
      "borderColor": "border",
      "borderWidth": 1,
      "opacity": 0.75
    },
    "legendValues": {
      "foregroundColor": "text-secondary",
      "fontWeight": "Semibold",
      "fontSize": 13
    },
    "donutCenter": {
      "foregroundColor": "text-primary",
      "fontWeight": "Bold",
      "fontSize": 28
    },
    "stateMessages": {
      "foregroundColor": "warning"
    }
  },
  "centerNumberFormat": {
    "precision": 2,
    "magnitude": "Millions"
  },
  "dataItems": [
    {
      "presentation": {
        "slice": {
          "backgroundColor": "#4a90d9"
        },
        "legendValue": {
          "fontWeight": "Semibold",
          "foregroundColor": "text-secondary"
        },
        "tooltip": {
          "fontSize": 12,
          "foregroundColor": "text-primary"
        },
        "sliceLabels": {
          "fontSize": 12,
          "foregroundColor": "text-primary"
        }
      },
      "numberFormat": {
        "precision": 1,
        "magnitude": "Thousands"
      }
    }
  ]
}
```

`presentation.slice.backgroundColor` is the per-item slice and legend-marker
fill. Legacy Excel workbooks with a `Colour` column still upload successfully;
that import-only column maps into `presentation.slice.backgroundColor`. New JSON
and Excel exports use the `Presentation` contract.

## Step 4: Save and Verify

```bash
cap templates widget-templates create --file pie-widget.json --json
cap templates widget-templates get <widget-template-id> --json
cap reporting widgets pie-chart <widget-template-id> \
  --org-nodes <org-node-id> \
  --data-interval month \
  --periods "Jan 25" \
  --json
```

`create` and `save` return an acknowledgement object, not the full updated
template. Use `get` after saving when automation needs to inspect the persisted
`widgetTemplate`.

The render output uses server-resolved metadata. Automation consumers should read
`styleConfiguration`, `centerNumberFormat`, `center`, `labelDisplay`, `legend`,
`dataItems[].presentation`, `dataItems[].numberFormat`,
`dataItems[].legendValue`, `dataItems[].sliceLabel`, and
`selectionDiagnostics` when value selection is configured.

## Unsupported Payloads

Pie styling rejects raw CSS, HTML, JavaScript, callbacks, unsafe URLs,
`url(...)`, arbitrary chart configuration, generic `style` bags, `styleBag`,
and `propertyBag`. Use only the schema/sample fields.

Legacy Pie and Donut payloads that omit styling fields remain valid. Missing
style does not opt the template into configured-style defaults.
