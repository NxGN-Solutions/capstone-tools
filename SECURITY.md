# Security Policy

## Reporting Vulnerabilities

Do not open public GitHub issues for vulnerabilities, credentials, tenant data, screenshots containing customer information, or private implementation details.

Report security concerns through your NxGN support, implementation, or account contact. Include:

- affected tool: CLI, MCP, documentation, or release process;
- affected version and operating system;
- clear reproduction steps;
- impact and scope;
- whether any credential or customer data may have been exposed.

NxGN will triage reports through its private support process.

## Credential Handling

- Never commit credentials, API keys, OAuth tokens, tenant exports, or customer data to this repository.
- The CLI stores credentials locally under `~/.cap/`.
- The MCP server stores credentials locally under `~/.capstone-mcp/`.
- Credential files are machine-local and must not be copied between users or machines.

## Download Integrity

Each release asset has a matching `.sha256` file. Verify checksums for production installs when possible.

The release `manifest.json` records the expected file names and checksums for automation.
