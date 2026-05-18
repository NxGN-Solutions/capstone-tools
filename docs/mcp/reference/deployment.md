# MCP Server Deployment Guide

> Build and package the Capstone MCP server for distribution to colleagues.

## Overview

The MCP server is published as a **Native AOT single-file binary** — no .NET SDK required on the target machine. Each distribution package includes:

```
capstone-mcp-{platform}/
├── capstone-mcp (or capstone-mcp.exe)  ← Native binary
├── CLAUDE.md                            ← Standalone setup guide
└── docs/                                ← Recipes and reference documentation
    ├── SKILL.md
    ├── recipes/
    └── reference/
```

The `CLAUDE.md` in the dist folder is a **standalone guide** for recipients who don't have the source repo. It covers Claude Desktop configuration, authentication, available tools/resources/prompts, and troubleshooting.

GitHub Releases in the binary-only tooling distribution repo are the source of truth for MCP distribution. The recommended repo is `NxGN-Solutions/capstone-tools`; it can be public for stable client downloads while the source repo remains private. The same release also contains the matching CLI packages, `manifest.json`, checksums, agent guides, and skills.

---

## Supported Platforms

| Runtime ID | Platform | Binary | Notes |
|-----------|----------|--------|-------|
| `osx-arm64` | macOS (Apple Silicon) | `capstone-mcp` | Native AOT, Mach-O arm64 |
| `linux-x64` | Linux (x64) | `capstone-mcp` | Build on Linux only |
| `win-x64` | Windows (x64) | `capstone-mcp.exe` | Self-contained |

> **Cross-compilation limitation:** Native AOT cannot cross-compile across operating systems. Build `win-x64` on Windows, `osx-arm64` on macOS, and `linux-x64` on Linux.

---

## Quick Start

### Using the publish script (recommended)

```bash
# Build for current platform (auto-detected)
./scripts/publish-mcp.sh

# Build for a specific platform
./scripts/publish-mcp.sh osx-arm64
./scripts/publish-mcp.sh win-x64

# Build for all platforms (only works on matching OS)
./scripts/publish-mcp.sh all
```

Output: `dist/capstone-mcp-{rid}/` folder + `dist/capstone-mcp-{rid}.zip`

`all` builds every RID supported by the current host OS. Native AOT release builds for every OS are handled by the GitHub Actions matrix.

The script also writes:

```text
dist/capstone-mcp-{rid}.zip.sha256
dist/manifest.json
```

### Manual build

```bash
# 1. Publish the AOT binary
dotnet publish NxGN.Capstone.Mcp -r osx-arm64 -c Release -o artifacts/mcp/osx-arm64

# 2. Assemble the dist folder
mkdir -p dist/capstone-mcp-osx-arm64/docs
cp artifacts/mcp/osx-arm64/capstone-mcp dist/capstone-mcp-osx-arm64/
cp docs/mcp/dist-CLAUDE.md dist/capstone-mcp-osx-arm64/CLAUDE.md
cp -R docs/mcp/* dist/capstone-mcp-osx-arm64/docs/

# 3. Zip for distribution
cd dist && zip -r capstone-mcp-osx-arm64.zip capstone-mcp-osx-arm64/
```

---

## Directory Structure

```
repo/
├── artifacts/mcp/           ← Raw publish output (binary + PDBs)
│   ├── osx-arm64/
│   └── win-x64/
├── dist/                    ← Distribution packages (binary + docs + zip)
│   ├── capstone-mcp-osx-arm64/
│   ├── capstone-mcp-osx-arm64.zip
│   ├── capstone-mcp-osx-arm64.zip.sha256
│   ├── manifest.json
│   ├── capstone-mcp-win-x64/
│   └── capstone-mcp-win-x64.zip
└── scripts/
    └── publish-mcp.sh       ← Automated build + package script
```

| Directory | Purpose | Git-tracked |
|-----------|---------|-------------|
| `artifacts/` | Build output (binary + debug symbols) | No (`.gitignore`) |
| `dist/` | Distribution packages ready to share | No (`.gitignore`) |
| `scripts/` | Build and deployment scripts | Yes |
| `docs/mcp/dist-CLAUDE.md` | Source for the standalone setup guide | Yes |

