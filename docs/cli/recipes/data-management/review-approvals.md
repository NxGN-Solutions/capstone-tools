# Recipe: Review Approvals

> Process pending change requests through the validation workflow.

## When to Use

- "What needs approval?"
- "Review pending changes"
- "Any change requests waiting?"
- "Process the approval queue"
- "Approve/reject change requests"

## Required Context

Before starting, Claude needs:
- [ ] User has approval permissions (will error if not)
- [ ] Data interval for the queue being reviewed (`day|week|month|quarter|year`)
- [ ] Optional: Specific period or location to filter

---

## Step-by-Step

### Step 1: List Pending Change Requests

**Purpose:** See what's waiting for approval.

**Command:**
```bash
cap data change-requests list \
  --data-interval month \
  --status "validation-required,validation-in-progress" \
  --json
```

**If none pending:**
```
No pending change requests.

All caught up! Check back later or run this command to see resolved requests:
cap data change-requests list --data-interval month --status approved --json
```

**If requests exist, present summary:**
```
📋 Pending Change Requests (5 items)

┌────────┬────────────────────────┬──────────────┬─────────────┬─────────────────┬───────────────────┐
│ ID     │ Metric                 │ Location     │ Period      │ Change          │ Submitted By      │
├────────┼────────────────────────┼──────────────┼─────────────┼─────────────────┼───────────────────┤
│ cr-001 │ Diesel Consumption     │ Site A       │ Mar 2025    │ 1,500 → 1,650   │ Jane Smith        │
│ cr-002 │ Electricity Usage      │ Site B       │ Mar 2025    │ (new) → 12,500  │ Bob Jones         │
│ cr-003 │ Water Consumption      │ Site A       │ Feb 2025    │ 2,300 → 2,100   │ Jane Smith        │
│ cr-004 │ Safety Incidents       │ Site C       │ Q1 FY 25    │ 3 → 2           │ Alice Williams    │
│ cr-005 │ Training Hours         │ Corporate    │ Mar 2025    │ 450 → 475       │ Bob Jones         │
└────────┴────────────────────────┴──────────────┴─────────────┴─────────────────┴───────────────────┘

Would you like to:
1. Review all requests in order
2. Select specific requests to review
3. Filter by submitter or location
```

---

### Step 2: Review Individual Request

**Purpose:** Show full details before making a decision.

**Command:**
```bash
cap data change-requests get <id> --json
```

**Present details:**
```
═══════════════════════════════════════════════════════════════
CHANGE REQUEST: cr-001
═══════════════════════════════════════════════════════════════

Metric:         Diesel Consumption
Location:       Corporate > Region 1 > Site A
Period:         March 2025
Unit:           litres

┌─────────────┬───────────────┐
│ Old Value   │ 1,500         │
│ New Value   │ 1,650         │
│ Difference  │ +150 (+10%)   │
└─────────────┴───────────────┘

Submitted By:   Jane Smith
Submitted At:   2025-03-15 14:30
Reason:         Discovered unreported fleet refueling

Comments:       "Found records from secondary fuel card
                that weren't included in original submission."

═══════════════════════════════════════════════════════════════

Decision: [Approve / Reject]
```

---

### Step 3: Get User Decision

**Purpose:** Collect approve/reject decision with confirmation.

**If APPROVE:**
```
You are approving:
- Change Request: cr-001
- Change: 1,500 → 1,650 litres
- For: Diesel Consumption at Site A (March 2025)

Optional comment for audit trail: _____________

Confirm approval? [Yes/No]
```

**If REJECT:**
```
You are rejecting:
- Change Request: cr-001
- Requested change: 1,500 → 1,650 litres

⚠️  A rejection reason is required for audit purposes.

Rejection reason: _____________

Confirm rejection? [Yes/No]
```

---

### Step 4: Execute Decision

**Purpose:** Process the approval or rejection.

**For Approval:**
```bash
cap data change-requests validate <id> --result approve --comments "Verified supporting documentation" --json
```

**For Rejection:**
```bash
cap data change-requests validate <id> --result reject --comments "Supporting documentation insufficient" --json
```

**Confirmation:**
```
✅ Change Request cr-001 APPROVED

The value for Diesel Consumption at Site A (March 2025)
has been updated from 1,500 to 1,650 litres.

───────────────────────────────────────────────────────
```

