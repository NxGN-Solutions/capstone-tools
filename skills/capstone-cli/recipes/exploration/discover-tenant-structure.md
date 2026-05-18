# Recipe: Discover Tenant Structure

> Explore what's configured in your Capstone tenant: org hierarchy, categories, frameworks, and metrics.

## When to Use

- "What's available in this tenant?"
- "Show me the organization structure"
- "What metrics exist?"
- "What frameworks are configured?"
- "Give me an overview of the data model"

## Required Context

Before starting, Claude should know:
- [x] User is authenticated (run `cap auth whoami` to verify)
- [ ] Which areas to focus on (default: all)

---

## Step-by-Step

### Step 1: Verify Authentication

**Purpose:** Ensure we can access the tenant before running discovery commands.

**Command:**
```bash
cap auth whoami --json
```

**What to look for:**
- `tenantName` — The active tenant being explored
- `userName` — Confirms authenticated user

**If error:**
- `AUTH_REQUIRED` → Run `cap auth login` first
- `TOKEN_EXPIRED` → Re-authenticate with `cap auth login`

---

### Step 2: Discover Organizational Structure

**Purpose:** Understand the location/facility hierarchy where data is captured.

**Command:**
```bash
cap masterdata org-nodes list --json
```

**What to look for:**
- Total node count
- Hierarchy depth (how many levels)
- Root nodes (Level 0 items)
- Leaf nodes (no children — where data is typically entered)

**Key fields in output:**
```json
{
  "items": [
    {
      "id": "...",
      "name": "Corporate",
      "level": 0,
      "treePath": "Corporate"
    },
    {
      "id": "...",
      "name": "Site A",
      "level": 1,
      "treePath": "Corporate > Site A"
    }
  ]
}
```

**If empty:**
- "No organizational structure is configured. This tenant may need initial setup."

**Insight:** Org nodes represent the physical/logical places where data is captured. Deeper hierarchies allow more granular reporting.

---

### Step 3: Discover Disciplines (Categories)

**Purpose:** Understand how metrics are categorized (Environmental, Social, Governance, etc.).

**Command:**
```bash
cap masterdata disciplines list --json
```

**What to look for:**
- Discipline names at each level
- Hierarchy structure (disciplines can be nested)
- Common patterns: Environmental > Energy, Environmental > Emissions

**If empty:**
- "No disciplines configured. Metrics won't have category groupings."

**Insight:** Disciplines help organize metrics by topic. Most ESG setups have Environmental, Social, and Governance at the top level with detailed subcategories.

---

### Step 4: Discover Frameworks (Reporting Standards)

**Purpose:** Identify which external reporting standards are configured.

**Command:**
```bash
cap masterdata frameworks list --json
```

**What to look for:**
- Framework names (GRI, SASB, CDP, TCFD, etc.)
- Framework nodes representing disclosure topics

**If empty:**
- "No frameworks configured. This may be a custom reporting setup without external standards."

**Insight:** Frameworks enable mapping metrics to specific disclosure requirements. Common in ESG reporting.

---

### Step 5: Discover Metrics

**Purpose:** Understand what's being measured.

**Command:**
```bash
cap model metrics list --json
```

**What to look for:**
- Total metric count
- Type breakdown (Input vs Calculation)
- Discipline distribution
- Data interval patterns (Monthly, Quarterly, Annual)

**To separate inputs from calculations:**
```bash
cap model inputs list --json        # Manual/automated data entry points
cap model calculations list --json  # Formula-derived metrics
```

**Key fields in output:**
```json
{
  "items": [
    {
      "id": "...",
      "name": "Electricity Consumption",
      "metricType": "Input",
      "discipline": { "name": "Environmental" },
      "dataInterval": "Monthly"
    }
  ]
}
```

**If empty:**
- "No metrics configured. The data model needs to be set up before data can be captured."

---

### Step 6: Synthesize Results

**Combine the data into a structured summary:**

Present to user in this format:

```
## Tenant Overview: {tenantName}

### Organizational Structure
- **Total Nodes:** {count}
- **Hierarchy Depth:** {max_level + 1} levels
- **Root Nodes:** {list of level 0 names}
- **Leaf Nodes (data entry points):** {count} locations

### Categories (Disciplines)
- {discipline 1}
  - {subdiscipline 1a}
  - {subdiscipline 1b}
- {discipline 2}
  - {subdiscipline 2a}

### Reporting Frameworks
- {framework 1} ({node_count} disclosure topics)
- {framework 2} ({node_count} disclosure topics)
OR: No external frameworks configured

### Metrics
- **Total:** {count} metrics
- **Inputs (manual data):** {input_count}
- **Calculations (formulas):** {calc_count}
- **By Discipline:**
  - Environmental: {count}
  - Social: {count}
  - Governance: {count}
- **By Interval:**
  - Monthly: {count}
  - Quarterly: {count}
  - Annual: {count}
```

---

## Example Interaction

**User:** "What's in this Capstone tenant?"

**Claude:** Let me explore your tenant structure.

```bash
cap auth whoami --json
# → Tenant: Acme Corporation

cap masterdata org-nodes list --json
# → 15 nodes, 3 levels deep

cap masterdata disciplines list --json
# → Environmental, Social, Governance

cap masterdata frameworks list --json
# → GRI, SASB

cap model metrics list --json
# → 42 metrics total
```

**Claude:**

## Tenant Overview: Acme Corporation

### Organizational Structure
- **Total Nodes:** 15
- **Hierarchy Depth:** 3 levels
- **Root Nodes:** Corporate HQ
- **Leaf Nodes:** 12 locations (Site A, Site B, ...)

### Categories (Disciplines)
- Environmental
  - Energy
  - Emissions
  - Water
- Social
  - Health & Safety
  - Diversity
- Governance
  - Ethics
  - Compliance

### Reporting Frameworks
- GRI (48 disclosure topics)
- SASB (32 disclosure topics)

### Metrics
- **Total:** 42 metrics
- **Inputs:** 35 (manual data entry)
- **Calculations:** 7 (formula-derived)
- **By Discipline:**
  - Environmental: 25
  - Social: 12
  - Governance: 5

Would you like to explore any area in more detail?

---

## Variations

### Focus on Specific Area

If user only wants one area:
- "Just show me the org structure" → Skip to Step 2, present only org summary
- "What metrics exist?" → Skip to Step 5, focus on metric details
- "What frameworks do we use?" → Skip to Step 4

### Deep Dive Options

After initial discovery, offer:
- "Show me metrics in [discipline]" → Filter metrics by discipline
- "Show the hierarchy under [org node]" → Filter org nodes by parent
- "List all Input metrics" → Use `cap model inputs list`

---

## Related Recipes

- [Talk to Your Data](./talk-to-your-data.md) — Analyze specific metrics ✅
- [Create Metric Wizard](../configuration/create-metric.md) — Add new metrics ✅
- [Enter Data](../data-management/enter-data.md) — Record values ✅
