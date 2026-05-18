# Cross-Platform CLI Guide

The Capstone CLI (`cap`) runs on Windows, macOS, and Linux. Commands are identical across platforms — only shell syntax differs.

## CLI Invocation

| Context | Command |
|---------|---------|
| Pre-built binary (on PATH) | `cap <command>` |
| Pre-built binary (local) | `./cap` (macOS/Linux) or `.\cap.exe` (Windows) |
| Source checkout | `dotnet run --no-build --project NxGN.Capstone.Cli -- <command>` |

All documentation uses `cap` as the command name. Substitute with the appropriate invocation for your setup.

## Shell Syntax Reference

### Environment Variables

**Prefer `cap config set`** — this is cross-platform and persists across sessions:

```
cap config set api-url https://your-server:port
cap config set tenant your-tenant-id
cap config set output json
```

When environment variables are needed:

| | macOS / Linux (bash/zsh) | Windows (PowerShell) |
|--|--------------------------|----------------------|
| Set | `export CAPSTONE_API_URL="https://..."` | `$env:CAPSTONE_API_URL = "https://..."` |
| Read | `echo $CAPSTONE_API_URL` | `echo $env:CAPSTONE_API_URL` |
| Unset | `unset CAPSTONE_API_URL` | `Remove-Item Env:\CAPSTONE_API_URL` |

### Writing JSON to a File

**macOS / Linux:**
```bash
cat > entity.json << 'EOF'
{
  "name": "My Entity",
  "description": "..."
}
EOF
```

**Windows PowerShell:**
```powershell
@'
{
  "name": "My Entity",
  "description": "..."
}
'@ | Set-Content -Path entity.json
```

**Cross-platform alternative:** Write JSON using your editor or Claude's Write tool, then reference with `--file entity.json`.

### Piping and JSON Processing

**macOS / Linux (with jq):**
```bash
cap model inputs list --json | jq '.data[] | {id, name}'
```

**Windows PowerShell:**
```powershell
$result = cap model inputs list --json | ConvertFrom-Json
$result.data | Select-Object id, name
```

**Cross-platform (preferred in recipes):** Save to file and read:
```
cap model inputs list --json > inputs.json
```
Then read and parse `inputs.json` with your tools.

### Capturing Command Output

**macOS / Linux:**
```bash
METRIC_ID=$(cap model inputs create --file input.json --json | jq -r '.id')
cap model inputs get $METRIC_ID --json
```

**Windows PowerShell:**
```powershell
$result = cap model inputs create --file input.json --json | ConvertFrom-Json
$METRIC_ID = $result.id
cap model inputs get $METRIC_ID --json
```

### File Paths

| | macOS / Linux | Windows |
|--|---------------|---------|
| Temp directory | `/tmp/` | `$env:TEMP\` |
| Home directory | `~/` or `$HOME/` | `$HOME\` or `$env:USERPROFILE\` |
| Config directory | `~/.cap/` | `$HOME\.cap\` |
| Path separator | `/` | `\` (but `/` works in most contexts) |

**Prefer relative paths** (e.g., `workspace/scratch/entity.json`) — these work on all platforms.

### Command Chaining

| | macOS / Linux | Windows PowerShell |
|--|---------------|---------------------|
| Run if previous succeeded | `cmd1 && cmd2` | `cmd1; if ($LASTEXITCODE -eq 0) { cmd2 }` |
| Run regardless | `cmd1; cmd2` | `cmd1; cmd2` |
| Pipe output | `cmd1 \| cmd2` | `cmd1 \| cmd2` |

### Redirect Output to File

| | macOS / Linux | Windows PowerShell |
|--|---------------|---------------------|
| Overwrite | `cap ... --json > file.json` | `cap ... --json \| Set-Content file.json` |
| Append | `cap ... --json >> file.json` | `cap ... --json \| Add-Content file.json` |

> **Note:** `>` redirection works in PowerShell but may produce UTF-16 encoded files. Use `Set-Content` for reliable UTF-8 output.

## Recipe Conventions

Recipes in `docs/cli/recipes/` follow these cross-platform conventions:

1. **`cap` as command name** — substitute your invocation method
2. **`cap config set` for configuration** — instead of shell-specific `export`
3. **File-based JSON input** — save to file, then `--file`, instead of heredocs
4. **Relative paths** — `workspace/scratch/` instead of `/tmp/`
5. **`--json` with file redirect** — instead of inline `jq` pipelines

Where a recipe shows shell-specific syntax, adapt using the tables above.
