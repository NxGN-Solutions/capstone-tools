# Recipe: Manage API Keys

> Create, list, rotate, and delete Identity-backed Capstone API keys through `cap auth apikey`.

## When to Use

- "Create an API key for the CLI / MCP / an integration"
- "Rotate this key"
- "Delete a disposable E2E key"
- "Give a key explicit data or template grants"

## Required Context

- [ ] Authenticated session (`cap auth login`) or an existing `CAPSTONE_API_KEY` with `ApiKeys.Create`
- [ ] At least one coverable role id (`cap` role lookup, or a known id)
- [ ] Optional: a grants JSON file (Manage callers only)

---

## Step-by-Step

### Step 1: Create

Roles are required. `--grants-file` is honored only for callers holding `ApiKeys.Manage`; otherwise the server snapshots the caller's own data/template grants. Selected template ids that are not in the session tenant are rejected (400).

```bash
cap auth apikey create \
  --description "CLI automation" \
  --expires 90 \
  --role <role-id> \
  --json
```

With grants (Manage):

```bash
cap auth apikey create \
  --description "scoped ingest" \
  --expires 30 \
  --role <role-id> \
  --grants-file grants.json \
  --json
```

`grants.json` camelCase collections (any may be empty):

```json
{
  "scopePermissions": [],
  "dataPermissions": [],
  "spreadsheetCaptureTemplates": [],
  "spreadsheetReportTemplates": [],
  "dashboardTemplates": [],
  "orgNodeTemplates": []
}
```

Non-JSON create prints the one-time secret on stdout. JSON redacts it.

### Step 2: List

```bash
cap auth apikey list --json
```

### Step 3: Rotate

The previous secret stops working immediately (Identity hard-tombstone, no grace). Grants are copied to the successor.

```bash
cap auth apikey rotate <api-key-id> --force --json
```

Non-JSON rotate prints the new secret on stdout.

### Step 4: Delete

```bash
cap auth apikey delete <api-key-id> --force --json
```

`prune` remains for reviewed bulk cleanup (`--expired`, `--name-prefix`, `--older-than-days`; dry-run by default).
