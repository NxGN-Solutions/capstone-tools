# Recipe: Create Calculation

> Guided workflow for creating a formula-based Calculation metric.

## When to Use

- "I need to calculate emissions from fuel consumption"
- "Create a formula for energy intensity"
- "Add a derived metric that divides X by Y"
- "Set up a percentage calculation"
- "How do I add a calculated KPI?"

## Required Context

Before starting, Claude should know:
- [ ] What the user wants to calculate (name/concept)
- [ ] Which input metrics it derives from
- [ ] Optional: The formula logic

**If missing:** Claude will ask clarifying questions in Step 1.

---

## Step-by-Step

### Step 1: Clarify the Calculation

**Purpose:** Understand exactly what the user wants to compute.

**Ask if not provided:**
- "What would you like to call this calculation?" (e.g., "Energy Intensity")
- "What formula should it use?" (e.g., "Total Energy / Headcount")
- "Which existing metrics does it reference?"

**Look for:**
- Whether the result is a rate, ratio, percentage, or derived total
- Whether it should compute before or after aggregation (see Step 5)

---

### Step 2: Find Referenced Metrics

**Purpose:** Get IDs for metrics used in the formula.

**Command:**
```bash
cap model metrics list --json
```

**Match the user's description to existing metrics:**
```
Looking for formula <id>

Found:
- "Total Electricity" (<id>) — Input, kWh, Sum
- "Headcount" (<id>) — Input, count, Last Value

Formula: Total Electricity / Headcount = kWh per person
```

**If a referenced metric doesn't exist:**
- "The metric '[name]' doesn't exist yet. Would you like to create it first?"
- Guide to [Create Metric Wizard](./create-metric.md)

---

### Step 3: Select Discipline and Unit

**Purpose:** Categorize and set the output unit.

**Commands:**
```bash
cap masterdata disciplines list --json
cap masterdata units list --json
```

**Guidance:**
- The discipline should match the calculation's domain (e.g., Energy for intensity metrics)
- The unit should reflect the result (e.g., kWh/person for an intensity ratio)
- If the output unit doesn't exist, create it first: `cap masterdata units create`

---

### Step 4: Write the Formula

**Purpose:** Express the calculation in Capstone's formula language.

**Formula syntax:**
- Reference by name: `[Metric Name]`
- Reference by ID (recommended for CLI): `[<id>]`
- Arithmetic: `+`, `-`, `*`, `/`
- Conditional: `IF [condition] THEN [value] ELSE [value]`
- Comparison: `=`, `<>`, `>`, `<`, `>=`, `<=`
- Math: `ABS()`, `SQRT()`, `POW()`, `MOD()`, `PI()`
- Aggregation: `SUM()`, `AVG()`, `MEAN()`, `COUNT()`, `MIN()`, `MAX()`, `FIRST()`, `LAST()`, `SUMPRODUCT()`
- Statistics: `MEDIAN()`, `VARIANCE()`, `VARIANCEP()`, `STDEV()`, `STDEVP()`
- Null handling: `IFNULL()`, `COALESCE()`, `DIV()`
- Time: `NOW()`, `MONTH()`, `YEAR()`, `YEARSTART()`, `YEAREND()`, `MONTHSTART()`, `MONTHEND()`, `DATEOFFSET()`, `ISFUTURE()`
- Duration: `DAYS()`, `WEEKS()`, `MONTHS()`, `QUARTERS()`, `YEARS()`
- Forecasting: `FORECAST()`, `HOLT()`, `HOLTWINTERS()`, `HOLTWINTERSM()`

**Common patterns:**

| Pattern | Formula | Use Case |
|---------|---------|----------|
| Simple ratio | `[Numerator] / [Denominator]` | Intensity metrics |
| Safe division | `DIV([Numerator], [Denominator], null)` | Ratio that stays blank when there is no data |
| Percentage | `DIV([Part] * 100, [Total], null)` | Share calculations |
| Conversion | `[Source] * 0.001` | A converted metric other formulas need. For display in another unit, configure a unit conversion instead |
| Emission factor | `[Consumption] * 2.68` | CO2e from fuel consumption |

> **Tip:** Use IDs in formulas for CLI usage — they're unambiguous, rename-safe, and what the API stores internally.

