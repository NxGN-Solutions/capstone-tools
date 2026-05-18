# Capstone MCP Server: Setup Guide

> Configure Claude Desktop to connect to your Capstone tenant.

## Prerequisites

- [Claude Desktop](https://claude.ai/download) installed
- A Capstone account with API access
- Your Capstone API URL (provided by your administrator)

---

## 1. Install the MCP Server

The Capstone MCP server runs as a local process managed by Claude Desktop.

### Option A: Pre-built binary (recommended)

If you received a distribution zip (`capstone-mcp-osx-arm64.zip` or `capstone-mcp-win-x64.zip`):

1. Extract the zip to a convenient location (e.g., `~/tools/capstone-mcp/`)
2. Make the binary executable (macOS/Linux): `chmod +x capstone-mcp`
3. Skip to [Configure Claude Desktop](#2-configure-claude-desktop) — use the full path to the extracted binary

> See the [MCP Deployment Guide](reference/deployment.md) for building from source.

### Option B: Install via dotnet tool

```bash
# Install via dotnet tool
dotnet tool install --global capstone-mcp

# Verify installation
capstone-mcp --version
```

---

## 2. Configure Claude Desktop

Open Claude Desktop settings and edit the MCP server configuration:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

Add the Capstone server:

```json
{
  "mcpServers": {
    "capstone": {
      "command": "capstone-mcp",
      "env": {
        "CAPSTONE_API_URL": "https://your-capstone-instance.example.com"
      }
    }
  }
}
```

Replace `https://your-capstone-instance.example.com` with your actual Capstone API URL.

Restart Claude Desktop after saving.

---

## 3. Authenticate

In a Claude Desktop conversation, type:

> "Log in to Capstone"

Claude will call the `Login` tool, which opens your browser for Microsoft or Google SSO authentication. After completing the sign-in flow in your browser, Claude confirms authentication.

Credentials are stored locally at `~/.capstone-mcp/credentials.json` with restricted file permissions (owner-only on macOS/Linux).

---

## 4. Verify Connection

Ask Claude:

> "What's available in this tenant?"

Claude uses the `/discover-structure` prompt to explore your tenant's organization, metrics, disciplines, and frameworks. If you see your data, everything is working.

---

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `CAPSTONE_API_URL` | Yes | — | Capstone API base URL |
| `CAPSTONE_LOG_LEVEL` | No | `Information` | Log verbosity: `Trace`, `Debug`, `Information`, `Warning`, `Error`, `Critical` |

---

## Troubleshooting

### "CAPSTONE_API_URL environment variable is required"

The `CAPSTONE_API_URL` is missing from your Claude Desktop config. Add it to the `env` block in `claude_desktop_config.json`.

### Login opens browser but authentication fails

- Ensure your Capstone account has API access enabled
- Check that the API URL is correct (no trailing slash needed — the server trims it)
- Try specifying the OAuth provider: ask Claude "Log in to Capstone with Google"

### "Not authenticated" errors on tool calls

Your session may have expired. Ask Claude "Log in to Capstone" to re-authenticate. The MCP server automatically refreshes tokens when possible, but refresh tokens do expire.

### Server not appearing in Claude Desktop

1. Check the config file path is correct for your OS
2. Ensure `capstone-mcp` is on your PATH (run `which capstone-mcp` or `where capstone-mcp`)
3. Check Claude Desktop logs for MCP server startup errors

### Switching tenants

If your account has access to multiple tenants, ask Claude:

> "List my available tenants"
> "Switch to [tenant name]"

Claude uses the `ListTenants` and `SwitchTenant` tools to change your active tenant.

---

## What's Next

Once connected, try these prompts in Claude Desktop:

- `/discover-structure` — Explore your tenant's configuration
- `/talk-to-dashboard dashboardName="..."` — Analyze a specific dashboard
- `/target-gap-analysis` — Check metric performance against targets
- `/check-completeness period="Q4 2024"` — Audit data completeness

See the [MCP SKILL.md](SKILL.md) for the full list of available tools, resources, and prompts.
