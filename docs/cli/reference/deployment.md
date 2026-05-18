# CLI Deployment Guide

> Build and package the Capstone CLI for distribution to colleagues.

## Target Distribution Model

GitHub Releases in the binary-only tooling distribution repo are the source of truth for CLI distribution. The recommended repo is `NxGN-Solutions/capstone-tools`; it can be public for stable client downloads while the source repo remains private. Each release should contain:

- One zip per runtime ID: `capstone-cli-{rid}.zip`
- One checksum per zip: `capstone-cli-{rid}.zip.sha256`
- The matching MCP packages for the same version
- A compact `manifest.json` used by `cap update check --json`
- Root `README.md`, `AGENTS.md`, `CLAUDE.md`, `llms.txt`, `docs/`, and `skills/` on the distribution repo default branch

The release version comes from the shared .NET `Directory.Build.props` `<Version>` property. Release tags must match that version (with or without a leading `v`) so the CLI binary, API contracts, release assets, and update manifest stay aligned.

Package managers are preferred for managed installs because they own upgrade and uninstall behavior. Manual zip installs remain supported, but updates are explicit: users run `cap update check --json` to discover a newer release and then install it deliberately. The CLI must not silently self-update.

`cap update install` is intentionally not implemented yet. When added, it should verify SHA-256 checksums, support `--version`, and refuse package-manager installs.

---

## Overview

The CLI is published as a **Native AOT single-file binary** — no .NET SDK required on the target machine. Each distribution package includes:

```
capstone-cli-{platform}/
├── cap (or cap.exe)     ← Native binary
├── CLAUDE.md            ← Standalone usage guide for Claude Code users
└── docs/                ← Recipes and reference documentation
    ├── SKILL.md
    ├── recipes/
    └── reference/
```

The `CLAUDE.md` in the dist folder is a **standalone guide** designed for recipients who don't have the source repo. It covers first-run setup, workspace caching, shell syntax, and recipe navigation.

---

## Supported Platforms

| Runtime ID | Platform | Binary | Notes |
|-----------|----------|--------|-------|
| `osx-arm64` | macOS (Apple Silicon) | `cap` | ~11 MB, Mach-O arm64 |
| `linux-x64` | Linux (x64) | `cap` | Build on Linux only |
| `win-x64` | Windows (x64) | `cap.exe` | ~73 MB, self-contained |

> **Cross-compilation limitation:** Native AOT cannot cross-compile across operating systems. Build `win-x64` on Windows, `osx-arm64` on macOS, and `linux-x64` on Linux.

---

## Quick Start

### Using the publish script (recommended)

```bash
# Build for current platform (auto-detected)
./scripts/publish-cli.sh

# Build for a specific platform
./scripts/publish-cli.sh osx-arm64
./scripts/publish-cli.sh win-x64

# Build for all platforms (only works on matching OS)
./scripts/publish-cli.sh all
```

Output: `dist/capstone-cli-{rid}/` folder + `dist/capstone-cli-{rid}.zip`

`all` builds every RID supported by the current host OS. Native AOT release builds for every OS are handled by the GitHub Actions matrix.

The script also writes:

```text
dist/capstone-cli-{rid}.zip.sha256
dist/manifest.json
```

### Manual build

```bash
# 1. Publish the AOT binary
dotnet publish NxGN.Capstone.Cli -r osx-arm64 -c Release -o artifacts/cli/osx-arm64

# 2. Assemble the dist folder
mkdir -p dist/capstone-cli-osx-arm64/docs
cp artifacts/cli/osx-arm64/cap dist/capstone-cli-osx-arm64/
cp docs/cli/dist-CLAUDE.md dist/capstone-cli-osx-arm64/CLAUDE.md
cp -R docs/cli/* dist/capstone-cli-osx-arm64/docs/

# 3. Zip for distribution
cd dist && zip -r capstone-cli-osx-arm64.zip capstone-cli-osx-arm64/
```

---

## Directory Structure

```
repo/
├── artifacts/cli/           ← Raw publish output (binary + PDBs)
│   ├── osx-arm64/
│   └── win-x64/
├── dist/                    ← Distribution packages (binary + docs + zip)
│   ├── capstone-cli-osx-arm64/
│   │   └── CLAUDE.md        ← Copied from docs/cli/dist-CLAUDE.md
│   ├── capstone-cli-osx-arm64.zip
│   ├── capstone-cli-osx-arm64.zip.sha256
│   ├── manifest.json
│   ├── capstone-cli-win-x64/
│   └── capstone-cli-win-x64.zip
└── scripts/
    └── publish-cli.sh       ← Automated build + package script
```

| Directory | Purpose | Git-tracked |
|-----------|---------|-------------|
| `artifacts/` | Build output (binary + debug symbols) | No (`.gitignore`) |
| `dist/` | Distribution packages ready to share | No (`.gitignore`) |
| `scripts/` | Build and deployment scripts | Yes |
| `docs/cli/dist-CLAUDE.md` | Source for the standalone usage guide | Yes |

---

## What Ships in Each Package

### Binary (`cap` / `cap.exe`)

Native AOT compiled — runs without .NET installed. Includes all dependencies statically linked.

### CLAUDE.md

A standalone guide for recipients who use Claude Code. Contains:
- First-run setup (API URL, auth, tenant switching)
- Workspace caching pattern (tenant data cached locally for efficiency)
- Command structure and JSON output format
- Period format reference
- Recipe index
- Cross-platform shell syntax

