# Capstone CLI Documentation — Start Here

The Capstone CLI (`cap`) manages and analyzes data in the Capstone platform.
This folder is the agent- and human-facing documentation for it.

## Connect First

Before any discovery or data command, establish a connection — empty results
almost always trace back to auth/tenant, not missing data:

```bash
cap config set api-url <URL>   # 1. Point at the Capstone server
cap auth login                 # 2. Log in via OAuth (opens browser)
cap auth whoami                # 3. Confirm identity, tenant, and accessible tenants
```

If something looks wrong (or every command returns empty), run `cap status` and
`cap auth doctor` — `auth doctor` diagnoses authentication state and prints the
recovery commands to run.

## Where to Go Next

| I want to... | Read |
|--------------|------|
| Get the agent on-ramp (shell detection, setup, workspace cache, error contract) | [dist-CLAUDE.md](./dist-CLAUDE.md) |
| Teach an agent the CLI (concept glossary, domain routing, command patterns, error handling) | [SKILL.md](./SKILL.md) |
| Follow a step-by-step task workflow | [recipes/README.md](./recipes/README.md) |
| Look up a specific command | [reference/commands.md](./reference/commands.md) |
| Translate syntax across Windows/macOS/Linux shells | [reference/platform-guide.md](./reference/platform-guide.md) |
| Map vocabulary (KPI → Metric, site → Org Node, …) | [reference/glossary.md](./reference/glossary.md) |

## Discovering Commands at Runtime

The CLI is self-describing — prefer these over guessing command names or flags:

```bash
cap schema --json     # Full machine-readable command index + automation contracts
cap concepts --json   # Compact Capstone domain concepts for planning
cap meta lookups list # All known lookup names across CLI domains (then: meta lookups get)
```

`cap schema --json` is authoritative; it includes exit-code semantics and the
`upsertIdentity` contract used for JSON batch/upsert and Excel uploads.

## Notes

- Most reporting commands require `--data-interval` and `--periods`.
- `notifications templates` (`create`/`get`/`list`/`save`/`delete`) administers
  notification templates; see [reference/commands.md](./reference/commands.md).
