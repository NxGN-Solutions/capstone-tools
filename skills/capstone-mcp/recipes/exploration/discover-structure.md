# Recipe: Discover Structure

> Explore what's configured in your Capstone tenant.

## When to Use

- "What's available in this tenant?"
- "Show me the organization structure"
- "What metrics exist?"
- "What frameworks are configured?"
- "Give me an overview of the data model"

## Prompt Shortcut

```
/discover-structure
```

No arguments — explores the complete tenant structure.

---

## Required Context

| What's Needed | Source | If Missing |
|--------------|--------|------------|
| Authentication | `Whoami` tool | Call `Login` first |
| Focus area | User request | Default: explore everything |

---

## Workflow

### 1. Get organization hierarchy

```
model_orgNodes_list()
```

From the response `gridRows`, note:
- Total node count
- Hierarchy depth (max `level` value + 1)
- Root nodes (`level: 0`)
- Leaf nodes (nodes not appearing as anyone's `parentId`)

### 2. Get disciplines

```
model_disciplines_list()
```

Extract category names and tree structure from `gridRows`.

### 3. Get frameworks

```
model_frameworks_list()
```

Extract framework names and node counts from `gridRows`.

### 4. Get metrics overview

```
model_metrics_list()
```

From `gridRows`, calculate:
- Total metric count
- Count by `metricType` (Input vs Calculation)
- Count by discipline
- Count by `dataInterval`

### 5. Summarize the tenant structure

Combine all data into a structured overview (see Present Results below).

### 6. Suggest analyses

Based on the structure, recommend which prompts would be most valuable:
- If metrics have targets → suggest `/target-gap-analysis`
- If multiple locations → suggest `/compare-operations`
- If dashboards exist → suggest `/talk-to-dashboard`

---

## Present Results

```
## Tenant Overview: Example Organization

### Organizational Structure
- **Total Nodes:** 15
- **Hierarchy Depth:** 3 levels
- **Root Nodes:** Corporate HQ
- **Leaf Nodes:** 12 locations

### Categories (Disciplines)
- Environmental
  - Energy, Emissions, Water
- Social
  - Health & Safety, Diversity
- Governance
  - Ethics, Compliance

### Reporting Frameworks
- GRI (48 disclosure topics)
- SASB (32 disclosure topics)

### Metrics
- **Total:** 42 metrics
- **Inputs:** 35 (manual data)
- **Calculations:** 7 (formula-derived)
- **By Discipline:** Environmental: 25, Social: 12, Governance: 5

### Suggested Analyses
- Use /target-gap-analysis to check metrics with targets
- Use /find-trends to see 12-month trends
- Use /talk-to-dashboard to explore configured dashboards
```

---

## Example Interaction

**User:** "What's in this Capstone tenant?"

**Claude:**
```
Whoami()
→ Tenant: Example Organization, User: user@example.com

model_orgNodes_list()
→ 15 nodes, 3 levels deep (Corporate → 3 Regions → 12 Sites)

model_disciplines_list()
→ Environmental (Energy, Emissions, Water), Social (H&S, Diversity), Governance (Ethics)

model_frameworks_list()
→ GRI (48 nodes), SASB (32 nodes)

model_metrics_list()
→ 42 metrics total: 35 Input, 7 Calculation
```

**Claude:**

## Tenant Overview: Example Organization

### Organizational Structure
- **Total Nodes:** 15
- **Hierarchy:** Corporate → 3 Regions → 12 Sites
- **Data Entry Points:** 12 leaf locations

### Metrics
- **42 total:** 35 Input metrics + 7 Calculations
- **Environmental:** 25 metrics (energy, emissions, water, waste)
- **Social:** 12 metrics (safety, training, diversity)
- **Governance:** 5 metrics (ethics, compliance)

### Reporting Frameworks
- GRI (48 disclosure topics)
- SASB (32 disclosure topics)

### What You Can Do Next
- `/target-gap-analysis` — Check performance against targets
- `/talk-to-dashboard` — Analyze a specific dashboard
- `/find-trends` — See how metrics are trending
- Ask me about any specific metric, location, or dashboard

---

## Variations

### Focus on specific area

If user asks about a single area ("Just show metrics"), skip to that step and provide more detail.

### Deep dive

After overview, offer to drill into specific areas:
- "Show metrics in Environmental discipline" → filter `model_metrics_list` by discipline
- "Show the hierarchy under Region 1" → filter `model_orgNodes_list` by parent

---

## Edge Cases

- **Empty tenant:** No org nodes or metrics configured — explain the tenant needs setup.
- **Large tenant:** Hundreds of metrics — summarize by discipline rather than listing all.

---

## CLI Equivalent

[CLI: Discover Tenant Structure](../../../capstone-cli/recipes/exploration/discover-tenant-structure.md)
