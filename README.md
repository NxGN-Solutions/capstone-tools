# Capstone Tooling Downloads

This repository distributes the Capstone CLI and Capstone MCP server as prebuilt binaries. It intentionally does not contain application source code.

The latest CLI skills, public documentation, release manifests, and checksums
live in this repository:
`https://github.com/NxGN-Solutions/capstone-tools`. Point CLI users and agents
there for current operating guidance instead of private source-repository docs.

## What To Download

Use the latest GitHub Release for the environment you need to access, then choose the asset for your operating system.

| Tool | Asset | Use when |
|------|-------|----------|
| CLI | `capstone-cli-linux-x64.zip` | Running `cap` on Linux |
| CLI | `capstone-cli-osx-arm64.zip` | Running `cap` on Apple Silicon macOS |
| CLI | `capstone-cli-win-x64.zip` | Running `cap.exe` on Windows |
| MCP | `capstone-mcp-linux-x64.zip` | Connecting Claude Desktop to Capstone on Linux |
| MCP | `capstone-mcp-osx-arm64.zip` | Connecting Claude Desktop to Capstone on Apple Silicon macOS |
| MCP | `capstone-mcp-win-x64.zip` | Connecting Claude Desktop to Capstone on Windows |

Each zip has a matching `.sha256` checksum. The environment manifest lists the current release version, channel, tools, runtime IDs, file names, and checksums.

Environment-specific deployments update `environments/<slug>/manifest.json`, so demo and production can stay on their own latest versions. Before the first release, see `manifest.example.json` for the expected structure.

Manual demo and production deployments publish environment-scoped releases:

| Environment | Release tag pattern | Manifest |
|-------------|---------------------|----------|
| Demo | `demo-v<version>` | `environments/demo/manifest.json` |
| Production | `prod-v<version>` | `environments/prod/manifest.json` |

Environment-scoped packages include a `config/` folder with `configure-cli.sh`, `configure-cli.ps1`, shell environment files, a CLI `config.json`, and a Claude Desktop MCP config template already pointed at that deployment's API URL.

## Version

Current published version: `see environment manifests`

Channel: `environment-specific`

Release: see GitHub Releases

Environment: `see environment release assets`

API URL: `see environment release assets`

Manifest: `see environments/<slug>/manifest.json`

If that release URL returns 404, no matching release has been published yet. Use the repository docs and wait for the first release assets.

The CLI and MCP server are released together because they share the same Capstone API contracts. If the Capstone API/contracts version is bumped, install the matching tooling release.

## CLI Quick Start

Extract the CLI package, then run:

```bash
./cap config set api-url <capstone-api-url>
./cap auth login
./cap auth whoami
./cap schema --json
```

Windows PowerShell:

```powershell
.\cap.exe config set api-url <capstone-api-url>
.\cap.exe auth login
.\cap.exe auth whoami
.\cap.exe schema --json
```

For an environment-scoped release, run the included helper instead:

```bash
./config/configure-cli.sh
```

Windows PowerShell:

```powershell
.\config\configure-cli.ps1
```

See [CLI documentation](docs/cli/) and [CLI agent instructions](skills/capstone-cli/SKILL.md).

