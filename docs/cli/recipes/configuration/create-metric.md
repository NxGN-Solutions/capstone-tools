# Recipe: Create Metric Wizard

> Guided workflow for creating a new Input metric with proper configuration.

## When to Use

- "I need to track diesel consumption"
- "Create a metric for water usage"
- "Add a new KPI for safety incidents"
- "Set up tracking for [something measurable]"
- "How do I add a metric?"

## Required Context

Before starting, Claude should know:
- [ ] What the user wants to measure (metric name/concept)
- [ ] Optional: Description of what it measures
- [ ] Optional: Preferred unit of measure

**If missing:** Claude will ask clarifying questions in Step 1.

---

## Step-by-Step

### Step 1: Clarify What to Measure

**Purpose:** Understand exactly what the user wants to track.

**Ask if not provided:**
- "What would you like to call this metric?" (e.g., "Diesel Consumption")
- "What does it measure?" (brief description)
- "Do you have a preferred unit?" (e.g., litres, kWh, tonnes)

**Look for:**
- Clear, descriptive name
- Understanding of whether it's a count, amount, rate, or percentage
- Any regulatory or framework context

---

### Step 2: Select Discipline (Category)

**Purpose:** Categorize the metric for organization and reporting.

**Command:**
```bash
cap masterdata disciplines list --json
```

**Present to user:**
```
Available disciplines:
1. Environmental
   - Energy
   - Emissions
   - Water
   - Waste
2. Social
   - Health & Safety
   - Employee Engagement
3. Governance
   - Ethics
   - Compliance
```

**Get user selection:**
- "Which discipline best fits this metric?"
- If they're unsure, help categorize based on what they're measuring

**If no suitable discipline exists:**
- "The discipline you need doesn't exist yet. Would you like to create it first?"
- Guide to `cap masterdata disciplines create` if needed

---

### Step 3: Select Unit of Measure

**Purpose:** Ensure values have correct units for display and calculations.

**Command:**
```bash
cap masterdata units list --json
```

**Filter by relevance:**
- Energy metrics: kWh, MWh, GJ, BTU
- Volume metrics: L, kL, m³, gallons
- Mass metrics: kg, tonnes, lbs
- Count metrics: count, FTE, headcount
- Rates: per hour, per 100 employees

**Present to user:**
```
Suggested units for [metric type]:
- litres (L) — volume measurement
- kilolitres (kL) — larger volume
- cubic metres (m³) — large volume

Or search all units: [name]
```

**If unit doesn't exist:**
- "This unit isn't configured. Would you like to create it?"
- Guide to `cap masterdata units create` if needed

---

### Step 4: Determine Aggregation Method

**Purpose:** Define how values combine across locations and time.

**Explain the options:**

```
How should values aggregate?

┌──────────────┬─────────────────────────────────────────────────┐
│ Method       │ Use When                                        │
├──────────────┼─────────────────────────────────────────────────┤
│ Sum          │ Values add up (consumption, emissions, hours)   │
│              │ Example: Total diesel across all sites          │
├──────────────┼─────────────────────────────────────────────────┤
│ Average      │ Values should average (rates, scores, %)        │
│              │ Example: Average satisfaction score             │
├──────────────┼─────────────────────────────────────────────────┤
│ Last Value   │ Most recent value matters (status, count)       │
│              │ Example: Current headcount, account balance     │
├──────────────┼─────────────────────────────────────────────────┤
│ Count        │ Count occurrences (tracking completeness)       │
│              │ Example: Number of sites reporting data         │
├──────────────┼─────────────────────────────────────────────────┤
│ None         │ Node-specific, doesn't roll up                  │
│              │ Example: Site-specific certification status     │
└──────────────┴─────────────────────────────────────────────────┘
```

**Decision guidance:**
- "Is [metric name] something that adds up, or should it average?"
- "If Site A uses 100L and Site B uses 150L, is the total 250L (Sum) or 125L (Average)?"

**Common patterns:**
| Metric Type | Typical Aggregation |
|-------------|---------------------|
| Consumption (fuel, water, energy) | Sum |
| Emissions | Sum |
| Hours worked | Sum |
| Headcount | Last Value |
| Rates (per hour, per employee) | Average |
| Percentages | Average |
| Scores | Average |

---

### Step 5: Determine Data Interval

**Purpose:** Define how often data is captured.

**Options:**
- **Monthly** — Most common for operational metrics
- **Quarterly** — Financial and reporting metrics
- **Annual** — Strategic targets, certifications
- **Weekly/Daily** — High-frequency operational data

**Ask:**
- "How often will this data be captured?"
- Default to Monthly if user is unsure

---

### Step 6: Confirm Before Creating

**Purpose:** Verify all selections before committing.

**Present summary:**
```
Ready to create metric:

Name:           Diesel Consumption
Description:    Monthly diesel fuel usage for fleet vehicles
Discipline:     Environmental > Energy
Unit:           litres (L)
Aggregation:    Sum (values add up across locations and time)
Data Interval:  Monthly

Proceed? [Yes/No]
```

**Wait for user confirmation before executing.**

---

### Step 7: Create the Metric

**Purpose:** Execute the creation command.