**Source location:** `docs/cli/dist-CLAUDE.md` — edit this file when updating the distribution guide. The publish script copies it into each platform folder.

### docs/

Copied from `docs/cli/`. Contains:
- `SKILL.md` — vocabulary mapping and CLI overview
- `recipes/` — 22 step-by-step workflows
- `reference/` — commands, glossary, model-building, output formats, platform guide, template selection

---

## When to Rebuild

Rebuild the distribution when:
- CLI code changes (new commands, bug fixes)
- `Api.Contracts` changes (shared DTOs between CLI and API)
- `Directory.Build.props` `<Version>` is bumped
- Documentation updates (recipes, reference docs)
- `docs/cli/dist-CLAUDE.md` is updated

> **Important:** The CLI and MCP server share `Api.Contracts`. When contracts change, rebuild both. See [MCP Server Deployment](#also-rebuild-the-mcp-server) below.

---

## Also Rebuild the MCP Server

The MCP server (`NxGN.Capstone.Mcp`) also uses Native AOT and shares `Api.Contracts`. Rebuild it alongside the CLI:

```bash
dotnet publish NxGN.Capstone.Mcp -r osx-arm64 -c Release
```

The published binary path (referenced in Claude Desktop's `claude_desktop_config.json`):
```
NxGN.Capstone.Mcp/bin/Release/net10.0/osx-arm64/publish/capstone-mcp
```

Restart Claude Desktop after rebuilding to pick up the new binary.

---

## Distributing to Colleagues

1. Publish a GitHub Release containing the zip, checksum, and manifest assets
2. Prefer package-manager distribution for recurring users
3. For manual installs, share the release URL and the correct zip (`dist/capstone-cli-{rid}.zip`)
4. Recipient verifies the checksum, extracts the zip, and places the folder wherever convenient
5. Recipient runs `./cap auth login` (or `.\cap.exe auth login` on Windows) to authenticate

For Claude Code users, the included `CLAUDE.md` provides all the context needed. They can add the extracted folder as a project in Claude Code, and the `CLAUDE.md` will be automatically loaded.

---

## Release Manifest

`manifest.json` is compact and stable for CLI update checks:

```json
{
  "version": "0.4.63",
  "latestVersion": "0.4.63",
  "channel": "stable",
  "createdUtc": "2026-05-18T00:00:00Z",
  "releaseUrl": "https://github.com/NxGN-Solutions/capstone-tools/releases/tag/0.4.63",
  "tools": {
    "cli": {
      "artifacts": [
        {
          "rid": "osx-arm64",
          "fileName": "capstone-cli-osx-arm64.zip",
          "sha256": "<hex>"
        }
      ]
    },
    "mcp": {
      "artifacts": [
        {
          "rid": "osx-arm64",
          "fileName": "capstone-mcp-osx-arm64.zip",
          "sha256": "<hex>"
        }
      ]
    }
  },
  "artifacts": [
    {
      "tool": "cli",
      "rid": "osx-arm64",
      "fileName": "capstone-cli-osx-arm64.zip",
      "sha256": "<hex>"
    }
  ]
}
```

`cap update check --json` uses `CAPSTONE_CLI_RELEASE_MANIFEST_URL` or `--manifest-url` when provided. Without a manifest URL, it checks GitHub Releases metadata in `NxGN-Solutions/capstone-tools` for the requested `--channel` (`stable` or `preview`) and reports `installedVersion`, `latestVersion`, `updateAvailable`, `channel`, and `releaseUrl`.

If the distribution repo is private, set one of `CAPSTONE_TOOLING_RELEASES_TOKEN`, `CAPSTONE_RELEASES_TOKEN`, `GITHUB_TOKEN`, or `GH_TOKEN` before running `cap update check`. Public stable releases do not require a token.

For manual packaging, set `CLI_RELEASE_URL` before running `scripts/publish-cli.sh` when the final GitHub Release URL is already known.

---

## GitHub Actions Release Builds

Native AOT release builds must run on matching operating systems:

- `osx-arm64` on macOS
- `linux-x64` on Linux
- `win-x64` on Windows

The workflow at `.github/workflows/cli-release.yml` builds both CLI and MCP packages, writes a combined `manifest.json`, updates the binary-only distribution repo docs/skills, and creates the GitHub Release in that distribution repo. Configure `CAPSTONE_TOOLING_RELEASES_TOKEN` as a source-repo secret with write access to the distribution repo before running it.

---

## Troubleshooting

### AOT publish fails with "cannot find clang"

Install Xcode command line tools: `xcode-select --install`

### Binary is unexpectedly large

Windows self-contained builds are ~73 MB because they include the .NET runtime. macOS AOT builds are ~11 MB. This is normal.

### "Not authenticated" after sharing

Each machine needs its own `cap auth login` — credentials are stored in `~/.cap/credentials.json` and are not portable.

### Binary won't run on Intel Mac

The default build targets `osx-arm64` (Apple Silicon). For Intel Macs, build with `-r osx-x64`.

---

## See Also

- [Cross-Platform CLI Guide](platform-guide.md) — shell syntax differences
- [Commands Reference](commands.md) — full command listing
- [CLI CLAUDE.md](../../../NxGN.Capstone.Cli/CLAUDE.md) — developer-facing CLI docs
