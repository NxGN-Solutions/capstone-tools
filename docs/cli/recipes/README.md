# CLI Recipes

> Step-by-step workflows for common Capstone tasks.

---

## Platform Support

Recipes work on **Windows (PowerShell), macOS, and Linux**. All examples use `cap` as the command name — substitute your invocation method (e.g., `.\cap.exe` on Windows, `dotnet run --project ...` for source checkouts).

For shell-specific syntax translation (environment variables, JSON input, file paths), see the **[Cross-Platform CLI Guide](../reference/platform-guide.md)**.

## How Recipes Work

Recipes are **step-by-step guides** that Claude follows to accomplish multi-step tasks. Each recipe:

1. **Checks prerequisites** — What information is needed before starting
2. **Executes commands** — In the right order with error handling
3. **Synthesizes results** — Combines data into meaningful insights

### When to Use Recipes vs Direct Commands

| Situation | Approach |
|-----------|----------|
| Multi-step analysis | Use a recipe |
| Creating entities with dependencies | Use a recipe |
| Simple data retrieval | Direct command |
| Single entity lookup | Direct command |

### Recipe Format

Each recipe includes:
- **When to Use** — Trigger phrases and scenarios
- **Required Context** — Information needed (will ask if missing)
- **Step-by-Step** — Commands with expected outputs
- **Example Interaction** — Complete worked example

---

## Recipe Categories

### Exploration (Discovery)

| Recipe | Description | Status |
|--------|-------------|--------|
| [Discover Tenant Structure](./exploration/discover-tenant-structure.md) | Explore org hierarchy, disciplines, frameworks, metrics | ✅ Available |
| [Talk to Your Data](./exploration/talk-to-your-data.md) | Analyze specific metrics over time | ✅ Available |
| [Talk to Your Dashboard](./exploration/talk-to-your-dashboard.md) | Get insights from dashboard widgets | ✅ Available |
| [Check Data Completeness](./exploration/check-completeness.md) | Audit data quality and coverage | ✅ Available |

### Analysis

| Recipe | Description | Status |
|--------|-------------|--------|
| [Compare Operations](./analysis/compare-operations.md) | Compare Site A vs Site B | ✅ Available |
| [Dashboard Comparison](./analysis/dashboard-comparison.md) | Compare dashboard data across periods or locations | ✅ Available |
| [Deep Dashboard Analysis](./analysis/deep-dashboard-analysis.md) | Comprehensive cross-metric pattern discovery | ✅ Available |
| [Find Trends](./analysis/find-trends.md) | Time-series trend analysis | ✅ Available |
| [Spot Anomalies](./analysis/spot-anomalies.md) | Detect outliers and unusual values | ✅ Available |
| [Target Gap Analysis](./analysis/target-gap-analysis.md) | Actual vs target performance | ✅ Available |

### Reporting

| Recipe | Description | Status |
|--------|-------------|--------|
| [Verify Data](./reporting/verify-data.md) | Confirm widgets, dashboards, and reports return expected data | ✅ Available |
| [Generate Narrative](./reporting/generate-narrative.md) | AI-written performance summary | ✅ Available |

### Configuration

| Recipe | Description | Status |
|--------|-------------|--------|
| [Create Metric Wizard](./configuration/create-metric.md) | Guided metric creation | ✅ Available |
| [Create Calculation](./configuration/create-calculation.md) | Formula-based derived metrics | ✅ Available |
| [Create Widget Template](./configuration/create-widget-template.md) | Create InfoCard, PieChart, XYChart, and Table widgets | ✅ Available |
| [Build Dashboard](./configuration/build-dashboard.md) | Assemble widgets into a dashboard | ✅ Available |
| [Export to Excel](./configuration/export-to-excel.md) | Bulk export/import via Excel | ✅ Available |
| [Set Up Data Source](./configuration/set-up-data-source.md) | Configure external data integrations | ✅ Available |

### Data Management

| Recipe | Description | Status |
|--------|-------------|--------|
| [Seed Tenant](./data-management/seed-tenant.md) | Full tenant seeding from Excel fixtures | ✅ Available |
| [Enter Data](./data-management/enter-data.md) | Manual data entry workflow | ✅ Available |
| [Bulk Import Data](./data-management/bulk-import.md) | Excel upload workflow | ✅ Available |
| [Review Approvals](./data-management/review-approvals.md) | Change request approval workflow | ✅ Available |

