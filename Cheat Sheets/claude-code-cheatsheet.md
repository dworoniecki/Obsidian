# Claude Code Cheatsheet

Quick reference for Claude Code CLI commands, slash commands, keyboard shortcuts, hooks, MCP, and settings.

---

## Common Workflows

### Autonomous print mode
```bash
claude -p "refactor auth.py to use async/await" \
  --permission-mode auto-accept \
  --max-turns 10 \
  --max-budget-usd 2.00
```

### Continue last session non-interactively
```bash
claude -c -p "now write the tests"
```

### PR review
```bash
/review 123
# or
/review https://github.com/org/repo/pull/123
```

### Parallel isolated work
```bash
claude --worktree feature-payments   # Session 1
claude --worktree bugfix-auth        # Session 2 — separate branch, no conflicts
```

### Plan before executing a big change
```bash
claude --permission-mode plan "refactor the entire billing module"
# Claude shows the plan → you approve → it executes
```

### Run a bash command and include output in context
```
! git log --oneline -20
```

---

## CLI Invocation

| Command | Description |
|---------|-------------|
| `claude` | Start interactive session |
| `claude "query"` | Start with an initial prompt |
| `claude -p "query"` | Print mode — execute and exit (non-interactive) |
| `cat file \| claude -p "query"` | Process piped content |
| `claude -c` | Continue most recent session |
| `claude --resume` | Resume a session (interactive picker) |
| `claude update` | Update to latest version |

---

## Key CLI Flags

| Flag | Description |
|------|-------------|
| `--model sonnet\|opus\|haiku` | Select model by alias |
| `--model claude-sonnet-4-6` | Select model by full ID |
| `--model sonnet[1m]` | Use 1M token context variant |
| `--permission-mode plan` | Start in plan mode (show plan, require approval) |
| `--permission-mode auto-accept` | Auto-approve all tool use |
| `--dangerously-skip-permissions` | Skip all permission prompts |
| `--tools "Bash,Edit,Read"` | Restrict available tools |
| `--tools ""` | Disable all tools |
| `--allowedTools "Bash(git log *)"` | Auto-allow specific patterns |
| `--disallowedTools "Bash(curl *)"` | Block specific patterns |
| `--max-turns 3` | Limit agentic turns (print mode) |
| `--max-budget-usd 5.00` | Stop after spending $5 (print mode) |
| `--system-prompt "..."` | Override system prompt |
| `--append-system-prompt "..."` | Append to system prompt |
| `--add-dir ../lib` | Add extra working directories |
| `--mcp-config ./mcp.json` | Load MCP config file |
| `--worktree feature-x` | Run in isolated git worktree |
| `--output-format json` | JSON output (print mode) |
| `--verbose` | Show full turn-by-turn output |
| `--debug` | Enable debug logging |
| `-v / --version` | Show version |

---

## Slash Commands

### Session
| Command | Description |
|---------|-------------|
| `/clear` | Clear conversation and free context |
| `/compact [instructions]` | Compact context with optional focus |
| `/context` | Visualize context window usage |
| `/rename [name]` | Rename current session |
| `/resume [session]` | Resume a session by name/ID |
| `/fork [name]` | Fork conversation at current point |
| `/exit` / `/quit` | Exit session |

### Code & Git
| Command | Description |
|---------|-------------|
| `/review [PR number\|URL]` | Review a pull request |
| `/pr-comments [PR]` | Fetch PR comments |
| `/security-review` | Run security analysis on pending changes |
| `/diff` | Interactive diff viewer (Left/Right = sources, Up/Down = files) |
| `/init` | Initialize CLAUDE.md for the project |

### Info & Diagnostics
| Command | Description |
|---------|-------------|
| `/help` | Show help and available commands |
| `/cost` | Token and cost usage for session |
| `/usage` | Plan limits and rate status |
| `/stats` | Daily usage, sessions, streaks |
| `/doctor` | Diagnose installation and settings |
| `/status` | Version, model, account, connectivity |

