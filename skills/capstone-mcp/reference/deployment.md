# MCP Distribution Guide

> Publish and consume prebuilt Capstone MCP packages without exposing application source code.

## Distribution Model

The client-facing tooling repository publishes the Capstone MCP server alongside the matching Capstone CLI. It contains binaries, checksums, manifests, agent guidance, and setup documentation. It must not contain application source code, tenant data, customer names, internal endpoint names, real IDs, or private implementation links.

Public client-facing releases are limited to:

| Environment | Release tag pattern | Manifest |
|-------------|---------------------|----------|
| Demo | `demo-v<version>` | `environments/demo/manifest.json` |
| Production | `prod-v<version>` | `environments/prod/manifest.json` |

Other environment packages are not published to the client-facing tooling repository.

The MCP server and CLI are released together because they use the same Capstone API contract surface. When the application version changes, publish matching CLI and MCP packages for each environment that has been deployed to that version.

---

## What Ships

Each MCP release includes one zip and one checksum per supported runtime:

| Runtime ID | Platform | Binary |
|-----------|----------|--------|
| `osx-arm64` | macOS Apple Silicon | `capstone-mcp` |
| `linux-x64` | Linux x64 | `capstone-mcp` |
| `win-x64` | Windows x64 | `capstone-mcp.exe` |

Environment-scoped packages include `config/claude-desktop.capstone-mcp.json`, which already contains the target environment API URL. Replace only the `command` value with the absolute path to the extracted MCP binary.

The package does not include credentials. Authentication remains per user and per machine.

---

## Configure Claude Desktop

1. Download the release that matches the environment you need to access.
2. Download the `capstone-mcp-{rid}.zip` asset for your operating system.
3. Verify the matching `.sha256` checksum when possible.
4. Extract the zip to a stable folder.
5. Copy the generated MCP server block into Claude Desktop configuration.
6. Replace the `command` value with the absolute binary path.
7. Restart Claude Desktop.
8. Ask Claude to log in to Capstone.

Example server block:

```json
{
  "mcpServers": {
    "capstone-prod": {
      "command": "/absolute/path/to/capstone-mcp",
      "env": {
        "CAPSTONE_API_URL": "<capstone-api-url>"
      }
    }
  }
}
```

Windows uses the absolute path to `capstone-mcp.exe`.

---

## Environment Versioning

Demo and production can intentionally run different application versions. Use the MCP release for the environment being accessed, not the newest release globally.

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

## Troubleshooting

### "Not authenticated" after sharing

Each machine needs its own OAuth login. Credential files are not portable.

### Claude Desktop does not see the server

Ensure the `command` value is an absolute path to the binary. Restart Claude Desktop after any config change.

### Wrong environment data appears

Confirm the MCP config uses the API URL for the environment the user expects. Re-download the matching environment package if needed.

---

## See Also

- [MCP Setup Guide](../setup.md)
- [Tools Reference](./tools.md)
- [CLI Distribution Guide](../../capstone-cli/reference/deployment.md)
