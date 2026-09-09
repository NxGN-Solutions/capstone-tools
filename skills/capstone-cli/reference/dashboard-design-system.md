# Capstone Dashboard Design System

This is the **normative visual standard** for Capstone dashboards built through
the CLI. Follow it and a dashboard will look professional, consistent, and
on-brand without design judgment calls at every step. It was extracted from the
Redux Construction reference implementation (an executive WIP finance review)
and generalizes what made that dashboard work.

**How to use this document:**

1. Read [Layer 0 — Brand](#layer-0--brand-tokens) and research the client's
   corporate identity FIRST. Everything brandable derives from that research.
2. Apply Layers 1–4 (shell, sections, placement, chrome) verbatim — they are
   fixed structure, not choices.
3. Pick widget **variants** from Layer 5 per widget role. Copy the JSON, change
   only the data bindings and the brand slots.
4. Gate the result with [Layer 6 — Language & polish](#layer-6--language--polish-gates)
   and the [verification loop](#verification-loop).

Mechanics (payload shapes, save commands, roundtrips) live in
[create-widget-template.md](../recipes/configuration/create-widget-template.md)
and [build-dashboard.md](../recipes/configuration/build-dashboard.md). This
document defines *what values to put in those payloads*.

---

## Layer 0 — Brand tokens

Every dashboard uses **exactly one brand hue family**, expressed as four
derived roles. Everything else on the dashboard comes from the platform's
semantic tokens (next section) — never invent additional decorative colors.

### Research the client's CI first

Do not default to a house palette. Before building, research the client's
corporate identity and derive the brand hue from it:

- Ask the user for brand guidelines, a logo file, or their website.
- Look at their primary logo color, website header/nav color, and any
  published brand guide. Pick the **one dominant hue** (not a gradient, not
  the full palette).
- If the client has no usable CI (or the hue fails contrast — see below),
  fall back to the platform primary `#6172f3` and say so.

### The four derived roles

| Role | Purpose | Derivation from the brand hue | Redux example (green CI) |
|------|---------|-------------------------------|--------------------------|
| `brand-deep` | Header panel background | Very dark shade of the hue (~10–15% lightness). Must carry white text at ≥ 7:1 contrast | `#0A2A1E` |
| `brand-accent` | Card accent bars, chart series, inverted tooltips | The saturated mid-tone of the hue (~30–40% lightness). Must pass 4.5:1 against white | `#067647` |
| `brand-deep-border` | Border on the header panel | Slightly lighter than `brand-deep` (subtle edge, not a stripe) | `#0F3A2C` |
| `brand-tint` | Secondary text on `brand-deep` | Very light tint of the hue (~85–92% lightness) | `#D7E8DD` |

**Contrast requirements (non-negotiable):** white (`#FFFFFF`) on `brand-deep`
≥ 7:1; `brand-tint` on `brand-deep` ≥ 4.5:1; white on `brand-accent` ≥ 4.5:1.
Check with any WCAG contrast formula before committing.

**Where brand hexes are allowed** — these four slots and nowhere else:

1. Dashboard header panel (`header.style`)
2. Info Card accent bars (`panel.accentColor`)
3. XY chart series color + inverted tooltip background
4. Conditional-format thresholds may additionally use amber `#B45309`
   (exception highlighting — see Layer 6)

Every other color in every payload must be a semantic token from the tables
below.

### Color token registry

Dashboard and widget templates use one shared color-token registry. Discover
the current registry from the installed CLI:

```bash
cap meta lookups get color-tokens --json
cap templates widget-templates schema --widget-type info --json
cap templates dashboard-templates schema --json
```

`schema --json` lists the 17 canonical, non-deprecated token names in color
field `values[]`. The meta lookup returns the full registry: canonical tokens,
deprecated legacy tokens, and aliases. The table below is checked against
`docs/cli/reference/color-tokens.json`; update the generated artifact and this
table together.

| Token | Category | Light hex | CSS variable | Status | Role |
|-------|----------|-----------|--------------|--------|------|
| `primary` | Brand | `#6172f3` | `--cap-color-primary` | Canonical | Platform brand / emphasis |
| `secondary` | Brand | `#475467` | `--cap-color-secondary` | Canonical | Secondary emphasis |
| `chrome` | Brand | `#313a46` | `--cap-color-chrome` | Canonical | Dark chrome surfaces (headers, bands) |
| `success` | Status | `#027a48` | `--cap-color-success` | Canonical | Positive state |
| `warning` | Status | `#b54708` | `--cap-color-warning` | Canonical | Caution state |
| `danger` | Status | `#b42318` | `--cap-color-danger` | Canonical | Negative / breach state |
| `info` | Status | `#3667e3` | `--cap-color-info` | Canonical | Informational state |
| `neutral` | Status | `#667085` | `--cap-color-neutral` | Canonical | No-signal state |
| `surface` | Surface | `#ffffff` | `--cap-color-surface` | Canonical | Card/panel background |
| `surface-canvas` | Surface | `#f9fafd` | `--cap-color-surface-canvas` | Canonical | Page/canvas background |
| `surface-muted` | Surface | `#f7f8fb` | `--cap-color-surface-muted` | Canonical | Quiet chrome (tab strip, filter bar) |
| `text-primary` | Text | `#101828` | `--cap-color-text-primary` | Canonical | Primary text |
| `text-secondary` | Text | `#344054` | `--cap-color-text-secondary` | Canonical | Secondary text |
| `text-muted` | Text | `#667085` | `--cap-color-text-muted` | Canonical | De-emphasized text |
| `border` | Border | `#d0d5dd` | `--cap-color-border` | Canonical | Borders and separators |
| `white` | Absolute | `#ffffff` | `--cap-color-white` | Canonical | Absolute white |
| `black` | Absolute | `#000000` | `--cap-color-black` | Canonical | Absolute black |
| `brand-blue` | Brand | `#064181` | `--cap-color-brand-blue` | Deprecated | Deprecated legacy brand blue |
| `brand-magenta` | Brand | `#950349` | `--cap-color-brand-magenta` | Deprecated | Deprecated legacy brand magenta |

Aliases are accepted for backward compatibility and normalize to canonical
names on save: `accent` → `primary`, `body` → `text-secondary`, `ink` →
`text-primary`, `muted` → `text-muted`, `negative` → `danger`, `positive` →
`success`, `primary-dark` / `primaryDark` → `chrome`, and `unknown` →
`neutral`. New samples should use canonical names. Deprecated legacy tokens
remain valid but are excluded from schema defaults.

Strict save behavior applies to both dashboard and widget templates:

- API, CLI JSON, MCP, and browser saves reject invalid color values with
  actionable validation errors. Use a registry token or `#RGB`, `#RRGGBB`, or
  `#RRGGBBAA` hex.
- Widget and dashboard Excel uploads warn at the cell level and ignore invalid
  color cells; the row still imports.
- `rgb(...)`, `rgba(...)`, `var(...)`, raw CSS, HTML, scripts, callbacks, URLs,
  and unknown identifiers are not accepted color values.

Authoring standard: use semantic tokens everywhere. Raw hex is allowed by the
contract only for deliberate brand/CI slots listed above, the amber
`#B45309` exception highlight, or a documented one-off such as the chart
description inset `#f9fafd`.

Cross-layer notes:

- Safe widget font families: `theme`, `sans`, `serif`, `mono`, `nunito`,
  `roboto`, `poppins`, `arial`. **Always use `theme`** unless the client CI
  demands otherwise.
- Safe trend icons: `arrow-up`, `arrow-down`, `arrow-right`, `caret-up`,
  `caret-down`, `caret-right`, `chevron-up`, `chevron-down`, `triangle-up`,
  `triangle-down`, `plus`, `minus`, `equals`, `circle`, `check`, `xmark`,
  `none`.

---

## Layer 1 — Shell contract

Fixed structure. Only the marked `⟨brand-*⟩` slots change per client.

### Canvas (`dashboardStyle`)

```json
{
  "backgroundColor": "surface-canvas",
  "foregroundColor": "text-primary",
  "fontFamily": { "id": 0, "name": "Theme" },
  "contentPadding": { "top": 0, "right": 0, "bottom": 1, "left": 0, "unit": { "id": 1, "name": "Rem" } },
  "contentWidth": { "id": 3, "name": "Wide" },
  "contentAlignment": { "id": 3, "name": "Stretch" }
}
```

### Header — the branded panel

The header is the **only large branded surface** on the dashboard. Use all
four text slots; they establish hierarchy so section titles don't have to.

| Slot | Content pattern | Example |
|------|-----------------|---------|
| `eyebrow` | Domain / portfolio descriptor | "Multifamily construction portfolio" |
| `title` | Dashboard name in client language | "Redux Construction WIP Review" |
| `subtitle` | Cadence + data source | "Monthly WIP close \| Source: Procore workbook extract" |
| `badge` | Audience / purpose | "Executive finance review" |

```json
{
  "title": "…", "eyebrow": "…", "subtitle": "…", "badge": "…",
  "style": {
    "backgroundColor": "⟨brand-deep⟩",
    "headerBackgroundColor": "⟨brand-deep⟩",
    "contentBackgroundColor": "⟨brand-deep⟩",
    "foregroundColor": "⟨brand-tint⟩",
    "titleColor": "#FFFFFF",
    "borderColor": "⟨brand-deep-border⟩",
    "borderRadius": { "value": 8, "unit": { "id": 0, "name": "Px" } },
    "accentColor": "⟨brand-accent⟩",
    "accentWidth": { "value": 4, "unit": { "id": 0, "name": "Px" } }
  }
}
```

### Filter region — quiet, never boxed

```json
{
  "position": { "id": 0, "name": "Above" },
  "style": {
    "foregroundColor": "text-primary",
    "borderWidth": { "value": 0, "unit": { "id": 0, "name": "Px" } },
    "borderRadius": { "value": 8, "unit": { "id": 0, "name": "Px" } }
  },
  "spacing": {
    "padding": { "top": 0.875, "right": 0, "bottom": 0, "left": 0, "unit": { "id": 1, "name": "Rem" } },
    "margin": { "top": 0, "right": 0, "bottom": 0, "left": 0, "unit": { "id": 1, "name": "Rem" } }
  }
}
```

### Tab strip — neutral, never branded

```json
{
  "style": {
    "foregroundColor": "text-primary",
    "headerBackgroundColor": "surface-muted"
  },
  "hoverBackgroundColor": "surface-muted",
  "hoverForegroundColor": "text-primary"
}
```

> Use registry tokens from Layer 0 or accepted hex values only. Invalid color
> values fail strict saves and are flagged inline by the editor.

---

## Layer 2 — Section grammar

Sections follow a **narrative order** (this is the design methodology, not a
suggestion):

| # | Section role | Answers | Layout |
|---|--------------|---------|--------|
| 01 | Snapshot | "What is the state?" | 4-column KPI grid |
| 02 | Signals | "What changed / where is pressure?" | 2-column charts |
| 03 | Drilldown | "Which items exactly?" | 1-column tables |
| 04 | Narrative | "What do we conclude / do?" | 1-column text / AI summary |

Name sections with a numeric prefix (`01 Executive WIP Snapshot`) so ordering
survives roundtrips and is self-evident in JSON.

**Every section uses this exact recipe** — the only variables are `columns`
(4 / 2 / 1) and gap (`M` everywhere, `S` for the narrative closer):

```json
{
  "nodeLayout": {
    "layoutMode": { "id": 2, "name": "Grid" },
    "horizontalAlignment": { "id": 4, "name": "Stretch" },
    "verticalAlignment": { "id": 1, "name": "Start" },
    "headerBehavior": { "id": 1, "name": "Visible" },
    "spacing": {
      "margin": { "top": 0, "right": 0, "bottom": 1, "left": 0, "unit": { "id": 1, "name": "Rem" } },
      "gap": { "id": 3, "name": "M" }
    },
    "responsive": { "stackBelow": { "id": 2, "name": "Md" }, "columns": 4, "wrap": true }
  },
  "nodeStyle": {
    "backgroundColor": "surface-canvas",
    "headerBackgroundColor": "chrome",
    "titleColor": "white",
    "foregroundColor": "white",
    "fontFamily": { "id": 0, "name": "Theme" },
    "fontWeight": { "id": 4, "name": "Bold" },
    "fontSize": { "value": 1, "unit": { "id": 1, "name": "Rem" } },
    "typography": { "id": 1, "name": "Compact" },
    "borderWidth": { "value": 0, "unit": { "id": 0, "name": "Px" } },
    "borderRadius": { "value": 4, "unit": { "id": 0, "name": "Px" } },
    "shadow": { "id": 1, "name": "None" }
  }
}
```

The section header is a slim **chrome band with white bold 1rem text** — it
separates content without competing with the branded dashboard header. Section
bodies stay transparent (`surface-canvas`) so the widgets' white panels carry
the visual weight.

---

## Layer 3 — Placement roles

Widgets have exactly **three placement roles**. Do not invent per-widget
placements.

| Role | `widgetSize` | `widthBehavior` | `responsive.fullWidthBelow` | Used by |
|------|--------------|-----------------|------------------------------|---------|
| KPI card | 25% | `WidgetSizeDefault` | `Sm` | Info Cards in the snapshot row |
| Chart | 50% | `WidgetSizeDefault` | `Md` | XY charts, pies/donuts |
| Full-width | 100% | `Fill` | `Md` | Tables, TextBlocks, AI Summary |

All three share `horizontalAlignment: Stretch`, `verticalAlignment: Start`.

> `Fill` overrides multi-column grids — that's why it is reserved for the
> full-width role. Using `Fill` on a KPI card destroys the 4-column snapshot
> row.

---

## Layer 4 — Widget chrome and the type ramp

### Shared chrome (every widget type)

| Slot | Standard |
|------|----------|
| `panel` | `backgroundColor: surface`, `borderColor: border`, `borderWidth: 1`, `borderRadius: 8` |
| `panel.padding` | 16 (tables) / 18 (charts) / 20 (cards, text blocks) |
| `panel.gap` | 8 (10 for text blocks) |
| `stateMessages` | `foregroundColor: warning` |
| Footnote slot | `fontSize: 12`, `foregroundColor: text-muted` |

### The type ramp

All font sizes on a dashboard come from this ramp — no other sizes:

| Size / weight | Role |
|---------------|------|
| **30 Bold** | KPI value (`value` slot, Info Card) |
| **26 Bold** | Donut center value |
| **20 Bold** | TextBlock title |
| **16 Semibold** | Chart / table / donut title |
| **14 Semibold** | Info Card title (`text-secondary`) |
| **14 Normal / 1.5 line-height** | TextBlock body |
| **13** | Table cells, legend labels/values |
| **12** | Axis labels, footnotes, descriptions, grid headers |
| **12 Semibold + Uppercase + letterSpacing 1** | Grid headers, TextBlock subtitles (the "label" treatment) |

### Description insets (charts)

When a chart carries an explanation, style it as a quiet inset, not free text:

```json
"description": {
  "backgroundColor": "#f9fafd",
  "borderColor": "border", "borderWidth": 1, "borderRadius": 8,
  "fontFamily": "theme", "fontSize": 12,
  "foregroundColor": "text-secondary", "padding": 8
}
```

---

## Layer 5 — Widget type specifications

Copy the variant closest to the widget's role; change only data bindings,
titles, and the marked brand slots.

### 5.1 Info Card

**Variant `KPI-Accent`** — the standard snapshot-row card (Redux reference):

```json
"styleConfiguration": {
  "panel": {
    "backgroundColor": "surface", "accentColor": "⟨brand-accent⟩",
    "fontFamily": "theme", "borderColor": "border", "borderWidth": 1,
    "borderRadius": 8, "accentSide": "Left", "accentWidth": 4,
    "padding": 20, "gap": 8
  },
  "title":    { "foregroundColor": "text-secondary", "fontFamily": "theme", "fontWeight": "Semibold", "fontSize": 14 },
  "value":    { "foregroundColor": "text-primary",  "fontFamily": "theme", "fontWeight": "Bold",     "fontSize": 30 },
  "footnote": { "foregroundColor": "text-muted",    "fontFamily": "theme", "fontSize": 12 },
  "stateMessages": { "foregroundColor": "warning" }
}
```

Other variants (delta from `KPI-Accent`):

| Variant | Delta | When |
|---------|-------|------|
| `KPI-Quiet` | Remove `accentColor`/`accentSide`/`accentWidth` | Secondary metrics that shouldn't compete with the headline row |
| `KPI-Alert` | `accentColor: "danger"` | A metric currently breaching its threshold — use sparingly, alert accents are earned by data, not chosen at design time |
| `KPI-Comparison` | Two data items (actual + comparator role-metric, e.g. Budget/Target/Forecast) | When the decision needs "vs what?" — the comparator is a metric, never a hardcoded number |

Card content contract: `title` = plain-language metric name, `footnote` = one
sentence explaining derivation ("Net billed less payments received."). Trend
sentiment follows the numeric sign; for cost-type metrics where up is bad,
swap the `trend.up`/`trend.down` colors (`danger`/`success`) — never rely on
color alone, the icon and value carry the signal too.

### 5.2 Pie / Donut

**Variant `Composition-Donut`** — the standard composition widget:

- 2–6 slices only. More than 6 → the data belongs in a ranked bar or table.
- `pieChartWidgetTemplateType: Donut`, legend on, per-slice `showInLegend: true`.
- Center shows a **live metric value** (`centerMode` metric-value +
  `centerNumberFormat`) — never a static text label that will go stale.

```json
"styleConfiguration": {
  "panel": { "backgroundColor": "surface", "borderColor": "border", "borderWidth": 1,
             "borderRadius": 8, "fontFamily": "theme", "foregroundColor": "text-primary",
             "gap": 8, "padding": 18 },
  "title":        { "fontFamily": "theme", "fontSize": 16, "fontWeight": "Semibold", "foregroundColor": "text-primary" },
  "chartArea":    { "backgroundColor": "surface", "padding": 8 },
  "donutCenter":  { "fontFamily": "theme", "fontSize": 26, "fontWeight": "Bold", "foregroundColor": "text-primary" },
  "legend":       { "backgroundColor": "surface", "gap": 8 },
  "legendLabels": { "fontFamily": "theme", "fontSize": 13, "foregroundColor": "text-primary" },
  "legendValues": { "fontFamily": "theme", "fontSize": 13, "fontWeight": "Semibold", "foregroundColor": "text-secondary" },
  "sliceBorders": { "borderColor": "surface", "borderWidth": 2 },
  "sliceLabels":  { "fontFamily": "theme", "fontSize": 12, "fontWeight": "Medium", "foregroundColor": "text-primary" },
  "tooltip":      { "backgroundColor": "surface", "fontSize": 12, "foregroundColor": "text-primary" },
  "stateMessages": { "foregroundColor": "warning" }
}
```

Variant `Mix-Pie`: same styling, `pieChartWidgetTemplateType: PieChart`, no
center — only when the whole-vs-parts metaphor matters more than the total.

Slice colors default to the platform palette; override per-slice
(`dataItems[].presentation`) only to echo a color system the client already
uses (their status colors, their spreadsheet conventions).

### 5.3 XY Chart

**Variant `Breakdown-Column`** — single-period composition/comparison
(Redux reference):

```json
"styleConfiguration": {
  "version": "1",
  "panel": { "backgroundColor": "surface", "borderColor": "border", "borderWidth": 1,
             "borderRadius": 8, "foregroundColor": "text-primary", "gap": 8, "padding": 18 },
  "title":     { "fontFamily": "theme", "fontSize": 16, "fontWeight": "Semibold", "foregroundColor": "text-primary" },
  "axisLabel": { "fontSize": 12, "foregroundColor": "text-secondary" },
  "axisTitle": { "fontFamily": "theme", "fontSize": 12, "fontWeight": "Normal", "foregroundColor": "text-muted" },
  "categoryAxis": { "labelDensity": "Compact", "labelRotation": 0, "minLabelGap": 20 },
  "dataLabel": { "displayMode": "Always", "foregroundColor": "surface" },
  "series":    { "color": "primary", "opacity": 0.95, "strokeWidth": 2 },
  "tooltip":   { "backgroundColor": "⟨brand-accent⟩", "borderRadius": 6, "fontWeight": "Semibold",
                 "foregroundColor": "surface", "padding": 8 },
  "numberFormat": {
    "axisValue": { "precision": 0 },
    "dataLabel": { "precision": 1, "magnitude": "None" },
    "legendValue": { "precision": 1 }
  },
  "stateMessages": { "foregroundColor": "warning" }
}
```

Per-series brand override (`dataItems[].presentation`): `series.color:
⟨brand-accent⟩` and the **inverted tooltip** (tooltip background = series
color, `foregroundColor: surface`) — this ties hover feedback to the series
identity.

| Variant | Delta | Rules |
|---------|-------|-------|
| `Trend-Line` | `xyChartWidgetTemplateDataItemType: Line` | `timePeriodAggregationMethod: None` at template **and** data-item level (any other value collapses the trend to one point). Actual = brand accent; plan/budget comparator = near-black dashed; forecast = amber dotted |
| `Breakdown-Column` | (reference above) | `dataLabel Always` only when ≤ ~8 categories; otherwise labels off, tooltip carries values |
| `Ranking-Bar` | `Bar` type + `invertAxes: true` | For ranked comparisons with long labels; top-N via `partitioningRankMode` + limit |

Keep the sample's `axes` array verbatim and reference its axis id from every
data item — axis label text is the axis `name` (put the unit there:
"Revenue (R)").

### 5.4 Table

**Variant `Drilldown`** — the full population with exception highlighting
(Redux reference):

```json
"styleConfiguration": {
  "panel": { "backgroundColor": "surface", "borderColor": "border", "borderWidth": 1,
             "borderRadius": 8, "padding": 16 },
  "gridHeader": { "backgroundColor": "surface", "fontFamily": "theme", "fontSize": 12,
                  "fontWeight": "Semibold", "foregroundColor": "text-secondary",
                  "letterSpacing": 1, "textTransform": "Uppercase" },
  "rowLabels":  { "fontFamily": "theme", "fontSize": 13, "foregroundColor": "text-primary" },
  "valueCells": { "fontFamily": "theme", "fontSize": 13, "foregroundColor": "text-secondary",
                  "textAlign": "End" },
  "rowLabelHeader": "Scope",
  "conditionalFormatRules": [
    { "configuredColumnId": "metric:⟨metric-id⟩", "operator": "Negative",
      "style": { "foregroundColor": "danger" } },
    { "configuredColumnId": "metric:⟨metric-id⟩", "operator": "GreaterThan", "threshold": 250000,
      "style": { "foregroundColor": "#B45309" } }
  ]
}
```

Table rules:

- Numbers are **right-aligned** (`textAlign: End`), row labels left.
- `rowLabelHeader` gets a real name ("Scope", "Project") — never the default.
- Conditional rules highlight **genuine exceptions only**: tune thresholds so
  ≤ ~20% of visible rows fire. A rule firing on most rows is wallpaper, not a
  signal. `danger` token for hard breaches, amber `#B45309` for watch items.
- The footnote **explains the color coding**: "Amber: open AR above $250k.
  Red: projected cost overrun."

Variant `Watchlist`: same styling; content = **top-N worst** by the risk
metric (rank + limit), titled as a watchlist ("Project Risk Watchlist"), with
the same conditional rules. Use it to answer the "which ones?" question
without shipping the whole population.

Tables answer *exact-value lookup*. If the task is comparison or ranking, use
a bar chart and keep the exact values in a report template.

### 5.5 Text Block

**Variant `Narrative`** — the close commentary (Redux reference):

```json
"styleConfiguration": {
  "panel": { "backgroundColor": "surface", "borderColor": "border", "borderWidth": 1,
             "borderRadius": 8, "fontFamily": "theme", "foregroundColor": "text-primary",
             "gap": 10, "padding": 20 },
  "title":    { "fontFamily": "theme", "fontSize": 20, "fontWeight": "Bold",
                "foregroundColor": "text-primary", "lineHeight": 1.2 },
  "subtitle": { "fontFamily": "theme", "fontSize": 12, "fontWeight": "Semibold",
                "foregroundColor": "text-secondary", "letterSpacing": 1,
                "lineHeight": 1.4, "textTransform": "Uppercase" },
  "description": { "fontFamily": "theme", "fontSize": 14, "fontWeight": "Normal",
                   "foregroundColor": "text-primary", "lineHeight": 1.5 },
  "footnote": { "fontFamily": "theme", "fontSize": 12, "foregroundColor": "text-muted",
                "lineHeight": 1.4 },
  "stateMessages": { "foregroundColor": "warning" }
}
```

| Variant | Content contract |
|---------|------------------|
| `Narrative` | Title uses the period token ("@[period] WIP Close"); body = what happened this period in client language; footnote = source attribution |
| `Escalation` | States the dashboard's escalation trigger explicitly ("Escalate any project where …") — this puts the decision contract on the dashboard itself. Place it at the top of the drilldown section, before the table it governs |

### 5.6 AI Summary

No styleConfiguration of its own (it is a dashboard tree node, `widgetTemplate:
null`). Placement: full-width role, **last widget in a section**, context
`Peers`. One per dashboard (or per tab) — it is the "Insights" closer, not a
per-section garnish.

---

## Number format matrix

Precision is hierarchy: KPI cards compress, tables show the working numbers.

| Context | `magnitude` | `precision` | Example |
|---------|-------------|-------------|---------|
| KPI card, values ≥ $1M | `Millions` | 2 | `$1.56M` |
| KPI card, values in $100k–999k | `Thousands` | 1 | `$735.2k` |
| KPI card, percentage | `None` | 1 | `21.4%` |
| Donut center / legend values | match the KPI showing the same metric | 1 | `$735.2k` |
| Chart data labels | `None` | 1 | `21.4` |
| Chart axis values | `None` | 0 | `20` |
| Table cells, currency | `None` | 0 | `1,556,214` |
| Table cells, percentage | `None` | 1 | `21.4%` |

Two KPI cards showing values in the same unit must use the same
magnitude+precision — `$1.56M` next to `$1.09M`, never `$1.6M` next to
`$1.09M` (2 dp is the Millions standard so nearby values stay
distinguishable).

---

## Layer 6 — Language & polish gates

A dashboard that passes Layers 1–5 is *consistent*. These gates make it
*good*. Check every one before calling a dashboard done:

1. **Client language only.** Every visible string (titles, subtitles,
   footnotes, section names) is in the audience's professional vocabulary.
   Zero modeler jargon: no "projection", "lens", "child node",
   "after-aggregation", "org node".
2. **No hardcoded periods.** Use `@[period]` tokens in titles and footnotes;
   the filter drives the text.
3. **One explanation per widget.** A footnote or description explains
   derivation once — no duplicated sentences across widgets.
4. **Exceptions, not wallpaper.** Conditional formatting thresholds tuned so
   highlights are rare enough to mean something.
5. **Live values over static text.** Donut centers, comparisons, and callout
   numbers come from metrics, never typed-in numbers that go stale.
6. **The narrative is complete.** State → change → which items → what to do:
   if the "which items?" question has no widget (a watchlist), add it; if the
   escalation rule isn't stated on the dashboard, add the Escalation text
   block.
7. **Demo/period robustness.** Selecting an adjacent period must not produce
   an all-zero dashboard. Load at least two periods of data.
8. **Color discipline audit.** Grep your payloads for `#`: every hex found
   must be one of the four brand roles, `#B45309`, or `#f9fafd` — anything
   else is a violation.

---

## Verification loop

After building or restyling, verify from the CLI (details in
[verify-data.md](../recipes/reporting/verify-data.md)):

```bash
# 1. Contract truth per widget type (enums, bounds)
cap templates widget-templates schema --widget-type <info|pie|xy|table> --json

# 2. Typed render checks — styling + data actually resolve
cap reporting widgets info-card <id> --org-nodes <id> --data-interval month --periods "<period>" --json
cap reporting widgets pie-chart <id> ... --json     # returns styleConfiguration, center, legend, dataItems

# 3. Contrast / style diagnostics come back on the render contract — fix any LowContrast warnings

# 4. Dashboard-level audit
cap templates dashboard-templates audit <dashboard-id> --strict --json
```

Render contracts are the ground truth that a style survived save-and-resolve —
`save` acknowledgements alone don't prove the widget renders.