### Configuration
| Command | Description |
|---------|-------------|
| `/config` / `/settings` | Open settings interface |
| `/keybindings` | Edit keybindings.json |
| `/permissions` | View and update tool permissions |
| `/hooks` | Manage hook configurations |
| `/model [model]` | Switch model (Left/Right arrows adjust effort) |
| `/theme` | Change color theme |
| `/vim` | Toggle vim mode |
| `/terminal-setup` | Configure Shift+Enter for multiline input |
| `/memory` | Edit CLAUDE.md and manage auto-memory |

### Modes & Features
| Command | Description |
|---------|-------------|
| `/plan` | Enter plan mode |
| `/fast [on\|off]` | Toggle fast mode |
| `/rewind` / `/checkpoint` | Rewind conversation and restore code |
| `/tasks` | List and manage background tasks |
| `/copy` | Copy last response to clipboard |
| `/export [filename]` | Export conversation as text |
| `/add-dir <path>` | Add a working directory |
| `/mcp` | Manage MCP servers and OAuth |
| `/ide` | Manage IDE integrations |

---

## Keyboard Shortcuts

### General
| Shortcut | Action |
|----------|--------|
| `Enter` | Submit prompt |
| `Ctrl+C` | Cancel input or generation |
| `Ctrl+D` | Exit session |
| `Ctrl+L` | Clear screen (keeps history) |
| `Ctrl+O` | Toggle verbose transcript |
| `Ctrl+R` | Reverse history search |
| `Ctrl+G` | Open prompt in external editor |
| `Ctrl+B` | Background current task |
| `Ctrl+T` | Toggle task list |
| `Shift+Tab` | Cycle permission modes |
| `Esc Esc` | Rewind / checkpoint dialog |
| `Up` / `Down` | Navigate prompt history |
| `Option+P` (Mac) | Model picker |
| `Option+T` (Mac) | Toggle extended thinking |

