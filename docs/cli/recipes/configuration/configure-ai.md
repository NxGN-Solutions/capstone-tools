# Recipe: Configure a Tenant's AI

> Turn AI on or off for a tenant, choose whose key pays (the NxGN default or the client's key), the provider and the model, test a key, and read usage, through `cap system tenants ai-config`.

## When to Use

- "Turn AI Summaries off for this tenant"
- "Use the client's own Anthropic key"
- "Try DeepSeek on a test tenant"
- "Why is AI unavailable for this tenant?"
- "How many AI requests did this tenant make last month?"

## Required Context

- [ ] Authenticated session (`cap auth login`) or `CAPSTONE_API_KEY`
- [ ] **System Manage** to change settings (**System View** to read them)
- [ ] For another tenant than your current one: **Tenants Scope Unrestricted**
- [ ] For the client's key: the key, in a file or ready to paste. Never put it on the command line

A tenant that has never saved AI settings uses the defaults: AI on, the NxGN default, Default model. The NxGN default is the model and key the deployment sets, and its provider follows that model (Claude Haiku 4.5 on Anthropic unless the deployment chooses another). With `--key-source nxgn`, `--provider` is ignored.

---

## Step-by-Step

### Step 1: See the current settings

```bash
cap system tenants ai-config get --tenant-id <tenant-id>
```

The output shows AI on/off, key source, provider, model (`default` follows the provider's default), whether a client key is stored (`API key : set (ending a91e)`), its status (`Healthy`, `Failing`, `Unknown`) and the last test. Usage has its own command, `ai-config usage`. The key itself is never shown.

### Step 2: Turn AI on or off

```bash
cap system tenants ai-config set --tenant-id <tenant-id> --ai off
```

`set` changes only the options you pass and keeps the rest. It saves the tenant, as the tenant page does: everything other than its AI settings goes back as it was, and an AI-only change doesn't recalculate the tenant's data.

### Step 3: Use the client's key

The key is read from standard input (or a hidden prompt on a terminal) and tested with the provider before it is stored. A key that fails the test is not stored, and the current settings stay in use.

```bash
cap system tenants ai-config set --tenant-id <tenant-id> \
  --ai on --key-source client --provider anthropic --api-key-stdin < client-key.txt
```

Anthropic keys must belong to a workspace. Changing the provider removes the stored key, so pass a new one with the provider change.

### Step 4: Try DeepSeek

To try DeepSeek on a tenant, use the client's key. The client's DeepSeek key needs a recorded confirmation that the tenant accepts its data location (the People's Republic of China). The CLI prints the notice before sending.

```bash
cap system tenants ai-config set --tenant-id <tenant-id> \
  --ai on --key-source client --provider deepseek --model deepseek-flash \
  --acknowledge-data-location --api-key-stdin < deepseek-key.txt
```

### Step 5: Choose a model

```bash
cap system tenants ai-config models --provider anthropic
cap system tenants ai-config set --tenant-id <tenant-id> --model claude-sonnet-5-5
cap system tenants ai-config set --tenant-id <tenant-id> --model default
```

`default` follows the provider's default, so the tenant moves when NxGN changes it.

### Step 6: Test a key

```bash
cap system tenants ai-config test --tenant-id <tenant-id>
cap system tenants ai-config test --tenant-id <tenant-id> --provider anthropic --api-key-stdin < new-key.txt
```

Without a key, `test` tests the key the tenant uses now. A passing test of a failing stored key restores AI at once. `test` exits 1 when the test fails. Tests are limited to 10 a minute per person.

### Step 7: Remove the client's key

```bash
cap system tenants ai-config set --tenant-id <tenant-id> --key-source nxgn
cap system tenants ai-config remove-key --tenant-id <tenant-id>
```

While AI uses the client's key, the key can't be removed: first turn AI off (`--ai off`) or switch to the NxGN default (`--key-source nxgn`). `remove-key` then forgets the stored key and leaves AI as it is. The key still works at the provider: ask the client to revoke it there.

### Step 8: Read usage

```bash
cap system tenants ai-config usage --tenant-id <tenant-id> --from 2026-09-01 --to 2026-09-30 --group-by model
```

Group by `feature`, `model` or `day`. Usage lists requests, tokens and failures; no prompt, response or key is recorded.

---

## When AI Summaries fail

`cap reporting dashboards get-insights` explains why AI didn't run:

| Code | Meaning | What to do |
|---|---|---|
| `ai_disabled` | AI is off for the tenant | `ai-config set --ai on` |
| `ai_unavailable` + reason | The key or settings can't work (`key_rejected`, `permission_denied`, `billing`, `model_unavailable`, `not_configured`) | Fix the key or settings; retrying won't help |
| `ai_busy` + reason | The provider is rate limiting or unreachable | Retry shortly |

## MCP

The MCP tools `system_tenantAiConfig_get`, `system_tenantAiConfig_set` and `system_tenantAiUsage_get` read and change the same settings, but never take a key or record the DeepSeek confirmation: a tool argument passes through an AI conversation. Use the tenant page or this CLI for those.

## Related

- [Commands reference](../../reference/commands.md#tenant-ai-configuration)
- [Manage API Keys](./manage-api-keys.md)
