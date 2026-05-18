# Capstone Tooling Downloads

This repository distributes the Capstone CLI and Capstone MCP server as prebuilt binaries. It intentionally does not contain application source code.

## What To Download

Use the latest GitHub Release for your operating system.

| Tool | Asset | Use when |
|------|-------|----------|
| CLI | `capstone-cli-linux-x64.zip` | Running `cap` on Linux |
| CLI | `capstone-cli-osx-arm64.zip` | Running `cap` on Apple Silicon macOS |
| CLI | `capstone-cli-win-x64.zip` | Running `cap.exe` on Windows |
| MCP | `capstone-mcp-linux-x64.zip` | Connecting Claude Desktop to Capstone on Linux |
| MCP | `capstone-mcp-osx-arm64.zip` | Connecting Claude Desktop to Capstone on Apple Silicon macOS |
| MCP | `capstone-mcp-win-x64.zip` | Connecting Claude Desktop to Capstone on Windows |

Each zip has a matching `.sha256` checksum. `manifest.json` lists the current release version, channel, tools, runtime IDs, file names, and checksums.

`manifest.json` is updated by the release workflow. Before the first release, see `manifest.example.json` for the expected structure.

## Version

Current published version: `0.4.63`

Channel: `stable`

Release: https://github.com/NxGN-Solutions/capstone-tools/releases/tag/0.4.63

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

For help, use your NxGN implementation, support, or account contact. Security reporting guidance is in [SECURITY.md](SECURITY.md), and support request guidance is in [SUPPORT.md](SUPPORT.md).

Last updated: `2026-05-18T08:51:34Z`