Or:
```
❌ Change Request cr-001 REJECTED

The value for Diesel Consumption at Site A (March 2025)
remains at 1,500 litres.

Reason provided: "Supporting documentation insufficient"

───────────────────────────────────────────────────────
```

---

### Step 5: Continue or Summarize

**If more requests:**
```
4 change requests remaining.

Next: cr-002 - Electricity Usage at Site B (new value: 12,500)

Continue reviewing? [Yes/No]
```

**If done:**
```
═══════════════════════════════════════════════════════════════
REVIEW SESSION COMPLETE
═══════════════════════════════════════════════════════════════

Processed: 5 change requests

┌─────────────┬─────────┐
│ Approved    │ 3       │
│ Rejected    │ 2       │
│ Remaining   │ 0       │
└─────────────┴─────────┘

Approved requests:
• cr-001: Diesel Consumption at Site A (1,500 → 1,650)
• cr-002: Electricity Usage at Site B (new → 12,500)
• cr-005: Training Hours at Corporate (450 → 475)

Rejected requests:
• cr-003: Water Consumption at Site A (reason: Incorrect period)
• cr-004: Safety Incidents at Site C (reason: Awaiting investigation)

═══════════════════════════════════════════════════════════════
```

---

## Example Interaction

**User:** "What needs approval?"

**Claude:**
```bash
cap data change-requests list \
  --data-interval month \
  --status "validation-required,validation-in-progress" \
  --json
```

Found 3 pending change requests:

| ID | Metric | Location | Change | Submitted By |
|----|--------|----------|--------|--------------|
| cr-001 | Diesel Consumption | Site A | 1,500 → 1,650 | Jane Smith |
| cr-002 | Electricity Usage | Site B | (new) → 12,500 | Bob Jones |
| cr-003 | Water Consumption | Site A | 2,300 → 2,100 | Jane Smith |

Would you like to review them?

**User:** "Yes, start with cr-001"

**Claude:**
```bash
cap data change-requests get <change-request-id> --json
```

**Change Request cr-001**

| Field | Value |
|-------|-------|
| Metric | Diesel Consumption |
| Location | Site A |
| Period | March 2025 |
| Old Value | 1,500 litres |
| New Value | 1,650 litres |
| Change | +150 (+10%) |
| Reason | Discovered unreported fleet refueling |
| Submitted By | Jane Smith |

Do you want to **Approve** or **Reject** this change?

**User:** "Approve, the fuel records check out"

**Claude:**
```bash
cap data change-requests validate <change-request-id> --result approve --comments "Fuel records verified" --json
```

✅ **Approved!** Diesel Consumption at Site A (March 2025) updated to 1,650 litres.

Next: cr-002 - Electricity Usage at Site B. Continue?

---

## Variations

### Bulk Approval

If user wants to approve multiple similar requests:
```
"These 3 requests are all from Jane Smith for March 2025.
Would you like to:
1. Review each individually
2. Approve all at once (with shared comment)
3. Reject all at once"
```

### Filtering

If user wants specific requests:
```bash
# By submitter
cap data change-requests list --data-interval month --status "validation-required,validation-in-progress" --json | filter by submitter

# By period
cap data change-requests list --data-interval month --status "validation-required,validation-in-progress" --periods "Mar 2025" --json

# By location
cap data change-requests list --data-interval month --status "validation-required,validation-in-progress" --org-nodes <id> --json
```

### Multi-Level Approval

If tenant uses multiple validation levels:
```
⚠️  Multi-level approval required

This change request requires Level 2 approval.
Current status: Level 1 approved by John Manager

You are providing Level 2 approval.

Proceed? [Yes/No]
```

---

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| "Permission denied" | No approval permissions | Contact administrator |
| "Already processed" | Request was handled by someone else | Refresh list |
| "Rejection reason required" | Tenant requires rejection reasons | Provide a reason |
| "Invalid validation level" | Wrong approval level | Check validation level config |

---

## Related Recipes

- [Enter Data](./enter-data.md) — Submit new data ✅
- [Bulk Import Data](./bulk-import.md) — Import data via Excel ✅
- [Discover Tenant Structure](../exploration/discover-tenant-structure.md) — View org structure ✅
