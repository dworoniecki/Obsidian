# Cortex Code Cheatsheet

Quick reference for the Cortex Code CLI (`cortex`) and `snow cortex` commands, slash commands, keyboard shortcuts, hooks, MCP, and settings.

---

## Common Workflows

### One-shot non-interactive
```bash
cortex -p "List all tables in ANALYTICS_DB" -c production
```

### Pipe JSON output for scripting
```bash
cortex -p "How many active facilities?" --output-format stream-json | jq .
```

### Resume last session
```bash
cortex --continue
```

### Run a plan before executing a big change
```bash
cortex --plan "Refactor all dbt models in the marts/ layer to use new grain key"
# Cortex shows the plan → you approve → it executes
```

### Execute a task file
```bash
cortex -f generate_strat_report.txt -c production
```

### Execute SQL directly in session
```
/sql SELECT counterparty_name, SUM(upb) FROM ANALYTICS.FACILITIES GROUP BY 1 ORDER BY 2 DESC LIMIT 10
```

### Run a shell command in context
```
! git log --oneline -10
```

### Inject a Snowflake table into context
```
#ANALYTICS.PUBLIC.FACILITIES Write a query to get total UPB by counterparty
```

### Invoke the built-in self-help guide
```
$cortex-code-guide
```

### Compact a long session
```
/compact
```

---

## Installation

```bash
# macOS / Linux
curl -LsS https://ai.snowflake.com/static/cc-scripts/install.sh | sh
# Installs to ~/.local/bin/cortex

# Windows (PowerShell)
irm https://ai.snowflake.com/static/cc-scripts/install.ps1 | iex

# Update
cortex update              # Latest version
cortex update <version>    # Specific version
```

**Prerequisites:** Snowflake account with `SNOWFLAKE.CORTEX_USER` database role + Snowflake CLI installed.

---

## CLI Invocation

| Command | Description |
|---------|-------------|
| `cortex` | Start interactive session |
| `cortex -p "prompt"` | Print mode — execute and exit (non-interactive) |
| `cortex -f task.txt` | Execute prompt from file and exit |
| `cortex --continue` | Resume most recent conversation |
| `cortex --resume` | List recent sessions interactively and pick one |
| `cortex --resume <id>` | Resume specific session by ID |
| `cortex --resume last` | Resume the last session |
| `cortex update` | Update to latest version |

---

## Key CLI Flags

| Flag | Description |
|------|-------------|
| `-c, --connection <name>` | Named connection from `connections.toml` |
| `-w, --workdir <path>` | Set working directory for file operations |
| `-m, --model <model>` | Select AI model (default: `auto`) |
| `--plan` | Require approval before all actions |
| `--bypass` | Auto-approve all planned actions |
| `--dangerously-allow-all-tool-calls` | Disable all tool permission prompts |
| `-r, --resume <id>` | Resume session by ID |
| `--continue` | Resume most recent session |
| `-p, --print "<prompt>"` | One-shot prompt, print response, exit |
| `-f, --file <file>` | Read prompt from file, execute, exit |
| `--output-format stream-json` | JSON output for scripting/piping |
| `-V, --version` | Show installed version |
| `--help` | Show CLI help |

---

## Slash Commands

### Session
| Command | Description |
|---------|-------------|
| `/help` | Show help |
| `/compact` | Compress conversation context while preserving key information |
| `/new` | Start a new session |
| `/rename <title>` | Rename current session |
| `/resume` / `/r` / `/sessions` | List and resume previous sessions |
| `/rewind` | Step back N turns (interactive picker) |
| `/fork` | Fork current session into a new one |
| `/clear` / `/cls` | Clear the screen |
| `/feedback` | Submit session feedback |
| `/update` | Update Cortex Code CLI |
| `/exit` / `/quit` | Exit session |

### Model & Planning Mode
| Command | Description |
|---------|-------------|
| `/model` | Show current model or interactively select one |
| `/plan` | Enable planning mode (requires approval before actions) |
| `/plan_off` | Disable planning mode |
| `/bypass` | Enable bypass mode (auto-approve all) |
| `/bypass-off` | Disable bypass mode |
| `/status` | Show current configuration status |

### SQL & Data
| Command | Description |
|---------|-------------|
| `/sql <query>` | Execute a SQL query directly |
| `/sql <query> --limit <n>` | Execute SQL, limit displayed rows |
| `/table [<file>]` / `/csv` | Open interactive table viewer |
| `/connections` / `/conn` | Manage Snowflake connections |

