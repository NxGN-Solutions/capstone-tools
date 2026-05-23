# Recipe: Generate Narrative

> Write a human-readable executive summary of performance.

## When to Use

- "Write a summary of Q1 performance"
- "Executive summary for the board"
- "Narrative report for [period]"
- "Put this data into words"

## Required Context

Before starting, Claude needs:
- [ ] **Period** — What time frame to summarize
- [ ] **Focus** — All metrics or specific areas
- [ ] **Audience** — Board, management, external (tone adjustment)
- [ ] **Comparison** — Prior period, prior year, targets

---

## Step-by-Step

### Step 1: Identify Key Metrics

**Purpose:** Select what to include in narrative.

**If user specified:**
Use those metrics/disciplines.

**If not specified, prioritize:**
- Top 5 metrics by importance (if flagged)
- One from each discipline
- Any with targets
- Any with significant changes

---

### Step 2: Resolve Report Template

**Purpose:** Find a report template that includes the narrative metrics.

**Command:**
```bash
cap templates report-templates list --json
```

---

### Step 3: Gather Performance Data

**Current period:**
```bash
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json
```

**Comparison periods:**
```bash
cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval quarter \
  --periods "Q4 FY 24" \
  --json

cap reporting computed-values list \
  --template <report-template-id> \
  --data-interval quarter \
  --periods "Q1 FY 24" \
  --json
```

---

### Step 4: Calculate Statistics

**For each metric:**
- Current value
- Change from prior period
- Change from prior year
- Target achievement (if applicable)

**Aggregate:**
- Overall performance direction
- Highlight achievements
- Identify concerns

---

### Step 5: Generate Narrative

**Structure:**

**Paragraph 1: Executive Summary (2-3 sentences)**
```
"In Q1 2025, [Organization] demonstrated [overall performance
characterization]. Key achievements included [highlight], while
[area] requires continued focus."
```

**Paragraph 2: Key Highlights (bullets)**
```
Highlights:
• [Achievement 1 with specific number]
• [Achievement 2 with specific number]
• [Achievement 3 with specific number]
```

**Paragraph 3: Areas of Concern (bullets)**
```
Areas Requiring Attention:
• [Concern 1 with context]
• [Concern 2 with recommended action]
```

**Paragraph 4: Outlook (1-2 sentences)**
```
"Looking ahead, [projection or key initiative]. The organization
remains [on track/focused on] achieving its [annual/strategic] goals."
```

---

### Step 6: Present Narrative

**Tone:** Professional, factual, action-oriented

**Balance:** Achievements and concerns (avoid all positive or all negative)

**Numbers:** Include key figures without overwhelming

---

## Example Interaction

**User:** "Write an executive summary for Q1 ESG performance"

**Claude:**
```bash
cap model metrics list --json
# Selecting key ESG metrics

cap templates report-templates list --json
# Found: ESG Report (rpt-123)

cap reporting computed-values list \
  --template rpt-123 \
  --data-interval quarter \
  --periods "Q1 FY 25" \
  --json

cap reporting computed-values list \
  --template rpt-123 \
  --data-interval quarter \
  --periods "Q4 FY 24" \
  --json

cap reporting computed-values list \
  --template rpt-123 \
  --data-interval quarter \
  --periods "Q1 FY 24" \
  --json
```

**Claude:**

---

## Q1 FY 2025 ESG Performance Summary

In the first quarter of FY 2025, Example Organization continued its trajectory toward sustainability goals, achieving notable reductions in greenhouse gas emissions while maintaining strong safety performance. However, water consumption and waste diversion rates present opportunities for improvement in the coming quarters.

### Key Highlights

• **Emissions reduced 8%** from Q4 FY 2024, keeping the company on track for its 2030 net-zero commitment. This represents a 22% reduction compared to Q1 FY 2024.

• **Zero recordable safety incidents** for the third consecutive quarter, reflecting the success of the enhanced safety training program launched in July 2024.

• **Renewable energy adoption reached 45%** of total electricity consumption, up from 38% in Q4, driven by the completion of the Site B solar installation.

### Areas Requiring Attention

• **Water consumption increased 12%** quarter-over-quarter, primarily due to expanded manufacturing operations at Site C. The sustainability team is evaluating water recycling technologies for Q2 implementation.

• **Waste diversion rate declined to 68%** from 74%, below the 75% target. Contributing factors include packaging changes from new suppliers. Procurement is engaging vendors on sustainable packaging alternatives.

### Outlook

Looking ahead, Q2 priorities include commissioning the water recycling system at Site C and completing the supplier sustainability assessments. With continued focus on these areas, the organization remains positioned to achieve its annual environmental targets and maintain its industry-leading safety record.

---

## Variations

### Board-Level Summary

More concise, focus on strategic implications:
```
"Q1 performance supports strategic sustainability positioning.
Key risk: water consumption trajectory."
```

### Detailed Report

Expand with specific data tables, trend charts, location breakdowns.

### External/Public

Focus on achievements, position challenges as "opportunities," reference frameworks (GRI, CDP).

---

## Related Recipes

- [Talk to Your Data](../exploration/talk-to-your-data.md) — Analyze underlying data
- [Target Gap Analysis](../analysis/target-gap-analysis.md) — Target status
- [Find Trends](../analysis/find-trends.md) — Trend context