**Command:**
```bash
cat <<'EOF' | cap model inputs create --json
{
  "id": "<empty-id>",
  "name": "Diesel Consumption",
  "description": "Monthly diesel fuel usage for fleet vehicles",
  "reference": "",
  "discipline": { "id": "<discipline-id>" },
  "unitOfMeasure": { "id": "<unit-id>" },
  "dataInterval": { "id": 2, "name": "Month" },
  "precision": 0,
  "orgStructureAggregationMethod": { "id": 0, "name": "Sum" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "allowForecastedData": false,
  "requireValidation": false,
  "requireDataCapture": false,
  "inputDataFeeds": [
    {
      "id": "<empty-id>",
      "name": "Manual Capture",
      "dataSource": { "id": "<manual-capture-datasource-id>" },
      "dataSourceInstruction": "",
      "orgNodeOverrides": []
    }
  ],
  "attributeValues": []
}
EOF
```

> **Note:** Use the zero ID for `id` when creating new metrics. Enum fields use `{ "id": <int>, "name": "<name>" }` format and entity references use `{ "id": "<id>" }` format. See [Model Building Reference](../../reference/model-building.md) for enum ID lookup tables and payload templates.

**What to look for:**
- Success response: `{ "success": true, "id": "<new-id>" }`
- The returned ID is needed if other calculations will reference this metric

**If error:**
- `VALIDATION_ERROR` → Check JSON structure matches DTO format (common: using flat field names instead of nested objects)
- Duplicate names are allowed — the system won't warn you

---

### Step 8: Verify Creation

**Purpose:** Confirm the metric was created correctly.

**Command:**
```bash
cap model metrics get <new-id> --json
```

**Present confirmation:**
```
✅ Metric created successfully!

ID:             <id>
Name:           Diesel Consumption
Description:    Monthly diesel fuel usage for fleet vehicles
Type:           Input
Discipline:     Environmental > Energy
Unit:           litres (L)
Aggregation:    Sum
Data Interval:  Monthly

Next steps:
- Enter data: "Record 500 litres of diesel for Site A in March"
- View in list: cap model inputs list --json
```

---

## Example Interaction

**User:** "I need to track diesel consumption at our sites"

**Claude:** I'll help you create a diesel consumption metric. Let me gather a few details.

**Claude:** What discipline should this fall under? Here are your options:
```bash
cap masterdata disciplines list --json
```
→ Environmental > Energy seems like a good fit for fuel consumption. Is that right?

**User:** "Yes, Environmental/Energy"

**Claude:** And for the unit of measure, I see these options for volume:
- litres (L)
- kilolitres (kL)
- gallons (gal)

**User:** "Litres"

**Claude:** How should values aggregate? Since diesel consumption adds up (Site A's usage + Site B's usage = total usage), **Sum** is appropriate. Agreed?

**User:** "Yes"

**Claude:** Here's what I'll create:
```
Name:           Diesel Consumption
Discipline:     Environmental > Energy
Unit:           litres (L)
Aggregation:    Sum
Data Interval:  Monthly
```
Proceed?

**User:** "Yes"

**Claude:**
```bash
cat <<'EOF' | cap model inputs create --json
{
  "id": "<empty-id>",
  "name": "Diesel Consumption",
  "description": "Monthly diesel fuel usage for fleet vehicles",
  "reference": "",
  "discipline": { "id": "<id>" },
  "unitOfMeasure": { "id": "<id>" },
  "dataInterval": { "id": 2, "name": "Month" },
  "precision": 0,
  "orgStructureAggregationMethod": { "id": 0, "name": "Sum" },
  "timePeriodAggregationMethod": { "id": 1, "name": "Sum" },
  "allowForecastedData": false,
  "requireValidation": false,
  "requireDataCapture": false,
  "inputDataFeeds": [],
  "attributeValues": []
}
EOF
```

→ `{ "success": true, "id": "<id>" }`

Metric created! You can now enter data using: "Record 500 litres of diesel for Site A in March"

---

## Variations

### Creating a Calculation Instead

If user wants a formula-derived metric:
- "This sounds like a calculation (derived from other metrics) rather than an input. Would you like to create a calculation instead?"
- Calculations require additional decisions: calculation phase (Before/After Aggregation), formula syntax, and ID references
- See [Model Building Reference](../../reference/model-building.md#create-calculation) for the payload template, enum IDs, and the batch creation pattern for calculations that reference each other

### Bulk Creation

If user needs many similar metrics:
- "Would you like to create multiple metrics at once? I can help you build a JSON file for bulk import via `cap model inputs upload-excel`"

### Rate Metrics

If metric is a rate (per employee, per hour):
- "Rate metrics typically use **Average** aggregation. Sum would give meaningless results."
- Explain why: "2 injuries/million hours + 3 injuries/million hours ≠ 5 injuries/million hours"

---

## Related Recipes

- [Discover Tenant Structure](../exploration/discover-tenant-structure.md) — See what's configured ✅
- [Enter Data](../data-management/enter-data.md) — Record values for your new metric ✅
- [Talk to Your Data](../exploration/talk-to-your-data.md) — Analyze metric values ✅
