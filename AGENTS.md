# Capstone Tooling Agent Guide

This repository contains binary distributions and agent-facing documentation for Capstone tools. It does not contain Capstone application source code.

The canonical public home for the latest tooling downloads, CLI skills, MCP
skills, and documentation is
`https://github.com/NxGN-Solutions/capstone-tools`. Point CLI users and agents
to that repository when they need current guidance.

## Choose The Right Tool

- Use the CLI (`cap`) for repeatable command-line work, scripting, JSON export/import, local cache workflows, and deterministic operations.
- Use the MCP server (`capstone-mcp`) when Claude Desktop should call Capstone tools directly through MCP.
- Use both when an implementation workflow needs Claude Desktop exploration plus repeatable CLI commands.

## First Steps

1. Identify the user's operating system and shell.
2. Download the matching zip from the latest environment-specific release when one exists (`demo-v<version>` or `prod-v<version>`), otherwise use the latest generic release.
3. Verify the `.sha256` checksum when possible.
4. Extract the package to a stable local path.
5. Use the included `config/` helpers when present; they already contain the correct environment API URL.
6. If no `config/` folder is present, ask the user for the Capstone API URL.
7. Authenticate through the user's browser or configured MCP login flow.

## CLI Usage Rules

- Prefer `--json` for all machine-readable commands.
- Set `CAPSTONE_OUTPUT_VERSION=2` when a workflow needs the shared
  `{ ok, data, error, warnings, meta }` JSON envelope from commands using the
  shared formatter. Leave it unset for legacy per-command JSON shapes.
- Run `cap schema --json`, `cap concepts --json`, and `cap workflows list --json` before planning unfamiliar work.
- Use the `exitCodes` section from `cap schema --json` instead of grepping
  stderr text.
- Interpret exit codes consistently: `0` success, `1` local CLI failure or
  cancellation, `2` validation/client-actionable failure, `3` auth/tenant
  failure, `4` server/API/network/rate-limit/malformed-response failure, `5`
  timeout, and `6` partial result.
- Use the `upsertIdentity` section from `cap schema --json` for imports:
  flat entities match by exact full name only after trimming leading/trailing
  whitespace, and tree entities match by exact full path only after trimming
  leading/trailing whitespace. Matching is case-sensitive. JSON batch/upsert
  workflows and Excel upload workflows use the same identity semantics. Use
  `upsertIdentity.surfaces` to choose JSON
  `import-json --upsert` or Excel `upload-excel` based on the diagnostics the
  workflow needs.
- For rerunnable tenant builds, prefer `import-json --upsert --json` for
  masterdata, model definitions, and widget/dashboard templates before falling
  back to one-item create/save loops.
- Use `cap system tenants snapshot --output snapshot --json` when a workflow
  needs a reviewable export of those same import-ready JSON surfaces plus a
  manifest. Treat restore as explicit reviewed `import-json --upsert` commands,
  not as an automatic mutation; use
  `cap workflows show restore-from-snapshot --json` for the dry-run, validate,
  apply, and audit sequence.
- After model or seed changes, run `cap data recalculation wait <version>
  --json`, then use `cap reporting computed-values audit --metrics <ids>
  --strict --json` for automation-safe output verification.
- Before live dashboard data checks, run `cap templates dashboard-templates
  audit <dashboard-template-id> --strict --json` to catch missing widget
  references or structural template issues.
- Cache tenant discovery data in a workspace folder before making repeated list calls.
- Keep create/save payloads in explicit JSON files under a workspace `scratch/` folder.
- After a successful mutation, refresh the relevant cache file and update notes.
- For environment-scoped packages, prefer `config/configure-cli.sh` or `config/configure-cli.ps1` over manually entering the API URL.

Load the full CLI skill from `skills/capstone-cli/SKILL.md`.

## MCP Usage Rules

- Configure Claude Desktop with an absolute path to `capstone-mcp` or `capstone-mcp.exe`.
- Set `CAPSTONE_API_URL` in the MCP server environment.
- For environment-scoped packages, start from `config/claude-desktop.capstone-mcp.json` and only replace the binary path.
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
- `environments/<slug>/manifest.json` - environment-specific release metadata for update checks.
- `LICENSE.md`, `SECURITY.md`, `SUPPORT.md` - distribution terms and support boundaries.
