# Recipe: Enter Data

> Record a single input value with proper validation and confirmation.

## When to Use

- "Record 1500 MWh for March"
- "Enter 250 litres of diesel at Site A"
- "Log this month's headcount as 42"
- "Submit [value] for [metric/location/period]"

## Required Context

Before starting, Claude needs to resolve:
- [ ] **Metric** — Which metric to record (by name or ID)
- [ ] **Location** — Which org node (by name or ID)
- [ ] **Period** — Which time period (e.g., "March 2025")
- [ ] **Value** — The numeric value to record

**Claude will ask for missing information or help resolve ambiguous references.**

---

## Step-by-Step

### Step 1: Identify the Metric

**Purpose:** Resolve metric name to ID.

**If user gave metric name:**
```bash
cap model inputs list --json
```

**Look for:**
- Exact match on name (case-insensitive)
- If multiple matches, present options
- If no match, suggest similar names

**Example resolution:**
```
User said: "diesel"
Found metrics containing "diesel":
1. Diesel Consumption (ID: abc123)
2. Diesel Fleet Usage (ID: def456)

"Which one do you mean?"
```

**If user gave ID:** Use directly.

**If no metric found:**
- "I couldn't find a metric matching '[name]'. Here are available Input <id>"
- Present list for selection

---

### Step 2: Identify the Location (Org Node)

**Purpose:** Resolve location name to ID.

**If user gave location name:**
```bash
cap masterdata org-nodes list --json
```

**Look for:**
- Match on node name
- Consider tree path for disambiguation (e.g., "Site A" under "Region 1" vs "Region 2")

**Example resolution:**
```
User said: "Site A"
Found org nodes matching "Site A":
1. Corporate > Region 1 > Site A (ID: org123)
2. Corporate > Region 2 > Site A (ID: org456)

"Which Site A? Region 1 or Region 2?"
```

**If no location found:**
- Present org hierarchy for selection
- Show leaf nodes where data is typically entered

---

### Step 3: Identify the Time Period

**Purpose:** Resolve period reference to specific period.

**If user gave period:**
```bash
cap data time-periods list --data-interval month --json
```

**Handle ambiguity:**
- "March" → "Which year? 2024 or 2025?"
- "Q1" → Check quarter type is configured, ask year
- "Last month" → Calculate from current date

**Period resolution examples:**
| User Says | Resolution |
|-----------|------------|
| "March" | Ask: "March 2024 or March 2025?" |
| "March 2025" | Find period with name "Mar 2025" or "March 2025" |
| "Q1 FY 25" | Find quarterly period matching |
| "This month" | Current month period |
| "Last month" | Previous month period |

**If period not found:**
- "That period doesn't exist. Available periods are..."
- Show most recent periods

---

### Step 4: Validate the Value

**Purpose:** Ensure value is valid before submission.

**Checks:**
- Value is numeric
- Value is within reasonable range (if known)
- Unit matches metric configuration

**If value seems unusual:**
- "You're entering 15000 kWh. Last month was 1500 kWh. Is this correct?"
- Allow override if user confirms

---

### Step 5: Confirm Before Saving

**Purpose:** User must explicitly confirm the save action.

**Present summary:**
```
Ready to save:

Metric:     Diesel Consumption
Location:   Corporate > Region 1 > Site A
Period:     March 2025
Value:      1,500 litres

⚠️  This will overwrite any existing value for this combination.

Proceed? [Yes/No]
```

**Wait for explicit confirmation.**

---

### Step 6: Save the Value

**Purpose:** Execute the save command.

> **Important:** The save endpoint requires the full business key (`metric`, `orgNode`, `timePeriodType`, `startDate`) for every input value, even when updating an existing record by ID. The `id` field uses the zero ID for new values, or an existing input value ID for updates.

**Command:**
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

**If successful:**
```
✅ Saved 1 input value(s).
```

---

### Step 7: Handle Errors

**Common errors and recovery:**

#### Locked Period

```
❌ ERROR: Period is locked

The period "March 2025" is locked for data entry.

Options:
1. Submit a Change Request to modify this value
2. Contact an administrator to unlock the period
3. Enter data for a different period

Would you like to create a Change Request instead?
```

