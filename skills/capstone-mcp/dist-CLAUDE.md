# Capstone MCP Server

The Capstone MCP server (`capstone-mcp`) connects Claude Desktop to your Capstone data management platform. This distribution contains a pre-built executable — no .NET SDK required.

## First-Run Setup

### 1. Configure Claude Desktop

Open the Claude Desktop MCP configuration file:

- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

Add the Capstone server (adjust the path to where you extracted the binary):

**macOS:**
```json
{
  "mcpServers": {
    "capstone": {
      "command": "/path/to/capstone-mcp",
      "env": {
        "CAPSTONE_API_URL": "https://your-capstone-instance.example.com"
      }
    }
  }
}
```

**Windows:**
```json
{
  "mcpServers": {
    "capstone": {
      "command": "C:\\path\\to\\capstone-mcp.exe",
      "env": {
        "CAPSTONE_API_URL": "https://your-capstone-instance.example.com"
      }
    }
  }
}
```

Replace `https://your-capstone-instance.example.com` with your actual Capstone API URL (provided by your administrator).

Restart Claude Desktop after saving.

### 2. Authenticate

In a Claude Desktop conversation, say:

> "Log in to Capstone"

Claude calls the `Login` tool, which opens your browser for Microsoft or Google SSO. After completing sign-in, Claude confirms authentication.

Credentials are stored locally at `~/.capstone-mcp/credentials.json` with restricted file permissions.

### 3. Verify Connection

Ask Claude:

> "What's available in this tenant?"

Claude uses the `/discover-structure` prompt to explore your tenant. If you see your data, everything is working.

### 4. Switch Tenant (if needed)

> "List my available tenants"
> "Switch to [tenant name]"

---

## What This Server Provides

| Mechanism | Count | Description |
|-----------|-------|-------------|
| **Tools** | 41 | Data queries, model inspection, template CRUD, widget rendering — called by Claude as needed |
| **Resources** | 7 | Auto-loaded context (metrics, org structure, availability, tool guide) — Claude already knows your model |
| **Prompts** | 8 | Guided analysis workflows invoked via slash commands |

### Tool Categories

| Category | Tools | Purpose |
|----------|-------|---------|
| Auth (6) | `Login`, `Logout`, `Whoami`, `ListTenants`, `SwitchTenant`, `Languages` | Authentication and tenant management |
| Model (5) | `model_metrics_list`, `model_metrics_get`, `model_orgNodes_list`, `model_frameworks_list`, `model_disciplines_list` | Inspect model structure |
| Data (5) | `data_timePeriods_list`, `data_availability`, `data_inputValues_list`, `data_inputValues_save`, `data_computedValues_list` | Query and save data |
| Reporting (2) | `reporting_dashboards_getData`, `reporting_widgets_getData` | Dashboard/widget data as CSV |
| Templates (6) | `templates_dashboards_list`, `templates_widgets_list`, `templates_spreadsheetReports_list`, `templates_spreadsheetCaptures_list`, `templates_spreadsheetReports_get`, `templates_spreadsheetCaptures_get` | Discover templates |
| Template CRUD (10) | `templates_widgets_create/save/delete`, `templates_reports_create/save/delete`, `templates_dashboards_get/create/save/delete` | Create, update, and delete templates |
| Apps (7) | `apps_dashboard_render`, `apps_widget_infoCard`, `apps_widget_pieChart`, `apps_widget_xyChart`, `apps_widget_table`, `apps_widget_aiSummary`, `apps_chart_render` | Visual widgets in conversation |
| Status (1) | `GetStatus` | Server health check |

### Resources (Auto-Loaded Context)

| Resource URI | Content |
|-------------|---------|
| `capstone://user/context` | Current user and tenant |
| `capstone://model/metrics` | Metric hierarchy with types, units, formulas |
| `capstone://model/organization` | Org node tree (sites, regions) |
| `capstone://model/disciplines` | Metric categories |
| `capstone://model/frameworks` | Reporting frameworks |
| `capstone://guide/tool-selection` | Tool selection guide for LLMs |
| `capstone://data/availability` | Which periods have data |

### Prompts (Slash Commands)

| Prompt | Description |
|--------|-------------|
| `/discover-structure` | Explore tenant configuration |
| `/talk-to-dashboard` | Analyze a specific dashboard |
| `/target-gap-analysis` | Compare actuals vs targets |
| `/spot-anomalies` | Find unusual values |
| `/compare-operations` | Compare locations or sites |
| `/find-trends` | Identify metric trends |
| `/check-completeness` | Audit data completeness |
| `/generate-narrative` | Write a data-driven summary |

---

## Recipes

Detailed step-by-step workflows are in the bundled `docs/` directory:

| Task | Recipe |
|------|--------|
| Explore what data exists | `docs/recipes/exploration/discover-structure.md` |
| Analyze a specific metric | `docs/recipes/exploration/talk-to-data.md` |
| Get dashboard insights | `docs/recipes/exploration/talk-to-dashboard.md` |
| Compare locations | `docs/recipes/analysis/compare-operations.md` |
| Find trends | `docs/recipes/analysis/find-trends.md` |
| Spot anomalies | `docs/recipes/analysis/spot-anomalies.md` |
| Track actuals vs targets | `docs/recipes/analysis/target-gap-analysis.md` |
| Check data completeness | `docs/recipes/exploration/check-completeness.md` |
| Generate written summary | `docs/recipes/reporting/generate-narrative.md` |

---

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `CAPSTONE_API_URL` | Yes | — | Capstone API base URL |
| `CAPSTONE_LOG_LEVEL` | No | `Information` | Log verbosity: `Trace`, `Debug`, `Information`, `Warning`, `Error`, `Critical` |

---

## Reference Documentation

| Document | Contents |
|----------|----------|
| `docs/SKILL.md` | Concept glossary, tool categories, query patterns |
| `docs/reference/tools.md` | All 41 tools with parameters and response formats |
| `docs/reference/resources.md` | Auto-loaded resource details |
| `docs/reference/prompts.md` | Prompt parameters and usage |
| `docs/reference/glossary.md` | Term definitions |
| `docs/reference/data-model.md` | Enum values, aggregation methods |

---

## Troubleshooting

### "CAPSTONE_API_URL environment variable is required"

The `CAPSTONE_API_URL` is missing from your Claude Desktop config. Add it to the `env` block in `claude_desktop_config.json`.

### Login opens browser but authentication fails

- Ensure your Capstone account has API access enabled
- Check that the API URL is correct (no trailing slash needed)
- Try specifying the provider: ask Claude "Log in to Capstone with Google"

### "Not authenticated" errors on tool calls

Your session may have expired. Say "Log in to Capstone" to re-authenticate. The server automatically refreshes tokens when possible, but refresh tokens do expire.

### Server not appearing in Claude Desktop

1. Verify the binary path in your config is correct and absolute
2. Check that the binary is executable (`chmod +x capstone-mcp` on macOS/Linux)
3. Restart Claude Desktop after config changes
4. Check Claude Desktop logs for MCP server startup errors

### Binary won't run on Intel Mac

The default build targets `osx-arm64` (Apple Silicon). For Intel Macs, request an `osx-x64` build.
