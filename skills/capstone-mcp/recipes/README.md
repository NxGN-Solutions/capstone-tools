# MCP Recipes

> Step-by-step workflow guides for Capstone MCP prompts and direct tool usage.

---

## Prompt vs Direct Tool Calls

| Situation | Approach |
|-----------|----------|
| Multi-step analysis with known pattern | Use a prompt (e.g., `/target-gap-analysis`) |
| Cross-metric or cross-location analysis | Use a prompt |
| Simple data retrieval | Direct tool call |
| Single entity lookup | Direct tool call |
| Data entry | Direct tool call (`data_inputValues_save`) |

---

## User Intent Routing

When the user's request doesn't map to a specific prompt, use this table to pick the right recipe:

| User Says | Recipe | Why |
|-----------|--------|-----|
| "What's in this tenant?" / "Show me the structure" | [Discover Structure](./exploration/discover-structure.md) | Explore org hierarchy, metrics, frameworks |
| "What are my emissions?" / "Show me [metric]" | [Talk to Your Data](./exploration/talk-to-data.md) | Answer natural-language metric questions |
| "What does the dashboard show?" | [Talk to Dashboard](./exploration/talk-to-dashboard.md) | Interpret dashboard widgets and sections |
| "Are we on track?" / "How are targets?" | [Target Gap Analysis](./analysis/target-gap-analysis.md) | Compare actuals vs targets |
| "How are emissions trending?" | [Find Trends](./analysis/find-trends.md) | Time-series direction and rate of change |
| "Any unusual data?" / "Outliers?" | [Spot Anomalies](./analysis/spot-anomalies.md) | Statistical outlier detection |
| "Compare Site A vs Site B" | [Compare Operations](./analysis/compare-operations.md) | Side-by-side location comparison |
| "What data is missing for Q1?" | [Check Completeness](./exploration/check-completeness.md) | Audit data quality and coverage |
| "Write an executive summary" | [Generate Narrative](./reporting/generate-narrative.md) | AI-written performance report |

---

## Recipe Index

### Analysis (4 recipes)

| Recipe | Prompt | Description |
|--------|--------|-------------|
| [Target Gap Analysis](./analysis/target-gap-analysis.md) | `/target-gap-analysis` | Compare actuals vs targets with status indicators |
| [Spot Anomalies](./analysis/spot-anomalies.md) | `/spot-anomalies` | Detect statistical outliers |
| [Compare Operations](./analysis/compare-operations.md) | `/compare-operations` | Compare two locations side-by-side |
| [Find Trends](./analysis/find-trends.md) | `/find-trends` | Time-series trend analysis |

### Exploration (4 recipes)

| Recipe | Prompt | Description |
|--------|--------|-------------|
| [Discover Structure](./exploration/discover-structure.md) | `/discover-structure` | Explore org hierarchy, metrics, frameworks |
| [Talk to Your Data](./exploration/talk-to-data.md) | _(direct tools)_ | Answer natural-language metric questions |
| [Talk to Dashboard](./exploration/talk-to-dashboard.md) | `/talk-to-dashboard` | Get insights from dashboard widgets |
| [Check Completeness](./exploration/check-completeness.md) | `/check-completeness` | Audit data quality and coverage |

### Reporting (1 recipe)

| Recipe | Prompt | Description |
|--------|--------|-------------|
| [Generate Narrative](./reporting/generate-narrative.md) | `/generate-narrative` | AI-written performance summary |

---

## Recipes Not Available in MCP v1

These CLI recipes don't have MCP equivalents yet. Use the [CLI](../../capstone-cli/recipes/README.md) for these workflows.

| CLI Recipe | Why Not in MCP |
|------------|---------------|
| Dashboard Comparison | Requires period selection UI |
| Deep Dashboard Analysis | Complex cross-metric correlation (deferred) |
| Create Metric Wizard | v1 is read-only |
| Create Widget Template | v1 is read-only |
| Enter Data | Use `data_inputValues_save` directly |
| Bulk Import Data | Excel operations deferred |
| Review Approvals | Change request workflow deferred |

---

## See Also

- [SKILL.md](../SKILL.md) — Quick prompt routing table
- [Glossary](../reference/glossary.md) — Term definitions with MCP tool references
- [Prompts Reference](../reference/prompts.md) — Prompt arguments and tool orchestration
- [CLI Recipes](../../capstone-cli/recipes/README.md) — CLI recipe equivalents
