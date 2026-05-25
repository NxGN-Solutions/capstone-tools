# Capstone Tooling Downloads

This repository distributes the Capstone CLI and Capstone MCP server as prebuilt binaries. It intentionally does not contain application source code.

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

Last updated: `2026-05-25T10:54:45Z`
