# Capstone Tooling For Claude

This repository is the binary distribution and documentation hub for Capstone's Claude-facing tools:

- `cap` - Capstone CLI for Claude Code, terminals, scripts, and deterministic JSON workflows.
- `capstone-mcp` - MCP server for Claude Desktop.

The Capstone source repository is private. This repository is intended for clients, implementers, and agents who need the tools and operating guidance, not source code.

Use `https://github.com/NxGN-Solutions/capstone-tools` as the canonical public
location for the latest CLI skills, MCP skills, and documentation.

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
- Set `CAPSTONE_OUTPUT_VERSION=2` only when the workflow expects the shared
  `{ ok, data, error, warnings, meta }` JSON envelope; otherwise leave the
  default per-command JSON shapes in place.
- Use `cap schema --json` for the current command, output, and exit-code
  contract before writing automation.
- Interpret exit codes consistently: `0` success, `1` local CLI failure or
  cancellation, `2` validation/client-actionable failure, `3` auth/tenant
  failure, `4` server/API/network/rate-limit/malformed-response failure, `5`
  timeout, and `6` partial result. Do not branch on free-text stderr.
- For imports and upserts, use the `upsertIdentity` schema contract: flat
  entities match by exact full name only after trimming leading/trailing
  whitespace, and tree entities match by exact full path only after trimming
  leading/trailing whitespace. Matching is case-sensitive. Excel upload and
  JSON batch/upsert use the same identity semantics. Use
  `upsertIdentity.surfaces` to choose JSON `import-json --upsert` or Excel
  `upload-excel` based on the diagnostics the workflow needs.
- For rerunnable tenant builds, prefer `import-json --upsert --json` for
  masterdata, model definitions, and widget/dashboard templates before using
  one-item create/save loops.
- Use `cap system tenants snapshot --output snapshot --json` when you need a
  reviewable export of those same import-ready JSON surfaces plus a manifest.
  Restore is the explicit reviewed `import-json --upsert` path; use
  `cap workflows show restore-from-snapshot --json` for the dry-run, validate,
  apply, and audit sequence.
- After model or seed changes, run `cap data recalculation wait <version>
  --json`, then use `cap reporting computed-values audit --metrics <ids>
  --strict --json` for automation-safe output verification.
- Before live dashboard data checks, run `cap templates dashboard-templates
  audit <dashboard-template-id> --strict --json` to catch missing widget
  references or structural template issues.
- Prefer MCP prompts/resources for Claude Desktop discovery.

Current release: `see environment manifests` (`environment-specific`)

Last updated: `2026-06-09T11:18:12Z`
