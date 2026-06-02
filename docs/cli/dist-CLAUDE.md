# Capstone CLI

The Capstone CLI (`cap`) is a command-line tool for managing and analyzing data in the Capstone platform. This distribution contains a pre-built executable — no .NET SDK required.

## CRITICAL: Detect Your Shell and Adapt

Before running any commands, **detect the current shell** and use the correct syntax throughout the session.

| Shell | CLI invocation | Detect with |
|-------|---------------|-------------|
| **PowerShell** (Windows) | `.\cap.exe <command>` | `$PSVersionTable` exists |
| **cmd.exe** (Windows) | `.\cap.exe <command>` | `%OS%` exists |
| **bash/zsh** (macOS/Linux) | `./cap <command>` | `echo $SHELL` |

For the full cross-platform syntax reference, see `docs/reference/platform-guide.md`.

### Quick Reference — Shell Differences

| Task | bash / zsh | PowerShell |
|------|-----------|------------|
| Set env var | `export VAR="value"` | `$env:VAR = "value"` |
| Set config (cross-platform) | `cap config set key value` | `cap config set key value` |
| Write JSON to file | `cat > file.json << 'EOF'`...`EOF` | `@'`...`'@ \| Set-Content file.json` |
| Redirect output | `cap ... --json > file.json` | `cap ... --json \| Set-Content file.json` |
| Parse JSON | `cap ... --json \| jq '.field'` | `(cap ... --json \| ConvertFrom-Json).field` |
| Capture into variable | `ID=$(cap ... --json \| jq -r '.id')` | `$ID = (cap ... --json \| ConvertFrom-Json).id` |
| Temp directory | `/tmp/` | `$env:TEMP\` |
| Home directory | `~/` | `$HOME\` |

**Prefer cross-platform patterns:**
- Use `cap config set` instead of environment variables where possible
- Use the Write tool to create JSON files, then `--file` flag (works everywhere)
- Use relative paths (`workspace/scratch/`) instead of `/tmp/`

## First-Run Setup

On first use, **ask the user which Capstone server they want to connect to**. They will provide the URL (e.g., `https://capstone.example.com` or `https://localhost:7184`).

**1. Configure the API URL (user provides the URL):**

```
cap config set api-url <URL-FROM-USER>
```

**2. Log in via OAuth (opens browser):**

```
cap auth login
```

**3. Verify connection:**

```
cap auth whoami
```

This shows your username, current tenant, and accessible tenants.

**4. Switch tenant (if needed):**

```
cap auth tenants                    # List available tenants
cap auth switch-tenant <tenant-id>  # Switch to a specific tenant
```

> If any command returns a connection error, ask the user to confirm the API URL is correct and the server is reachable.

## Automation Error Contract

Use `--json` for machine-readable workflows and branch on exit codes, not
free-text stderr. Run `cap schema --json` for the live contract.

| Exit | Meaning |
|------|---------|
| `0` | Success |
| `1` | Local CLI failure or user cancellation |
| `2` | Validation/client-actionable failure, including not found and non-retriable API 4xx |
| `3` | Authentication or tenant failure |
| `4` | Server/API/network/rate-limit/malformed-response failure |
| `5` | Timeout |
| `6` | Partial result |

## Workspace & Tenant Cache

This workspace uses a structured directory to cache tenant data and build working memory across sessions. **This is critical for efficiency — always check the cache before querying the API.**

### Directory Structure

```
workspace/
├── {tenant-name}/
│   ├── context.md              # Working notes: what you've learned about this tenant
│   ├── model/
│   │   ├── metrics.json        # Cached: cap model metrics list --json
│   │   ├── inputs.json         # Cached: cap model inputs list --json
│   │   └── calculations.json   # Cached: cap model calculations list --json
│   ├── masterdata/
│   │   ├── org-nodes.json      # Cached: cap masterdata org-nodes list --json
│   │   ├── disciplines.json    # Cached: cap masterdata disciplines list --json
│   │   ├── frameworks.json     # Cached: cap masterdata frameworks list --json
│   │   └── units.json          # Cached: cap masterdata units list --json
│   ├── templates/
│   │   ├── widget-templates.json     # Cached: cap templates widget-templates list --json
│   │   ├── dashboard-templates.json  # Cached: cap templates dashboard-templates list --json
│   │   └── capture-templates.json    # Cached: cap templates capture-templates list --json
│   ├── periods/
│   │   ├── daily.json          # Cached: cap data time-periods list --data-interval day --json
│   │   ├── weekly.json         # Cached: cap data time-periods list --data-interval week --json
│   │   └── monthly.json        # Cached: cap data time-periods list --data-interval month --json
│   └── scratch/                # Temp JSON files for create/save operations
```

