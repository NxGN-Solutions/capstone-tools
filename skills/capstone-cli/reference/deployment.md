# CLI Distribution Guide

> Publish and consume prebuilt Capstone CLI packages without exposing application source code.

## Distribution Model

The client-facing tooling repository publishes prebuilt Capstone CLI and MCP server packages, agent guidance, user documentation, checksums, and release manifests. It must not contain application source code, tenant data, customer names, internal endpoint names, real IDs, or private implementation links.

Public client-facing releases are limited to:

| Environment | Release tag pattern | Manifest |
|-------------|---------------------|----------|
| Demo | `demo-v<version>` | `environments/demo/manifest.json` |
| Production | `prod-v<version>` | `environments/prod/manifest.json` |

Other environment packages are not published to the client-facing tooling repository.

The CLI and MCP server are released together because they use the same Capstone API contract surface. When the application version changes, publish matching CLI and MCP packages for each environment that has been deployed to that version.

---

## What Ships

Each CLI release includes one zip and one checksum per supported runtime:

| Runtime ID | Platform | Binary |
|-----------|----------|--------|
| `osx-arm64` | macOS Apple Silicon | `cap` |
| `linux-x64` | Linux x64 | `cap` |
| `win-x64` | Windows x64 | `cap.exe` |

Environment-scoped packages include a `config/` folder:

| File | Purpose |
|------|---------|
| `configure-cli.sh` | Configure the extracted CLI on macOS/Linux |
| `configure-cli.ps1` | Configure the extracted CLI on Windows |
| `capstone-env.sh` | Shell exports for macOS/Linux |
| `capstone-env.ps1` | Shell exports for PowerShell |
| `cli-config/config.json` | Preconfigured CLI config template |
| `claude-desktop.capstone-mcp.json` | MCP config template for Claude Desktop |

The package does not include credentials. Authentication remains per user and per machine.

---

## Install CLI

1. Download the release that matches the environment you need to access.
2. Download the `capstone-cli-{rid}.zip` asset for your operating system.
3. Verify the matching `.sha256` checksum when possible.
4. Extract the zip to a stable folder.
5. Run the included configuration helper when present.

macOS/Linux:

```bash
./config/configure-cli.sh
./cap auth login
./cap auth whoami
```

Windows PowerShell:

```powershell
.\config\configure-cli.ps1
.\cap.exe auth login
.\cap.exe auth whoami
```

For generic packages without a `config/` folder:

```bash
cap config set api-url <capstone-api-url>
cap auth login
```

---

## Update Checks

`cap update check --json` uses the environment manifest when `CAPSTONE_CLI_RELEASE_MANIFEST_URL` is set by the environment config helper. You can also pass a manifest URL explicitly:

```bash
cap update check --manifest-url https://raw.githubusercontent.com/NxGN-Solutions/capstone-tools/main/environments/prod/manifest.json --json
```

Update checks report the installed version, latest version, channel, release URL, and whether an update is available. The CLI does not silently self-update.

---

## Environment Versioning

Demo and production can intentionally run different application versions. Use the tooling release for the environment being accessed, not the newest release globally.

| User Group | Recommended Package |
|------------|---------------------|
| Consultants validating stable pre-production data | Latest `demo-v<version>` release |
| Production users and production support | Latest `prod-v<version>` release |
| Other internal environments | Private distribution channels |

---

## Publishing Guardrails

The public release process prepares documentation from this source tree and then audits the generated output before committing to the tooling repository. The audit must fail if generated public content contains:

- real IDs or UUIDs;
- known tenant or client names;
- internal source project names;
- private source documentation links;
- non-demo/prod public release entries;
- source build instructions intended only for private repository users.

If the audit fails, fix the source documentation or release metadata first. Do not patch the generated tooling repository by hand.

---

## See Also

- [Cross-Platform CLI Guide](platform-guide.md)
- [Commands Reference](commands.md)
- [MCP Deployment Guide](../../capstone-mcp/reference/deployment.md)
