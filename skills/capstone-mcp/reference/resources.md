# MCP Resources Reference

> Auto-loaded contextual data that eliminates discovery tool calls.

---

## How Resources Work

Resources are **read-only data** that Claude can access automatically. They're loaded into context when relevant, providing background knowledge about the tenant without explicit tool calls.

| Behavior | Details |
|----------|---------|
| **Loaded by** | Claude Desktop (automatic, not user-triggered) |
| **Cache** | Model data: 5 min TTL; Data availability: 1 min TTL |
| **Invalidation** | On tenant switch (`SwitchTenant`), all caches clear |
| **Format** | JSON |

---

## Resource URIs

### `capstone://user/context`

Current user identity and tenant context.

**Content:**
- User name and email
- Current tenant name and ID
- User roles and permissions
- Default language

**Updates:** On authentication or tenant change.

**Use:** Verify authentication state. Understand what the user can access.

---

### `capstone://model/metrics`

Complete metric hierarchy for the current tenant.

**Content:**
- All metrics (Input and Calculation types)
- Metric names, descriptions, types
- Units of measure
- Data intervals
- Discipline assignments
- Parent-child relationships

**Cache TTL:** 5 minutes.

**Use:** Answer "what metrics exist?" without calling `model_metrics_list`. Find metric IDs for data queries.

---

### `capstone://model/organization`

Organization structure tree.

**Content:**
- All org nodes with names and levels
- Parent-child relationships (`parentId` and `parentName` per node)
- Tree paths (e.g., "Corporate > Region 1 > Site A")

**Cache TTL:** 5 minutes.

**Use:** Answer "what locations exist?" without calling `model_orgNodes_list`. Understand hierarchy depth and structure. Use `parentId`/`parentName` to determine direct parent-child relationships (e.g., "which nodes are direct children of Enterprise B?").

---

### `capstone://model/disciplines`

Discipline (metric category) hierarchy.

**Content:**
- All disciplines with names
- Parent-child relationships
- Category structure (e.g., Environmental > Energy > Electricity)

**Cache TTL:** 5 minutes.

**Use:** Understand how metrics are categorized. Map user questions about "environmental" or "social" to specific disciplines.

---

### `capstone://model/frameworks`

Reporting framework hierarchy.

**Content:**
- All frameworks (GRI, SASB, CDP, TCFD, etc.)
- Framework nodes representing disclosure topics
- Parent-child relationships

**Cache TTL:** 5 minutes.

**Use:** Understand which reporting standards are configured. Map metrics to framework disclosures.

---

### `capstone://data/availability`

Data availability by time period.

**Content:**
- Which time periods have data
- Coverage indicators per data interval

**Cache TTL:** 1 minute (shorter because data changes more frequently).

**Use:** Find periods with data before running queries. Avoid querying empty periods.

---

### `capstone://guide/tool-selection`

Decision guide for choosing the right Capstone tool based on user intent.

**Content:**
- **Quick decision matrix** — maps common needs (monthly totals, ad-hoc queries, visual charts, etc.) directly to the right tool and key parameter
- Step-by-step decision tree: SEE → DATA → special cases
- Common scenario walkthroughs (monthly summaries, weekly trends, site comparisons)
- Key parameter guide (dataInterval, time periods, orgNodeIds)
- Resource cross-references

**Cache TTL:** None (static content, never changes).

**Use:** Consult when deciding which tool to call for a user's question. Start with the decision matrix for fast routing. Especially useful for distinguishing between `data_computedValues_list` (template-based), `data_computedValues_query` (ad-hoc by metric ID), and `reporting_dashboards_getData` (full dashboard export).

---

## Resource Usage Strategy

### Before Tools

Check resources first to avoid unnecessary tool calls:

```
1. Check capstone://model/metrics → already know metric names and IDs
2. Check capstone://model/organization → already know org node IDs
3. Check capstone://data/availability → know which periods have data
4. Call data_computedValues_list → get the actual values (only 1 tool call)
```

Without resources, this would require 3 discovery tool calls before the data query.

### Fallback

If a resource returns an error (e.g., not authenticated), fall back to the corresponding tool:

| Resource | Fallback Tool |
|----------|--------------|
| `capstone://user/context` | `Whoami` |
| `capstone://model/metrics` | `model_metrics_list` |
| `capstone://model/organization` | `model_orgNodes_list` |
| `capstone://model/disciplines` | `model_disciplines_list` |
| `capstone://model/frameworks` | `model_frameworks_list` |
| `capstone://data/availability` | `data_availability` |

---

## See Also

- [SKILL.md](../SKILL.md) — Quick start with resources
- [Tools Reference](./tools.md) — Full tool documentation