---

## Selecting a Recipe

### By User Intent

| User Says... | Recipe |
|--------------|--------|
| "Analyze our [metric] data..." | Talk to Your Data |
| "What insights from the [dashboard]..." | Talk to Your Dashboard |
| "Analyze the entire [dashboard]..." | Deep Dashboard Analysis |
| "Find correlations across all metrics..." | Deep Dashboard Analysis |
| "How does Q1 compare to Q4..." | Dashboard Comparison |
| "Compare [location A] vs [location B] dashboard..." | Dashboard Comparison |
| "Compare [location A] vs [location B]..." | Compare Operations |
| "How is [metric] trending..." | Find Trends |
| "Any unusual values in..." | Spot Anomalies |
| "Are we on track for [target]..." | Target Gap Analysis |
| "What data is missing for..." | Check Data Completeness |
| "Does this widget have data..." | Verify Data |
| "Verify the dashboard shows values..." | Verify Data |
| "Check if my calculation is computing..." | Verify Data |
| "Test the widget I just created..." | Verify Data |
| "Write a summary of..." | Generate Narrative |
| "What metrics/orgs/frameworks exist..." | Discover Tenant Structure |
| "I need to create a metric for..." | Create Metric Wizard |
| "Create a calculation for..." | Create Calculation |
| "Create a chart/widget for..." | Create Widget Template |
| "Add an InfoCard/PieChart/XYChart/Table..." | Create Widget Template |
| "Build a dashboard for..." | Build Dashboard |
| "Export metrics to Excel..." | Export to Excel |
| "Connect to our database/MQTT..." | Set Up Data Source |
| "Record/enter [value] for..." | Enter Data |
| "Import this Excel file..." | Bulk Import Data |
| "Seed/set up a fresh tenant..." | Seed Tenant |
| "Upload all the Tharisa fixtures..." | Seed Tenant |
| "Review/approve pending requests..." | Review Approvals |

### By Task Type

| Task | Available Recipes |
|------|-------------------|
| **Discover what exists** | Discover Tenant Structure |
| **Analyze data** | Talk to Your Data, Find Trends, Compare Operations, Deep Dashboard Analysis |
| **Compare data** | Dashboard Comparison, Compare Operations |
| **Quality check** | Check Data Completeness, Spot Anomalies |
| **Track targets** | Target Gap Analysis |
| **Verify data flows** | Verify Data |
| **Generate reports** | Generate Narrative, Talk to Your Dashboard |
| **Create configuration** | Create Metric Wizard, Create Calculation, Create Widget Template, Build Dashboard, Set Up Data Source |
| **Bulk edit via Excel** | Export to Excel |
| **Enter/import data** | Enter Data, Bulk Import Data |
| **Seed a tenant** | Seed Tenant |
| **Manage approvals** | Review Approvals |

---

## All Recipes Available

All 22 recipes are now fully functional. The `reporting` commands shipped 2026-01-28. Dashboard analysis recipes added 2026-01-30. Seed Tenant recipe added 2026-02-07. Configuration recipes (calculations, dashboards, Excel, data sources) added 2026-02-07.

---

## Recipe Directory Structure

```
recipes/
├── README.md              # This file
├── exploration/
│   ├── discover-tenant-structure.md
│   ├── talk-to-your-data.md
│   ├── talk-to-your-dashboard.md
│   └── check-completeness.md
├── analysis/
│   ├── compare-operations.md
│   ├── dashboard-comparison.md
│   ├── deep-dashboard-analysis.md
│   ├── find-trends.md
│   ├── spot-anomalies.md
│   └── target-gap-analysis.md
├── reporting/
│   ├── verify-data.md
│   └── generate-narrative.md
├── configuration/
│   ├── build-dashboard.md
│   ├── create-calculation.md
│   ├── create-metric.md
│   ├── create-widget-template.md
│   ├── export-to-excel.md
│   └── set-up-data-source.md
└── data-management/
    ├── seed-tenant.md
    ├── enter-data.md
    ├── bulk-import.md
    └── review-approvals.md
```

---

## See Also

- [SKILL.md](../SKILL.md) — Vocabulary translation and CLI overview
- [Commands Reference](../reference/commands.md) — Quick command lookup
- [Template Selection Guide](../reference/template-selection.md) — Choose capture/report/widget/dashboard workflow first
- [Glossary](../reference/glossary.md) — Term definitions
