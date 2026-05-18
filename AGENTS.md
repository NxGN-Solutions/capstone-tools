# Capstone Tooling Agent Guide

This repository contains binary distributions and agent-facing documentation for Capstone tools. It does not contain Capstone application source code.

## Choose The Right Tool

- Use the CLI (`cap`) for repeatable command-line work, scripting, JSON export/import, local cache workflows, and deterministic operations.
- Use the MCP server (`capstone-mcp`) when Claude Desktop should call Capstone tools directly through MCP.
- Use both when an implementation workflow needs Claude Desktop exploration plus repeatable CLI commands.

## First Steps

1. Identify the user's operating system and shell.
2. Download the matching zip from the latest release.
3. Verify the `.sha256` checksum when possible.
4. Extract the package to a stable local path.
5. Ask the user for the Capstone API URL.
6. Authenticate through the user's browser or configured MCP login flow.

## CLI Usage Rules

- Prefer `--json` for all machine-readable commands.
- Run `cap schema --json`, `cap concepts --json`, and `cap workflows list --json` before planning unfamiliar work.
- Cache tenant discovery data in a workspace folder before making repeated list calls.
- Keep create/save payloads in explicit JSON files under a workspace `scratch/` folder.
- After a successful mutation, refresh the relevant cache file and update notes.

Load the full CLI skill from `skills/capstone-cli/SKILL.md`.

## MCP Usage Rules

- Configure Claude Desktop with an absolute path to `capstone-mcp` or `capstone-mcp.exe`.
- Set `CAPSTONE_API_URL` in the MCP server environment.
- Use MCP resources and prompts for exploration before making changes.
- Keep user authentication local to the user's machine; never copy credential files between machines.

Load the full MCP skill from `skills/capstone-mcp/SKILL.md`.

## Safety Boundaries

- Do not assume source-repository access is available.
- Do not ask users to paste tokens into chat.
- Do not embed API keys or GitHub tokens in config files committed to a repo.
- Do not bypass Capstone API authorization; the tooling should operate only as the authenticated user.
- Do not edit generated package contents as a substitute for updating the source docs and publishing a new release.

## Reference Map

- `README.md` - download and setup overview.
- `CLAUDE.md` - Claude-oriented setup and operating guidance.
- `docs/cli/` - CLI recipes and reference docs.
- `docs/mcp/` - MCP setup, tools, resources, prompts, and recipes.
- `manifest.json` - current release metadata and checksums.
- `LICENSE.md`, `SECURITY.md`, `SUPPORT.md` - distribution terms and support boundaries.