### Development Tools
| Command | Description |
|---------|-------------|
| `/sh <cmd>` / `! <cmd>` | Execute shell command |
| `/diff` / `/changes` / `/review` | Review git changes |
| `/worktree` | Manage git worktrees |
| `/dbt` | dbt operations |
| `/lineage` | dbt lineage visualization |
| `/tasks` | Show current task list |

### Configuration & Extensibility
| Command | Description |
|---------|-------------|
| `/settings` | View or modify settings interactively |
| `/theme` | Select color theme |
| `/add-dir <path>` | Add a working directory |
| `/skill` / `/skills` | Manage and list skills |
| `/mcp` / `/mcp-status` | MCP server status |
| `/hooks` | View hooks configuration |
| `/commands` / `/cmds` | Manage custom slash commands |
| `/agents` | View running subagents |
| `/sandbox` | Manage sandbox settings |

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Cancel current operation |
| `Ctrl+C Ctrl+C` | Exit Cortex Code CLI |
| `Ctrl+L` | Clear terminal screen |
| `Ctrl+B` | Background the current long-running command |
| `Ctrl+J` | Insert a newline in the prompt |
| `Shift+Tab` | Cycle through Normal / Plan / Bypass modes |
| `Up / Down` | Navigate command history |
| `Tab` | Command / path completion |

### Quick Prefixes (in prompt)
| Prefix | Effect | Example |
|--------|--------|---------|
| `/` | Slash command | `/sql SELECT ...` |
| `! <cmd>` | Run shell command and add output to context | `!git status` |
| `@<path>` | Inject file contents into context | `@src/models/loan.sql` |
| `@<path>$N-M` | Inject specific line range of a file | `@src/app.py$10-50` |
| `#<DB.SCHEMA.TABLE>` | Inject Snowflake table schema + sample rows | `#ANALYTICS.PUBLIC.FACILITIES` |
| `$<skill-name>` | Activate a skill inline | `$machine-learning` |

---

## Permission / Planning Modes

| Mode | Behavior |
|------|----------|
| **Normal** (default) | Prompts for confirmation on sensitive tool use |
| **Plan Mode** (`--plan`) | Shows a plan, requires approval before execution |
| **Bypass Mode** (`--bypass`) | Auto-approves all actions — use carefully |

Switch mid-session with `/plan`, `/plan_off`, `/bypass`, `/bypass-off`.

---

## Built-in Agent Tools

Tools operate under tiered permission levels (auto-approved → requires confirmation):

| Category | Tools |
|----------|-------|
| File (auto) | `Read`, `Write`, `Edit`, `Glob`, `Grep` |
| Shell | `Bash` (2 min default, 10 min max) |
| Agent | `RunSubagent`, `AskUserQuestion`, `Review` |
| Web | `WebSearch`, `WebFetch` |
| Snowflake | `SnowflakeSqlExecute`, `SnowflakeObjectSearch`, `SnowflakeProductDocs`, `ReflectSemanticModel`, `SnowflakeMultiCortexAnalyst` |
| Data | `DataDiff`, `NotebookExecute`, `NotebookEdit` |
| Plan / Memory | `EnterPlanMode`, `ExitPlanMode`, `Memory`* |

*Memory requires `CORTEX_ENABLE_MEMORY=true`

---

## Hooks

Hooks run shell commands at lifecycle events. Configure in `~/.snowflake/cortex/hooks.json`.

### Hook Events
| Event | Fires When |
|-------|-----------|
| `SessionStart` | Session begins |
| `PreToolUse` | Tool is about to execute |
| `PostToolUse` | Tool completed |
| `PermissionRequest` | Permission prompt triggered |

---

## MCP Servers

```bash
cortex mcp list            # List all servers
cortex mcp add             # Interactive setup
cortex mcp remove <name>   # Remove a server
/mcp                       # In-session MCP status viewer
```

Edit `~/.snowflake/cortex/mcp.json` directly for manual config.

---

## Configuration Files

All stored under `~/.snowflake/cortex/`:

| File | Purpose |
|------|---------|
| `settings.json` | Main settings |
| `permissions.json` | Tool access permissions |
| `mcp.json` | MCP server configurations |
| `hooks.json` | Lifecycle hook configuration |
| `conversations/` | Session history |
| `skills/` | Custom skills |
| `commands/` | Custom slash commands |

Shared with Snowflake CLI:
- `~/.snowflake/connections.toml` — connection definitions