### Cache Lifecycle

**On first connection to a tenant (or when `workspace/{tenant}/` doesn't exist):**

1. Create the directory structure
2. Run discovery commands and cache the results — redirect each command's `--json` output to the corresponding file
3. Write initial `context.md` with a summary of what was discovered (org structure, metric count, available periods)

**On subsequent sessions:**

1. Read `workspace/{tenant}/context.md` first — this is your memory of this tenant
2. Read cached JSON files instead of running list commands
3. Only re-query the API when you need fresh data or the user says something has changed

**When you create or modify entities:**

1. After a successful `create` or `save`, refresh the relevant cache file
2. Update `context.md` with what changed and why

**When the user says "refresh" or "re-cache":**

1. Re-run all discovery commands and overwrite cached files
2. Update `context.md` with the refresh timestamp

### context.md — Tenant Working Memory

This is the most important file. Write observations, not just data. Examples of what to record:

```markdown
# {Tenant Name} — Working Context
Last updated: 2026-02-13

## Org Structure
- 4 org nodes: Enterprise → Example Site 1, Example Site 2, Example Site 3
- Enterprise ID: <id>

## Key Metrics
- 14 inputs (production volumes, costs, revenues)
- 10 calculations (margins, ratios, change %)
- All metrics use "bottles" (btl) as primary unit

## Dashboards
- D1 "Executive P&L" — 3 tabs (Monthly, Weekly, Daily), 25 widgets
- D2 "OEE Dollar Bridge" — single tab, 8 widgets

## Observations
- Weekly periods start on Mondays, format: "(W6) Feb 2 2026"
- GM% calculation uses AfterAgg phase — don't set orgAgg/timeAgg
- Example Site 3 has no data for January (MQTT feed started Feb 1)

## Session Log
- 2026-02-13: Initial setup, cached model. User explored D1 dashboard data.
- 2026-02-14: Created new widget template for throughput comparison.
```

### Rules for Using the Cache

1. **Always check cache first.** Before running `cap model inputs list`, read `workspace/{tenant}/model/inputs.json`
2. **Cache is authoritative for IDs.** Don't re-query just to get an entity ID — look it up in the cached list
3. **context.md is your brain.** Read it at the start of every session. Write to it when you learn something
4. **Scratch files are ephemeral.** Use `workspace/{tenant}/scratch/` for temp JSON payloads during create/save operations
5. **Never cache reporting data.** Widget data, computed values, and dashboard data are time-sensitive — always query live
6. **Stale is OK for structure.** Metrics, org nodes, and templates rarely change mid-session. Only refresh when the user explicitly modifies them or asks for a refresh

## Command Structure

Commands follow the pattern: `cap <domain> <entity> <action> [options]`

| Domain | Contains | Example |
|--------|----------|---------|
| `auth` | Login, logout, tenants | `cap auth login` |
| `config` | **CLI settings only** (api-url, tenant, format) | `cap config set api-url ...` |
| `model` | Inputs, calculations, overrides, metrics | `cap model inputs list` |
| `masterdata` | Org nodes, disciplines, frameworks, units | `cap masterdata org-nodes list` |
| `templates` | Widget, dashboard, capture, report templates | `cap templates widget-templates list` |
| `data` | Input values, change requests, time periods | `cap data input-values list` |
| `reporting` | Computed values, widget data, dashboard data | `cap reporting widgets get-data` |

> **Common mistake:** `config` is for CLI settings, NOT platform templates. Dashboard and widget templates are under `templates`.

**Standard CRUD actions per entity:**

```
cap <domain> <entity> list [--json] [--limit N]
cap <domain> <entity> get <id> [--json]
cap <domain> <entity> create --file input.json
cap <domain> <entity> save --file input.json
cap <domain> <entity> delete <id>
cap <domain> <entity> download-excel --output file.xlsx
cap <domain> <entity> upload-excel file.xlsx
```

## Machine Contracts And Workflows

Before planning unfamiliar automation, ask the CLI for the current contract:

```
cap schema --json
cap concepts --json
cap workflows list --json
```

Use `CAPSTONE_OUTPUT_VERSION=2` only when a workflow expects the shared
`{ ok, data, error, warnings, meta }` envelope. Leave it unset for legacy
per-command JSON shapes.

The CLI uses the same identity semantics for JSON batch/upsert and Excel upload:

| Entity shape | Match key |
|--------------|-----------|
| Flat entities | Exact full name only after trimming leading/trailing whitespace; case-sensitive |
| Tree entities | Exact full path only after trimming leading/trailing whitespace; case-sensitive |

Do not use fuzzy matching, aliases, partial names, or tree-node short names for
upsert identity. Read `upsertIdentity` from `cap schema --json` when you need
to confirm the current contract.

## Model Build And Restore Workflows

Prefer rerunnable JSON imports for tenant builds:

```
cap masterdata units import-json --file units.json --upsert --json
cap masterdata org-nodes import-json --file org-nodes.json --upsert --json
cap masterdata disciplines import-json --file disciplines.json --upsert --json
cap masterdata frameworks import-json --file frameworks.json --upsert --json
cap model inputs import-json --file inputs.json --upsert --json
cap model calculations validate-batch --file calculations.json --upsert --json
cap model calculations import-json --file calculations.json --upsert --json
cap templates widget-templates import-json --file widget-templates.json --upsert --json
cap templates dashboard-templates import-json --file dashboard-templates.json --upsert --json
```

Use Excel upload when users need spreadsheet review and row/cell diagnostics;
use JSON import when agents need source-controlled payloads, dry-run plans, and
repeatable builds.

For snapshots and reviewed restore planning:

```
cap system tenants snapshot --output snapshot --json
cap workflows show restore-from-snapshot --json
```

Snapshot export is read-only. Restore is not an automatic command; follow the
workflow's dry-run, validate, apply, and audit sequence.

## JSON Output — Critical Details

**`--json` on `get` commands** returns the entity wrapped in a domain key:
```json
{"tenant": "...", "widgetTemplate": {...}}
```

**`--json` on `list` commands** returns a paginated envelope (NOT a bare array):
```json
{"tenant": "...", "data": [...items...], "pagination": {"offset": 0, "limit": 100, "total": 70}}
```

> Always access `.data` to get the items array from list commands.
> Text output truncates IDs (e.g., `<id>`). Use `--json` for full UUIDs.

## Reporting Data — Period Formats

Widget data interval is auto-detected from the widget template. Period strings must match exactly:

| Widget dataInterval | Period format | Example |
|--------------------|---------------|---------|
| Day | `YYYY-MM-DD` | `--periods "2026-01-15"` |
| Week | `(WN) Mon DD YYYY` | `--periods "(W5) Jan 26 2026"` |
| Month | `Mon YYYY` | `--periods "Jan 2026"` |

Discover available periods with:
```
cap data time-periods list --data-interval day
cap data time-periods list --data-interval week
cap data time-periods list --data-interval month
```

> **Mismatch = empty results.** If you get no data rows, verify the period format matches the widget's interval.

After model or seed changes, wait for recalculation and use audit commands for
automation-safe analysis:

```
cap data recalculation wait <target-model-version> --json
cap reporting computed-values audit --metrics <metric-id> --data-interval month --periods <period> --include-metric-details --strict --json
cap templates dashboard-templates audit <dashboard-template-id> --strict --json
```

## Recipes — Read Before Multi-Step Tasks

For standard workflows, **read the recipe first** from `docs/recipes/`. Recipes use `cap` as the command name — substitute your platform's invocation. See `docs/reference/platform-guide.md` for translation rules.

| Task | Recipe |
|------|--------|
| Explore what data exists | `docs/recipes/exploration/discover-tenant-structure.md` |
| Analyze a specific metric | `docs/recipes/exploration/talk-to-your-data.md` |
| Get dashboard insights | `docs/recipes/exploration/talk-to-your-dashboard.md` |
| Compare locations | `docs/recipes/analysis/compare-operations.md` |
| Compare dashboards across time/location | `docs/recipes/analysis/dashboard-comparison.md` |
| Deep cross-metric analysis | `docs/recipes/analysis/deep-dashboard-analysis.md` |
| Find trends | `docs/recipes/analysis/find-trends.md` |
| Spot anomalies | `docs/recipes/analysis/spot-anomalies.md` |
| Track actuals vs targets | `docs/recipes/analysis/target-gap-analysis.md` |
| Verify data flows | `docs/recipes/reporting/verify-data.md` |
| Generate written summary | `docs/recipes/reporting/generate-narrative.md` |
| Create a metric | `docs/recipes/configuration/create-metric.md` |
| Create a calculation | `docs/recipes/configuration/create-calculation.md` |
| Create a widget | `docs/recipes/configuration/create-widget-template.md` |
| Build a dashboard | `docs/recipes/configuration/build-dashboard.md` |
| Enter data | `docs/recipes/data-management/enter-data.md` |
| Bulk import from Excel | `docs/recipes/data-management/bulk-import.md` |
| Check data completeness | `docs/recipes/exploration/check-completeness.md` |

## Reference Documentation

| Document | Contents |
|----------|----------|
| `docs/SKILL.md` | Full concept glossary, domain vocabulary mapping, all commands by domain |
| `docs/reference/platform-guide.md` | Cross-platform shell syntax translation (bash vs PowerShell) |
| `docs/reference/commands.md` | Quick command lookup |
| `docs/reference/glossary.md` | Term definitions with examples |
| `docs/reference/model-building.md` | Enum lookup tables, payload templates |
| `docs/reference/output-formats.md` | Table vs JSON output format details |
| `docs/reference/template-selection.md` | Choose the right template workflow |

## Configuration & Credentials

| File | Purpose |
|------|---------|
| `~/.cap/config.json` | API URL, default tenant, output format, timeout |
| `~/.cap/credentials.json` | OAuth tokens, current tenant (managed by `auth login`) |

**Environment variable overrides** (take precedence over config). Prefer `cap config set` instead — it's cross-platform:

| Variable | Overrides |
|----------|-----------|
| `CAPSTONE_API_URL` | API base URL |
| `CAPSTONE_TENANT` | Default tenant |
| `CAPSTONE_OUTPUT` | Output format (`table` or `json`) |
| `CAPSTONE_API_KEY` | API key (ephemeral, not persisted) |

## Key Concepts

| You Say | Capstone Calls It | CLI Domain |
|---------|-------------------|------------|
| Location, site, facility | Org Node | `masterdata org-nodes` |
| KPI, measure, indicator | Metric | `model metrics` |
| Manual data point | Input | `model inputs` |
| Formula-based value | Calculation | `model calculations` |
| Category, type, topic | Discipline | `masterdata disciplines` |
| Reporting standard | Framework | `masterdata frameworks` |
| Unit, UoM | Unit of Measure | `masterdata units` |
| Chart, visualization | Widget | `reporting widgets` |
| Chart config | Widget Template | `templates widget-templates` |
| Dashboard layout | Dashboard Template | `templates dashboard-templates` |
| Data entry form | Capture Template | `templates capture-templates` |

> For the full concept glossary with 21 mappings, see `docs/SKILL.md`.

## Tips

- All commands support `--json` for machine-readable output — prefer this when parsing results
- Run `cap schema --json` and `cap workflows list --json` before generating automation
- Use `import-json --upsert --dry-run --json` before applying large model-build changes
- Use `cap status` to see current auth state, tenant, and config at a glance
- Token refresh is automatic — if a token expires mid-session, the CLI refreshes it
- Excel import/export is available on most entities: `download-excel` and `upload-excel`
- When creating/saving entities, use `get --json > file.json`, edit the file, then `save --file file.json` for roundtrip editing
