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
- Math: `SQRT()`, `POW()`, `MOD()`, `PI()`
- Aggregation: `SUM()`, `AVG()`, `COUNT()`, `MIN()`, `MAX()`, `FIRST()`, `LAST()`, `SUMPRODUCT()`
- Null handling: `IFNULL()`, `COALESCE()`, `DIV()`
- Time: `NOW()`, `MONTH()`, `YEAR()`, `YEARSTART()`, `YEAREND()`, `MONTHSTART()`, `MONTHEND()`, `DATEOFFSET()`, `ISFUTURE()`
- Duration: `DAYS()`, `WEEKS()`, `MONTHS()`, `QUARTERS()`, `YEARS()`

**Common patterns:**

| Pattern | Formula | Use Case |
|---------|---------|----------|
| Simple ratio | `[Numerator] / [Denominator]` | Intensity metrics |
| Safe division | `IF [Denominator] <> 0 THEN [Numerator] / [Denominator] ELSE 0` | Avoid divide-by-zero |
| Percentage | `IF [Total] <> 0 THEN [Part] / [Total] * 100 ELSE 0` | Share calculations |
| Conversion | `[Source] * 0.001` | Unit conversion (e.g., kWh to MWh) |
| Emission factor | `[Consumption] * 2.68` | CO2e from fuel consumption |

> **Tip:** Use IDs in formulas for CLI usage — they're unambiguous, rename-safe, and what the API stores internally.

**Present to user:**
```
Formula: IF [Headcount] <> 0 THEN [Total Electricity] / [Headcount] ELSE 0

Using IDs:
IF [<id>] <> 0 THEN [<id>] / [<id>] ELSE 0
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
- **Ratios and intensities** → After Aggregations (you want total/total, not average of ratios)
- **Unit conversions** → Before Aggregations (convert at each node, then roll up)
- **Emission factors** → Before Aggregations (apply factor per site, then sum)

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
Formula:         IF [Headcount] <> 0 THEN [Total Electricity] / [Headcount] ELSE 0
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
  "formula": "IF [<id>] <> 0 THEN [<id>] / [<id>] ELSE 0",
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
cap model formula-validation validate --formula "IF [<id>] <> 0 THEN [<id>] / [<id>] ELSE 0" --json
```

**Present confirmation:**
```
Calculation created successfully!

ID:             <id>
Name:           Energy Intensity
Formula:        IF [Headcount] <> 0 THEN [Total Electricity] / [Headcount] ELSE 0
Calc Phase:     After Aggregations

Next steps:
- Values will compute automatically when referenced inputs have data
- View results: cap reporting computed-values list --template <report-template-id> --data-interval month --periods "Jan 2026" --json
```

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
{ "name": "Energy Intensity", "formula": "IF [Headcount] <> 0 THEN [calc-energy-id] / [Headcount] ELSE 0", ... }
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