### Multiline Input
| Method | Keystroke |
|--------|-----------|
| Quick escape | `\` + `Enter` |
| macOS default | `Option+Enter` |
| Shift+Enter (most terminals) | Run `/terminal-setup` to configure |

### Quick Prefixes (in prompt)
| Prefix | Effect |
|--------|--------|
| `/` | Slash command or skill |
| `!` | Run bash command directly and add to context |
| `@` | File path autocomplete |

---

## Vim Mode

Enable with `/vim`. Two modes: INSERT and NORMAL.

### Navigation (NORMAL)
| Key | Action |
|-----|--------|
| `h/j/k/l` | Left / Down / Up / Right |
| `w/e/b` | Next word / end of word / previous word |
| `0 / $ / ^` | Start / end / first non-blank of line |
| `gg / G` | Start / end of input |
| `f{char}` | Jump to next occurrence of char |

### Editing (NORMAL)
| Key | Action |
|-----|--------|
| `x` | Delete character |
| `dd / D` | Delete line / to end of line |
| `cc / C` | Change line / to end of line |
| `cw / dw` | Change / delete word |
| `yy` | Yank (copy) line |
| `p / P` | Paste after / before |
| `.` | Repeat last change |
| `J` | Join lines |

### Text Objects (combine with `d`, `c`, `y`)
| Object | Targets |
|--------|---------|
| `iw / aw` | Inner / around word |
| `i" / a"` | Inner / around double quotes |
| `i( / a(` | Inner / around parentheses |
| `i{ / a{` | Inner / around braces |

---

## Permission Modes

| Mode | Behavior |
|------|----------|
| **Normal** (default) | Prompts for confirmation on tool use |
| **Plan Mode** | Shows a plan, requires approval before execution |
| **Auto-Accept** | Auto-approves all tool use — use carefully |

Switch with `Shift+Tab` or `/permissions`.

---

## Hooks

Hooks run shell commands (or HTTP calls) at lifecycle events. Configure in `~/.claude/settings.json` or `.claude/settings.json`.

### Hook Events
| Event | Fires When | Can Block? |
|-------|-----------|------------|
| `SessionStart` | Session begins | Yes |
| `UserPromptSubmit` | User submits a prompt | Yes |
| `PreToolUse` | Tool is about to execute | Yes |
| `PermissionRequest` | Permission prompt triggered | Yes |
| `PostToolUse` | Tool completed successfully | No |
| `PostToolUseFailure` | Tool failed | No |
| `Stop` | Session ends | Yes |
| `Notification` | Claude needs attention | No |
| `PreCompact` | Before context compaction | No |

### Example Hook Config
```json
{
  "hooks": {
    "format-on-save": {
      "event": "PostToolUse",
      "matcher": { "tool": "Edit" },
      "handler": {
        "type": "command",
        "command": "~/.claude/hooks/format.sh"
      }
    }
  }
}
```

---

## MCP Servers

### Add a Server
```bash
# HTTP (recommended for remote)
claude mcp add --transport http <name> <url>

# Stdio (local process)
claude mcp add --transport stdio <name> -- <command> [args]

# With auth header
claude mcp add --transport http notion https://mcp.notion.com/mcp \
  --header "Authorization: Bearer TOKEN"
```

### Scopes
| Scope | Location | Shared? |
|-------|----------|---------|
| `local` | `~/.claude.json` (per project) | No |
| `project` | `.mcp.json` (committed to repo) | Yes |
| `user` | `~/.claude.json` (all projects) | No |

```bash
claude mcp add --scope project --transport http stripe https://mcp.stripe.com
```

### Management
```bash
claude mcp list           # List all servers
claude mcp get <name>     # Details for a server
claude mcp remove <name>  # Remove a server
/mcp                      # In-session management and OAuth
```

### .mcp.json Example
```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "local-db": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@bytebase/dbhub"],
      "env": { "DSN": "postgresql://localhost/mydb" }
    }
  }
}
```

---

## Settings Files

| File | Scope |
|------|-------|
| `~/.claude/settings.json` | User — all projects |
| `.claude/settings.json` | Project — shared (commit this) |
| `.claude/settings.local.json` | Project — personal, not committed |
| `~/.claude/CLAUDE.md` | Global memory/instructions |
| `CLAUDE.md` | Project-level instructions (auto-loaded) |
| `.claude/CLAUDE.md` | Subfolder instructions |
| `~/.claude/keybindings.json` | Terminal keybinding overrides |
| `~/.claude/agents/` | Custom subagent definitions |

**Precedence (highest → lowest):** Managed → CLI flags → Local project → Shared project → User

### settings.json Example
```json
{
  "model": "claude-sonnet-4-6",
  "permissions": {
    "allow": ["Bash(npm run *)", "Bash(git log *)"],
    "ask":   ["Bash(git push *)"],
    "deny":  ["Bash(curl *)", "Read(.env)"]
  }
}
```

---

## Key Environment Variables

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | API key |
| `ANTHROPIC_MODEL` | Override default model |
| `CLAUDE_CODE_EFFORT_LEVEL` | `low` / `medium` / `high` |
| `CLAUDE_CODE_ENABLE_THINKING` | Enable extended thinking |
| `MAX_THINKING_TOKENS` | Extended thinking budget |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | Disable 1M context variants |
| `ENABLE_TOOL_SEARCH` | MCP tool search: `auto`, `true`, `false` |
| `MAX_MCP_OUTPUT_TOKENS` | MCP output token limit (default 25000) |
| `DISABLE_TELEMETRY` | Opt out of telemetry |
| `DISABLE_PROMPT_CACHING` | Disable prompt cache |

---

## Models

| Alias | Description |
|-------|-------------|
| `sonnet` | Claude Sonnet 4.6 — default, fast and capable |
| `opus` | Claude Opus 4.6 — most capable |
| `haiku` | Claude Haiku 4.5 — fastest, lowest cost |
| `sonnet[1m]` | Sonnet with 1M token context |
| `opusplan` | Opus for planning, Sonnet for execution |

Switch mid-session with `/model` or `Option+P`.