---

## What Ships in Each Package

### Binary (`capstone-mcp` / `capstone-mcp.exe`)

Native AOT compiled — runs without .NET installed. Includes all dependencies statically linked. Communicates with Claude Desktop over stdio using the MCP protocol.

### CLAUDE.md

A standalone guide for recipients configuring Claude Desktop. Contains:
- Claude Desktop configuration template (with platform-specific paths)
- Authentication flow (OAuth via browser)
- Available tools, resources, and prompts summary
- Recipe index
- Environment variables
- Troubleshooting

**Source location:** `docs/mcp/dist-CLAUDE.md` — edit this file when updating the distribution guide. The publish script copies it into each platform folder.

### docs/

Copied from `docs/mcp/`. Contains:
- `SKILL.md` — concept glossary, tool categories, query patterns
- `recipes/` — 11 step-by-step analysis and exploration workflows
- `reference/` — tools, resources, prompts, glossary, data model

---

## When to Rebuild

Rebuild the distribution when:
- MCP server code changes (new tools, bug fixes)
- `Api.Contracts` changes (shared DTOs between MCP and API)
- `Directory.Build.props` `<Version>` is bumped
- Documentation updates (recipes, reference docs)
- `docs/mcp/dist-CLAUDE.md` is updated

> **Important:** The MCP server and CLI share `Api.Contracts`. When contracts change, rebuild both. See [CLI Deployment Guide](../../cli/reference/deployment.md).

---

## Also Rebuild the CLI

The CLI (`NxGN.Capstone.Cli`) also uses Native AOT and shares `Api.Contracts`. Rebuild it alongside the MCP server:

```bash
./scripts/publish-cli.sh osx-arm64
```

See the [CLI Deployment Guide](../../cli/reference/deployment.md) for details.

---

## Distributing to Users

1. Publish the combined tooling release to `NxGN-Solutions/capstone-tools`
2. Share the release URL and the correct zip (`capstone-mcp-{rid}.zip`)
3. Recipient extracts the zip and places the folder wherever convenient
4. Recipient configures Claude Desktop:
   - Add the `capstone` server to `claude_desktop_config.json` (see template in the included `CLAUDE.md`)
   - Set `CAPSTONE_API_URL` to their Capstone instance URL
   - Restart Claude Desktop
5. Recipient authenticates: say "Log in to Capstone" in a conversation

For Claude Code users, the included `CLAUDE.md` provides all the context needed. They can add the extracted folder as a project in Claude Code, and the `CLAUDE.md` will be automatically loaded.

The workflow at `.github/workflows/cli-release.yml` is the normal release path. It builds CLI and MCP packages on OS-matched runners, updates the distribution repo documentation/skills, and creates the release assets there. Configure `CAPSTONE_TOOLING_RELEASES_TOKEN` as a source-repo secret with write access to the distribution repo before running it.

---

## Troubleshooting

### AOT publish fails with "cannot find clang"

Install Xcode command line tools: `xcode-select --install`

### Binary is unexpectedly large

Windows self-contained builds include the .NET runtime and are larger than macOS AOT builds. This is normal.

### "Not authenticated" after sharing

Each machine needs its own OAuth login — credentials are stored in `~/.capstone-mcp/credentials.json` and are not portable.

### Binary won't run on Intel Mac

The default build targets `osx-arm64` (Apple Silicon). For Intel Macs, build with `-r osx-x64`.

### Claude Desktop doesn't see the server

Ensure the `command` path in `claude_desktop_config.json` is an **absolute path** to the binary. Relative paths may not resolve correctly. Restart Claude Desktop after any config change.

---

## See Also

- [MCP Setup Guide](../setup.md) — User-facing setup instructions
- [CLI Deployment Guide](../../cli/reference/deployment.md) — CLI build and distribution
- [MCP CLAUDE.md](../../../NxGN.Capstone.Mcp/CLAUDE.md) — Developer-facing MCP docs