→ Guide to Change Request workflow if user says yes.

#### Validation Required

```
⚠️ Value saved but requires validation

The value has been saved with status "Pending Validation".
An approver must validate this entry before it's final.

Current validation level: Level 1 (Pending)
```

#### Permission Denied

```
❌ ERROR: Permission denied

You don't have permission to enter data for:
- Location: Site A
- Discipline: Environmental

Contact your administrator to request data permissions.
```

---

## Example Interaction

**User:** "Record 1500 litres of diesel for Site A in March"

**Claude:** Let me find that metric and location.

```bash
cap model inputs list --json | grep -i diesel
# Found: Diesel Consumption (abc123)

cap masterdata org-nodes list --json | grep -i "site a"
# Found: Site A (org789)
```

**Claude:** I found the metric "Diesel Consumption" and "Site A".

For the period "March" — which year?
1. March 2024
2. March 2025

**User:** "2025"

**Claude:**
```
Ready to save:

Metric:     Diesel Consumption
Location:   Corporate > Region 1 > Site A
Period:     March 2025
Value:      1,500 litres

Proceed? [Yes/No]
```

**User:** "Yes"

**Claude:**
```bash
cat <<'EOF' | cap data input-values save --json
{
  "inputValues": [
    {
      "id": "<empty-id>",
      "value": 1500,
      "metric": { "id": "abc123" },
      "orgNode": { "id": "org789" },
      "timePeriodType": { "id": 2, "name": "Month" },
      "startDate": "2025-03-01T00:00:00Z"
    }
  ]
}
EOF
```

✅ Saved 1 input value(s).

---

## Variations

### Batch Entry (Multiple Values)

If user has multiple values:
- "Would you like to enter more values for the same metric/location?"
- Loop back to Step 3 for additional periods
- Or suggest Excel import for many values

### Programmatic Batch Save (10+ values)

When populating many values across org nodes and time periods (e.g., plan metrics, seed data), generate JSON programmatically and pipe it to the save command:

1. **Gather IDs** — metric, org node, and time period start dates:
   ```bash
   cap model inputs list --json                           # metric IDs
   cap masterdata org-nodes list --json                   # org node IDs
   cap data time-periods list --data-interval week --json # weekly startDates
   ```

2. **Generate payload** — build JSON with all values using zero IDs:
   ```bash
   python3 -c "
   import json
   ZERO = '<empty-id>'
   values = []
   for org_id, rate in [('<org-1>', 300), ('<org-2>', 320)]:
       for start_date in ['2026-02-09T00:00:00Z', '2026-02-16T00:00:00Z']:
           values.append({
               'id': ZERO, 'value': rate,
               'metric': {'id': '<metric-id>'},
               'orgNode': {'id': org_id},
               'timePeriodType': {'id': 1, 'name': 'Week'},
               'startDate': start_date
           })
   print(json.dumps({'inputValues': values}))
   " | cap data input-values save --json
   ```

3. **Verify** — spot-check representative periods:
   ```bash
   cap data input-values list --template <id> --data-interval week \
     --periods "(W7) Feb 9 2026,(W9) Feb 23 2026" --json
   ```

**When to use this vs Excel import:**
- **Programmatic batch:** Values follow a pattern (same rate repeated, site-varied splits). Faster, scriptable, no file management.
- **Excel import:** Irregular values, user already has a spreadsheet, or non-technical user.

### Different Metric Type

If metric is a Calculation (not Input):
- "This is a calculation metric — its value is computed from a formula, not entered directly."
- Suggest entering the source Input metrics instead

### Historical Correction

If entering data for a past period:
- Check if period is locked
- Warn if existing value will be overwritten
- Suggest Change Request if locked

---

## Related Recipes

- [Bulk Import Data](./bulk-import.md) — Import many values via Excel ✅
- [Review Approvals](./review-approvals.md) — Approve pending values ✅
- [Create Metric Wizard](../configuration/create-metric.md) — Create new metrics ✅
- [Talk to Your Data](../exploration/talk-to-your-data.md) — Analyze entered values ✅