**Present to user:**
```
Formula: DIV([Total Electricity], [Headcount], null)

Using IDs:
DIV([<id>], [<id>], null)
```

---

### Step 5: Determine Calculation Phase

**Purpose:** Choose when the formula evaluates relative to aggregation.

```
When should this formula compute?

┌──────────────────────────┬──────────────────────────────────────────┐
│ Phase                    │ Use When                                  │
├──────────────────────────┼──────────────────────────────────────────┤
│ Before Aggregations (0)  │ Compute at each leaf node first, then    │
│                          │ aggregate results up the org tree.        │
│                          │ Example: Convert kWh to MWh at each site │
├──────────────────────────┼──────────────────────────────────────────┤
│ After Aggregations (1)   │ Aggregate inputs first, then compute     │
│                          │ the formula on the rolled-up totals.      │
│                          │ Example: Total Energy / Total Headcount  │
└──────────────────────────┴──────────────────────────────────────────┘
```

**Decision guidance:**
- **Ratios, intensities, percentages, target achievement** → After Aggregations (you want total/total, not average of ratios)
- **Period-over-period change** (`[X]|0| - [X]|-1|`) → After Aggregations, so each interval compares with its own previous period
- **Unit conversions with a factor** → Before Aggregations (convert at each node, then roll up)
- **Emission factors** → Before Aggregations (apply factor per site, then sum)
- **Counts of readings against a limit** → Before Aggregations, Sum (see the Modelling Rules in the [Model Building Reference](../../reference/model-building.md#modelling-rules))

**Set the data interval with the phase:**
- **Before Aggregations:** the engine evaluates the formula only for periods of the calculation's data interval, then collapses or expands the result with its time aggregation method. Set the data interval to the cadence of the inputs the formula reads. A Month calculation over quarterly readings counts each reading three times at quarter and year.
- **After Aggregations:** the formula is re-evaluated at every enabled interval, so the data interval is only the default display interval.
- Set the interval right at creation. If you change it later, verify the values at every interval again (Step 9).

---

### Step 6: Determine Aggregation Methods

**Purpose:** Define how the calculated result rolls up.

**Org Structure Aggregation:**
- Calculations that run **Before Aggregations** → typically `Sum` (values roll up after computing)
- Calculations that run **After Aggregations** → typically `None` (formula recomputes at each level)

**Time Period Aggregation:**
- Rate/percentage metrics → `None` (recompute for each time window)
- Cumulative totals → `Sum`

See [Model Building Reference](../../reference/model-building.md#common-aggregation-patterns) for the full pattern table.

---

### Step 7: Confirm Before Creating

**Purpose:** Verify all selections before committing.

**Present summary:**
```
Ready to create calculation:

Name:            Energy Intensity
Description:     Electricity consumption per person
Discipline:      Environmental > Energy
Unit:            kWh/person
Formula:         DIV([Total Electricity], [Headcount], null)
Calc Phase:      After Aggregations
Org Aggregation: None (recomputed at each level)
Time Aggregation: None

Proceed? [Yes/No]
```

**Wait for user confirmation before executing.**

---

### Step 8: Create the Calculation

**Purpose:** Execute the creation command.

**Command:**
```bash
cat <<'EOF' | cap model calculations create --json
{
  "id": "<empty-id>",
  "name": "Energy Intensity",
  "description": "Electricity consumption per person",
  "discipline": { "id": "<discipline-id>" },
  "unitOfMeasure": { "id": "<unit-id>" },
  "dataInterval": { "id": 2, "name": "Month" },
  "precision": 2,
  "calculationPhase": { "id": 1, "name": "After Aggregations" },
  "orgStructureAggregationMethod": { "id": 3, "name": "None" },
  "timePeriodAggregationMethod": { "id": 0, "name": "None" },
  "formula": "DIV([<id>], [<id>], null)",
  "attributeValues": []
}
EOF
```

> **Note:** Use the zero ID for `id` when creating. Enum fields use `{ "id": <int>, "name": "<name>" }` format. See [Model Building Reference](../../reference/model-building.md) for all enum ID lookup tables.

**What to look for:**
- Success response: `{ "success": true, "id": "<new-id>" }`
- The returned ID can be referenced in other calculations' formulas

**If error:**
- `VALIDATION_ERROR` → Check JSON structure; common issue: formula references a non-existent metric ID
- Formula syntax errors are returned with a description of the problem

---

### Step 9: Verify and Validate

**Purpose:** Confirm the calculation was created correctly.

**Commands:**
```bash
# Verify the calculation
cap model calculations get <new-id> --json

# Validate the formula syntax
cap model formula-validation validate --formula "DIV([<id>], [<id>], null)" --json
```

**Present confirmation:**
```
Calculation created successfully!

ID:             <id>
Name:           Energy Intensity
Formula:        DIV([Total Electricity], [Headcount], null)
Calc Phase:     After Aggregations

Next steps:
- Values will compute automatically when referenced inputs have data
- View results: cap reporting computed-values list --template <report-template-id> --data-interval month --periods "Jan 2026" --json
```

**Verify the numbers, not just the save:**
1. Wait for settlement: `cap data recalculation wait <model-version> --json`.
2. Pick one org node and one period with data. Read the calculation and its inputs at month, quarter and year with `cap reporting computed-values query`.
3. Recompute the value by hand from the inputs. The month, quarter and year values must follow the time aggregation rules in the [Model Building Reference](../../reference/model-building.md#cross-interval-time-behavior).
4. If a number is wrong, check the interval, phase, aggregation methods and formula against the engine evaluation order before suspecting the engine.

---

## Example Interaction

**User:** "I need to calculate CO2 emissions from our diesel consumption"

**Claude:** I'll help you create an emissions calculation. Let me find the diesel input metric.

```bash
cap model metrics list --json
```
Found "Diesel Consumption" (<id>) — Input, litres, Sum

**Claude:** For diesel CO2 emissions, the standard factor is 2.68 kg CO2e per litre. The formula would be:

```
[Diesel Consumption] * 2.68
```

This should compute **Before Aggregations** — apply the emission factor at each site, then sum up the org tree.

Here's what I'll create:
```
Name:           Diesel CO2 Emissions
Discipline:     Environmental > Emissions
Unit:           kg CO2e
Formula:        [<id>] * 2.68
Calc Phase:     Before Aggregations
Org Aggregation: Sum
Time Aggregation: Sum
```

Proceed?

**User:** "Yes"

**Claude:**
```bash
cat <<'EOF' | cap model calculations create --json
{
  "id": "<empty-id>",
  "name": "Diesel CO2 Emissions",
  "description": "CO2 equivalent emissions from diesel consumption (2.68 kg CO2e/L)",
  "discipline": { "id": "<emissions-discipline-id>" },
  "unitOfMeasure": { "id": "<kg-co2e-unit-id>" },
  "dataInterval": { "id": 2, "name": "Month" },
  "precision": 2,
  "calculationPhase": { "id": 0, "name": "Before Aggregations" },
  "orgStructureAggregationMethod": { "id": 0, "name": "Sum" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "formula": "[<id>] * 2.68",
  "attributeValues": []
}
EOF
```

Calculation created! Emissions will compute automatically wherever diesel consumption data exists.

---

## Variations

### Batch Creation (Multiple Related Calculations)

When creating calculations that reference each other, create them in dependency order:

```bash
# 1. Create base calculation first
cat <<'EOF' | cap model calculations create --json
{ "name": "Total Energy", "formula": "[Electricity] + [Gas] + [Diesel]", ... }
EOF
# Returns ID: calc-energy-id

# 2. Create dependent calculation using the new ID
cat <<'EOF' | cap model calculations create --json
{ "name": "Energy Intensity", "formula": "DIV([calc-energy-id], [Headcount], null)", ... }
EOF
```

See [Model Building Reference](../../reference/model-building.md#create-calculation) for the full payload template.

### Updating a Formula

Use `save` instead of `create` with the existing ID:

```bash
cap model calculations get <id> --json > calc.json
# Edit formula in calc.json
cap model calculations save --file calc.json --json
```

---

## Related Recipes

- [Create Metric Wizard](./create-metric.md) — Create input metrics for formula references ✅
- [Enter Data](../data-management/enter-data.md) — Enter values for input metrics ✅
- [Talk to Your Data](../exploration/talk-to-your-data.md) — Analyze calculated results ✅