### settings.json Example
```json
{
  "compactMode": false,
  "autoUpdate": true,
  "theme": "dark"
}
```

### permissions.json Example
```json
{
  "onlyAllow": ["Read", "Glob", "Grep"],
  "defaultMode": "ask",
  "dangerouslyAllowAll": false
}
```

---

## Key Environment Variables

| Variable | Purpose |
|----------|---------|
| `SNOWFLAKE_HOME` | Override default `~/.snowflake` directory |
| `SNOWFLAKE_DEFAULT_CONNECTION_NAME` | Set default connection name |
| `CORTEX_AGENT_MODEL` | Override model selection |
| `CORTEX_ENABLE_MEMORY` | Enable memory tool across sessions (`true`) |
| `COCO_DANGEROUS_MODE_REQUIRE_SQL_WRITE_PERMISSION` | Require confirmation for SQL writes |

---

## Settings Precedence (Highest → Lowest)

Managed (org policy) → CLI flags → In-session commands → Env vars → Config files → Defaults

---

## Models

Use `/model` in session to browse interactively. `auto` = Snowflake picks best available.

| Provider | Model Name |
|----------|------------|
| Snowflake | `auto` (default — recommended) |
| Anthropic | `claude-4-opus`, `claude-4-sonnet`, `claude-3-7-sonnet` |
| Meta | `llama4-maverick`, `llama4-scout`, `llama3.3-70b`, `llama3.1-405b` |
| Mistral | `mistral-large2`, `mixtral-8x7b` |
| DeepSeek | `deepseek-r1` |
| OpenAI | `openai-gpt-4.1`, `openai-o4-mini` |
| Snowflake | `snowflake-arctic`, `snowflake-llama-3.3-70b` |

---

## Extensibility

- **Skills** — inject domain context via Markdown files in `.cortex/skills/<name>/SKILL.md`
- **Subagents** — specialized autonomous agents (Markdown + YAML frontmatter)
- **Custom commands** — `/cmds` to register project-specific slash commands
- **Hooks** — shell scripts triggered on lifecycle events (see above)
- **MCP** — connect external tools via `cortex mcp add` or editing `mcp.json`

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | General error |
| `2` | Configuration error |
| `3` | Connection error |
| `4` | Permission denied |
| `130` | Interrupted (Ctrl+C) |

---

## `snow cortex` Commands (Snowflake CLI)

These are the Snowflake CLI commands for Cortex LLM functions — separate from the `cortex` CLI above.

**Install Snowflake CLI:** `brew install snowflake-cli` or `pipx install snowflake-cli`

### Subcommands

| Command | Description |
|---------|-------------|
| `snow cortex complete "<text>"` | Ask a question / run a prompt |
| `snow cortex complete --file conv.json` | Multi-turn conversation from JSON file |
| `snow cortex summarize "<text>"` | Summarize English text |
| `snow cortex summarize --file doc.txt` | Summarize from file |
| `snow cortex extract-answer "<q>" "<text>"` | Extract specific answer from text |
| `snow cortex sentiment "<text>"` | Analyze sentiment (positive/negative/neutral) |
| `snow cortex translate "<text>"` | Translate text between languages |

### Key Flags for `snow cortex complete`

| Flag | Default | Description |
|------|---------|-------------|
| `--model TEXT` | `llama3.1-70b` | LLM to use |
| `--backend [sql\|rest]` | `rest` | Execution backend |
| `--file FILE` | — | JSON conversation history (multi-turn) |
| `-c, --connection TEXT` | `default` | Named connection from `config.toml` |
| `--role TEXT` | — | Snowflake role to use |
| `--warehouse TEXT` | — | Warehouse to use |
| `--format [TABLE\|JSON\|CSV]` | `TABLE` | Output format |

### Examples
```bash
snow cortex complete "What is a warehouse lending facility?" -c myconn
snow cortex complete "What is an advance rate?" --model claude-4-sonnet -c myconn
snow cortex complete --file conversation.json -c myconn
snow cortex summarize --file deal_memo.txt -c myconn
snow cortex extract-answer "What is the advance rate?" "The facility has a 75% advance rate..." -c myconn
snow cortex sentiment "The borrower has consistently met obligations." -c myconn
```

### Multi-turn Conversation JSON Format
```json
[
  {"role": "user", "content": "What is UPB?"},
  {"role": "assistant", "content": "UPB stands for Unpaid Principal Balance..."},
  {"role": "user", "content": "How is it used in warehouse lending?"}
]
```