For machine-readable workflows, prefer `--json`. Set
`CAPSTONE_OUTPUT_VERSION=2` to opt into the shared JSON envelope
`{ ok, data, error, warnings, meta }` on commands that use the shared
formatter. Run `cap schema --json` to discover supported command arguments,
output models, the exit-code contract, and the `upsertIdentity` import
contract. For automation, treat exit code `0` as success, `1` as local CLI
failure or cancellation, `2` as validation/client-actionable failure, `3` as
auth/tenant failure, `4` as server/API/network/rate-limit/malformed-response
failure, `5` as timeout, and `6` as partial result. Do not grep stderr text for
control flow. For imports and upserts, flat entities match by exact full name only
after trimming leading/trailing whitespace, and tree entities match by exact
full path only after trimming leading/trailing whitespace. Matching is
case-sensitive. Excel upload and JSON batch/upsert use the same identity
semantics. `upsertIdentity.surfaces` describes when to use JSON
`import-json --upsert` or Excel `upload-excel`. For rerunnable tenant builds,
prefer `import-json --upsert --json` for masterdata, model definitions, and
widget/dashboard templates before using one-item create/save loops. Use
`cap system tenants snapshot --output snapshot --json` when you need a
reviewable export of those same import-ready JSON surfaces plus a manifest.
Restore is a reviewed workflow, not an automatic command: run
`cap workflows show restore-from-snapshot --json`, inspect the dry-run plans,
validate calculations, then apply the explicit `import-json --upsert` commands.
For post-build analysis checks, pair `cap data recalculation wait <version> --json`
with `cap reporting computed-values audit --metrics <ids> --strict --json`.
Before live dashboard checks, run `cap templates dashboard-templates audit
<dashboard-template-id> --strict --json` to catch missing widget references or
structural template issues.

## Breaking Changes

### Tenant delete force semantics

`cap system tenants delete --force` now uses the async force-delete operation
contract. The CLI calls the preview endpoint, prints row counts and blockers,
starts an operation with the preview confirmation phrase, then watches operation
status until terminal state or `--timeout-seconds`.

`--force` no longer sends `DELETE System/Tenants` with `force: true`. Direct API
callers must use `POST System/Tenants/Delete/Preview` and
`POST System/Tenants/Delete/Operations`; the legacy force flag is rejected.

If the watch times out, the server-side operation continues. Resume with:

```bash
cap system tenants delete-status <operation-id> --json
cap system tenants delete-watch <operation-id> --timeout-seconds 600
```

## MCP Quick Start

Extract the MCP package and configure Claude Desktop to run the absolute path to `capstone-mcp` or `capstone-mcp.exe`.

macOS/Linux:

```json
{
  "mcpServers": {
    "capstone": {
      "command": "/absolute/path/to/capstone-mcp",
      "env": {
        "CAPSTONE_API_URL": "<capstone-api-url>"
      }
    }
  }
}
```

Windows:

```json
{
  "mcpServers": {
    "capstone": {
      "command": "C:\\absolute\\path\\to\\capstone-mcp.exe",
      "env": {
        "CAPSTONE_API_URL": "<capstone-api-url>"
      }
    }
  }
}
```

Restart Claude Desktop, then ask Claude to log in to Capstone.

For an environment-scoped release, start from `config/claude-desktop.capstone-mcp.json`; it already contains the correct `CAPSTONE_API_URL`.

See [MCP documentation](docs/mcp/) and [MCP agent instructions](skills/capstone-mcp/SKILL.md).

## Agent Files

This repository includes agent-facing context:

- [AGENTS.md](AGENTS.md) for Codex and other coding agents.
- [CLAUDE.md](CLAUDE.md) for Claude Code and Claude Desktop users.
- [llms.txt](llms.txt) as a compact index for LLM ingestion.
- [skills/capstone-cli/SKILL.md](skills/capstone-cli/SKILL.md) for CLI workflows.
- [skills/capstone-mcp/SKILL.md](skills/capstone-mcp/SKILL.md) for MCP workflows.

## Security Model

The binaries contain no customer data and no embedded credentials. Access to Capstone data is controlled by the Capstone API through user authentication, tenant selection, and server-side authorization.

## License And Support

These downloads and documents are distributed under the [Capstone Tools Distribution License](LICENSE.md). This is a binary and documentation distribution repo, not an open-source source-code repository.

For help, use your Capstone implementation, support, or account contact. Security reporting guidance is in [SECURITY.md](SECURITY.md), and support request guidance is in [SUPPORT.md](SUPPORT.md).

Last updated: `2026-06-15T11:11:39Z`
