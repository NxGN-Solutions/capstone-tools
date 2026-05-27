# Capstone Tooling For Claude

This repository is the binary distribution and documentation hub for Capstone's Claude-facing tools:

- `cap` - Capstone CLI for Claude Code, terminals, scripts, and deterministic JSON workflows.
- `capstone-mcp` - MCP server for Claude Desktop.

The Capstone source repository is private. This repository is intended for clients, implementers, and agents who need the tools and operating guidance, not source code.

## Recommended Claude Setup

Use Claude Desktop with the MCP server when the user wants conversational exploration over their Capstone tenant. Use Claude Code with the CLI when the work involves files, repeatable commands, bulk JSON payloads, or implementation handoff.

For best results, provide Claude with:

- This root `CLAUDE.md`.
- `AGENTS.md`.
- `skills/capstone-cli/SKILL.md` when using the CLI.
- `skills/capstone-mcp/SKILL.md` when using MCP.
- Relevant recipes under `docs/cli/recipes/` or `docs/mcp/recipes/`.

## CLI Bootstrap

1. Download the right `capstone-cli-{rid}.zip` from the target environment release when available, such as `demo-v<version>` or `prod-v<version>`.
2. Extract it to a stable folder.
3. Configure the API URL with the included helper:

```bash
./config/configure-cli.sh
```

Windows PowerShell:

```powershell
.\config\configure-cli.ps1
```

If the package does not contain `config/`, configure the API URL manually:

```bash
./cap config set api-url <capstone-api-url>
```

4. Authenticate:

```bash
./cap auth login
```

5. Discover the available command surface:

```bash
./cap schema --json
./cap concepts --json
./cap workflows list --json
```

Windows users run `.\cap.exe` instead of `./cap`.

## MCP Bootstrap

1. Download the right `capstone-mcp-{rid}.zip` from the target environment release when available.
2. Extract it to a stable folder.
3. Start from `config/claude-desktop.capstone-mcp.json` and replace only the binary path. If the package does not contain `config/`, add the binary path and `CAPSTONE_API_URL` manually.
4. Restart Claude Desktop.
5. Ask Claude to log in to Capstone.

See `docs/mcp/setup.md` for complete configuration examples.

## Operating Notes

- Always ask which Capstone API URL to use unless it is already configured or the package contains environment-specific `config/` files.
- Authentication is per user and per machine.
- Tenant access is controlled by the Capstone API, not by the binary download.
- Prefer JSON output from the CLI for agent reasoning.
- Prefer MCP prompts/resources for Claude Desktop discovery.

Current release: `see environment manifests` (`environment-specific`)

Last updated: `2026-05-27T06:50:40Z`
