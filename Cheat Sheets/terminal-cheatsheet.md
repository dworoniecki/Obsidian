# Terminal Commands Cheatsheet for Data Engineering

## Common Workflows

| Command | Description | Example | Use Case |
|---------|-------------|---------|----------|
| `\|` (pipe) | Chain commands | `grep ERROR log \| wc -l` | Count error lines |
| `&&` | Run if previous succeeds | `dbt run && dbt test` | Run tests after models |
| `\|\|` | Run if previous fails | `command1 \|\| echo "Failed"` | Error handling |
| `>` | Redirect output to file | `ls > file_list.txt` | Save command output |
| `>>` | Append output to file | `echo "new line" >> log.txt` | Add to existing file |

### Process Monitoring (top & htop)
```bash
# --- top (built-in, no install needed) ---
top                    # Open interactive process viewer (sorted by CPU by default)
top -o cpu             # Sort by CPU usage (highest first)
top -o mem             # Sort by memory usage (highest first)
top -o rsize           # Sort by real memory size
top -u username        # Show only processes for a specific user

# While inside top, press:
#   o          → type a sort column (e.g. "cpu" or "mem") then Enter
#   q          → quit

# --- htop (better UI — install with: brew install htop) ---
htop                   # Open colorful interactive viewer
# While inside htop, press:
#   F6 (or >)  → choose sort column (CPU%, MEM%, etc.)
#   F9         → kill selected process
#   /          → search/filter by process name
#   u          → filter by user
#   t          → toggle tree view (see parent/child processes)
#   Space      → tag a process
#   q          → quit

# --- Quick one-liners (no interactive UI) ---
ps aux --sort=-%cpu | head -10    # Top 10 processes by CPU (Linux)
ps aux --sort=-%mem | head -10    # Top 10 processes by memory (Linux)
ps aux | sort -k3 -nr | head -10  # Top 10 by CPU% (Mac-compatible)
ps aux | sort -k4 -nr | head -10  # Top 10 by MEM% (Mac-compatible)
```

### Clear Directory Contents (Keep Folder)
```bash
# Clear all contents from a directory without deleting the directory itself
rm -rf /path/to/folder/*           # Delete all files and subdirectories
rm -rf /path/to/folder/.*          # Delete all hidden files (be careful!)
rm -rf /path/to/folder/* /path/to/folder/.*  # Delete everything including hidden

# Safer approach - clear specific types
rm -f /path/to/folder/*.csv        # Delete only CSV files
rm -rf /path/to/folder/temp_*      # Delete only items starting with "temp_"

# Clear current directory contents
rm -rf *                           # Delete everything in current directory
```

### Quick Data Analysis
```bash
# Count records in CSV
wc -l data.csv

# Get unique values in column 2
cut -d',' -f2 data.csv | sort | uniq

# Sum values in column 3
awk -F',' '{sum += $3} END {print sum}' data.csv

# Find files modified in last 24 hours
find . -name "*.csv" -mtime -1
```

### Log Analysis
```bash
# Find errors in dbt logs
grep -i error ~/.dbt/logs/dbt.log

# Count errors by type
grep ERROR app.log | awk '{print $4}' | sort | uniq -c

# Monitor live log file
tail -f /var/log/application.log
```

### File Management
```bash
# Create backup with timestamp
cp important_data.csv "backup_$(date +%Y%m%d).csv"

# Find large files
find . -size +100M -type f

# Clean up old log files
find logs/ -name "*.log" -mtime +30 -delete
```

## History & Command Tricks

```bash
# Re-run last command with sudo (instead of pressing up, home, typing sudo)
sudo !!

# Use last argument of previous command
mkdir my-project
cd !$               # expands to "cd my-project"

# Fix a typo in the last command without re-typing it
git chekcout main
^chekcout^checkout  # runs "git checkout main" immediately

# Ctrl+R — reverse search through your entire history
# Start typing any fragment; it finds the most recent match
# Press Ctrl+R again to cycle older matches
# Press Esc to edit before running, Enter to run immediately
(reverse-i-search)`docker': docker-compose up -d --build
```

### Expand History Size (add to ~/.bashrc or ~/.zshrc)
```bash
HISTSIZE=10000
HISTFILESIZE=20000
HISTCONTROL=ignoredups:erasedups   # no duplicate entries
```

## Keyboard Shortcuts

| Shortcut | Description |
|----------|-------------|
| `Ctrl + A` | Jump to beginning of line |
| `Ctrl + E` | Jump to end of line |
| `Ctrl + W` | Delete one word backward |
| `Ctrl + U` | Clear everything before cursor |
| `Ctrl + K` | Clear everything after cursor |
| `Ctrl + L` | Clear the screen (same as `clear`) |
| `Alt + F` | Jump forward one word |
| `Alt + B` | Jump backward one word |
| `Ctrl + C` | Cancel current command |
| `Ctrl + Z` | Suspend current process (resume with `fg`) |

> Combo: `Ctrl+A` to jump to line start, then `Ctrl+K` to delete the whole thing — faster than holding backspace.

## Navigation Commands

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `cd` | Change directory | `-` (previous dir) | `cd /Users/username/data` | Navigate to data folder |
| `ls` | List directory contents | `-l` (long format), `-a` (show hidden), `-h` (human readable), `-t` (sort by time), `-r` (reverse) | `ls -la` | Show all files with details |
| `pwd` | Print working directory | `-P` (physical path) | `pwd` | Check current location |
| `~` | Home directory shortcut | N/A | `cd ~/projects` | Navigate to home/projects |
| `.` | Current directory | N/A | `cp file.csv .` | Copy file to current location |
| `..` | Parent directory | N/A | `cd ../..` | Go up two directory levels |

## File Operations

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `cp` | Copy files/directories | `-r` (recursive), `-i` (interactive), `-v` (verbose), `-p` (preserve) | `cp -rv data/ backup/` | Create backup of data folder |
| `mv` | Move/rename files | `-i` (interactive), `-v` (verbose), `-n` (no overwrite) | `mv old_name.csv new_name.csv` | Rename data files |
| `rm` | Remove files | `-r` (recursive), `-f` (force), `-i` (interactive), `-v` (verbose), `-rf` (force recursive) | `rm -rf temp_folder/` | Delete folder and all contents forcefully |
| `mkdir` | Create directory | `-p` (create parents), `-v` (verbose), `-m` (mode/permissions) | `mkdir -p data/processed/2024` | Create nested folder structure |
| `rmdir` | Remove empty directory | `-p` (remove parents), `-v` (verbose), `--ignore-fail-on-non-empty` | `rmdir -p empty/nested/folders` | Clean up empty folder hierarchy |
| `touch` | Create empty file | `-t` (set timestamp), `-c` (no create) | `touch new_script.py` | Create new Python script |

## File Viewing Commands

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `cat` | Display entire file | `-n` (line numbers), `-A` (show all chars), `-b` (number non-blank) | `cat -n borrower_data.csv` | View file with line numbers |
| `less` | View file with pagination | `/text` (search), `G` (go to end), `g` (go to start), `-N` (line numbers) | `less -N large_dataset.csv` | Browse large files with line numbers |
| `head` | Show first lines | `-n` (number of lines), `-c` (number of bytes) | `head -n 20 data.csv` | Preview first 20 lines |
| `tail` | Show last lines | `-n` (number of lines), `-f` (follow), `-c` (number of bytes) | `tail -f -n 100 dbt.log` | Monitor last 100 lines of log |
| `grep` | Search text patterns | `-i` (ignore case), `-v` (invert), `-n` (line numbers), `-r` (recursive), `-c` (count) | `grep -i "error" pipeline.log` | Find errors (case insensitive) |

## Text Processing Commands

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `awk` | Pattern scanning and processing | `-F` (field separator), `-v` (variable), `-f` (script file) | `awk -F',' '$3 > 100000' loans.csv` | Filter loans over $100K |
| `sed` | Stream editor for filtering/transforming | `-i` (in-place), `-e` (expression), `-n` (quiet), `-r` (extended regex) | `sed -i 's/NULL/0/g' data.csv` | Replace NULL values in file |
| `cut` | Extract columns from delimited files | `-d` (delimiter), `-f` (fields), `-c` (characters), `--complement` (exclude fields) | `cut -d',' -f1,3,5 data.csv` | Select columns 1, 3, and 5 |
| `sort` | Sort lines of text | `-n` (numeric), `-r` (reverse), `-k` (key/column), `-u` (unique), `-t` (delimiter) | `sort -k2 -nr balances.csv` | Sort by column 2, numeric, descending |
| `uniq` | Report or omit repeated lines | `-c` (count), `-d` (duplicates only), `-u` (unique only), `-i` (ignore case) | `sort states.txt \| uniq -c` | Count occurrences of each state |
| `wc` | Word, line, character count | `-l` (lines), `-w` (words), `-c` (characters), `-m` (characters) | `wc -l borrower_data.csv` | Count records in file |
| `tr` | Translate/transform characters | `-d` (delete), `-s` (squeeze repeats), `-c` (complement) | `echo $PATH \| tr ':' '\n'` | Convert PATH to readable list |
| `nl` | Number lines of input | `-b` (body numbering), `-v` (starting number), `-s` (separator) | `cat file.txt \| nl` | Add line numbers to file |

## System Monitoring Commands

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `uptime` | Show how long system has been running, logged-in users, and load averages (1, 5, 15 min) | N/A | `uptime` | Check if machine is under sustained heavy load |
| `df` | Display filesystem disk space | `-h` (human readable), `-T` (filesystem type), `-i` (inodes) | `df -h` | Check available storage |
| `du` | Display directory space usage | `-h` (human readable), `-s` (summary), `-a` (all files), `-d` (max depth) | `du -sh data/*` | Check size of data folders |
| `htop` | Interactive process viewer (color UI, mouse support) | `F6` (sort by column), `F9` (kill process), `/` (search), `Space` (tag process), `u` (filter by user), `t` (tree view), `q` (quit) | `htop` | Monitor CPU/memory by process interactively |
| `lsof` | List open files | `-i` (network), `-p` (process ID), `-u` (user), `-c` (command) | `lsof -i :5432` | Check PostgreSQL connections |
| `ps` | Show running processes | `aux` (all users), `-ef` (full format), `-u` (user), `--sort` (sort by) | `ps aux \| grep python` | Find Python processes |
| `top` | Display running processes sorted by resource usage | `-o cpu` (sort by CPU), `-o mem` (sort by memory), `-o rsize` (sort by real memory), `-u username` (filter by user), `-pid 1234` (filter by PID) | `top -o mem` | Find processes using the most memory |

## Process Management

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `kill` | Terminate process by PID | `-9` (force kill), `-15` (terminate), `-l` (list signals) | `kill -9 1234` | Force stop hung process |
| `ctrl+c` | Interrupt current process | N/A | `ctrl+c` | Stop running command |
| `ctrl+z` | Suspend current process | N/A | `ctrl+z` | Pause long-running job |
| `fg` | Bring suspended/background job to foreground | job number (e.g. `fg %2`) | `fg` | Resume a paused process |
| `bg` | Resume suspended job in the background | job number | `bg` | Keep running after Ctrl+Z without blocking terminal |
| `jobs` | List all background/suspended jobs | `-l` (include PIDs) | `jobs` | See what's running in background |
| `&` | Run command in background immediately | N/A | `npm run build &` | Free the terminal while a long process runs |

## File Permissions

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `chmod` | Change file permissions | `+x` (add execute), `-w` (remove write), `755` (octal), `-R` (recursive) | `chmod +x script.py` | Make script executable |
| `chown` | Change file ownership | `-R` (recursive), `-h` (no dereference) | `chown -R user:group data/` | Change folder ownership |
| `ls -la` | List with permissions | `-l` (long format), `-a` (all files), `-h` (human readable) | `ls -la` | Check file permissions |

## Help and Information

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `man` | Manual pages | `-k` (keyword search), `-f` (short description) | `man grep` | Get detailed command help |
| `--help` | Command help flag | N/A | `awk --help` | Quick command reference |
| `which` | Locate command | `-a` (all instances) | `which python` | Find Python installation |
| `whereis` | Locate binary/source/manual | `-b` (binary), `-m` (manual), `-s` (source) | `whereis -b dbt` | Find dbt binary location |

## Package Management (Mac)

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `brew install` | Install package | `--cask` (GUI apps), `--HEAD` (latest dev), `--force` (reinstall) | `brew install --cask docker` | Install Docker Desktop |
| `brew update` | Update Homebrew | N/A | `brew update` | Get latest package info |
| `brew upgrade` | Upgrade packages | `--dry-run` (preview), `package_name` (specific) | `brew upgrade python` | Update specific package |
| `brew list` | List installed packages | `--cask` (GUI apps), `--versions` (show versions) | `brew list --versions` | See installed packages with versions |

## Environment and Variables

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `export` | Set environment variable | `-p` (print all), `-n` (remove variable) | `export DBT_PROFILES_DIR=/path` | Set dbt config path |
| `echo` | Display text/variables | `-n` (no newline), `-e` (enable escapes) | `echo $PATH` | Check PATH variable |
| `env` | Display all environment variables | `-i` (ignore environment), `-u` (unset variable) | `env \| grep DBT` | Find dbt-related variables |

## Git Commands

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `git clone` | Clone repository | `--depth` (shallow), `--branch` (specific branch) | `git clone --depth 1 repo-url` | Download project (latest only) |
| `git pull` | Update local repository | `--rebase` (rebase), `--force` (force update) | `git pull --rebase origin main` | Get latest changes with rebase |
| `git status` | Check repository status | `-s` (short format), `-b` (branch info) | `git status -s` | Quick status check |
| `git add` | Stage changes | `-A` (all changes), `-p` (patch mode), `.` (current dir) | `git add -A` | Stage all changes |
| `git commit` | Commit changes | `-m` (message), `-a` (all tracked), `--amend` (modify last) | `git commit -am "Fix model"` | Commit all tracked files |
| `git push` | Push changes to remote | `-u` (set upstream), `--force` (force push), `--tags` (include tags) | `git push -u origin feature-branch` | Push and set upstream |

## SSH Commands

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `ssh-keygen` | Generate SSH key pair | `-t` (type), `-b` (bits), `-C` (comment), `-f` (filename) | `ssh-keygen -t ed25519 -C "email"` | Create authentication keys |
| `ssh-copy-id` | Copy public key to server | `-i` (identity file), `-p` (port) | `ssh-copy-id -i ~/.ssh/id_rsa user@server` | Setup passwordless login |
| `ssh` | Connect to remote server | `-p` (port), `-i` (identity file), `-L` (local forward), `-A` (agent forward) | `ssh -A user@production-server` | Access remote with agent forwarding |
| `ssh-add` | Add key to SSH agent | `-l` (list keys), `-d` (delete), `-D` (delete all) | `ssh-add -l` | List loaded keys |

## Network Tools

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `curl` | Transfer data from servers | `-X` (method), `-H` (header), `-d` (data), `-o` (output), `-v` (verbose) | `curl -X POST -H "Content-Type: application/json" api-endpoint` | Test API with headers |
| `wget` | Download files from web | `-O` (output file), `-r` (recursive), `-c` (continue), `-q` (quiet) | `wget -O data.csv https://source.com/file` | Download with custom name |
| `netstat` | Display network connections | `-a` (all), `-n` (numeric), `-t` (TCP), `-u` (UDP), `-l` (listening) | `netstat -ant \| grep 5432` | Check TCP connections to port |
| `ping` | Test network connectivity to a host | `-c` (count), `-i` (interval), `-t` (ttl) | `ping -c 4 google.com` | Verify host is reachable |
| `ifconfig` | Show network interface configuration | N/A | `ifconfig` | Check IP addresses and interfaces |

## Advanced Text Editing

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `vim` | Text editor | `-r` (recover), `+n` (start at line), `-o` (split horizontal) | `vim +25 config.yml` | Edit file starting at line 25 |
| `nano` | Simple text editor | `-l` (line numbers), `-w` (no wrap), `-B` (backup) | `nano -l script.py` | Edit with line numbers |

## Scheduling

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `crontab -e` | Edit cron jobs | `-u` (user), `-l` (list), `-r` (remove all) | `crontab -e` | Schedule automated tasks |
| `crontab -l` | List cron jobs | `-u` (specific user) | `crontab -l` | View scheduled jobs |

## Clipboard Tools

```bash
# macOS — pipe any output directly to clipboard (no mouse needed)
cat some-file.txt | pbcopy
cat ~/.ssh/id_rsa.pub | pbcopy   # copy SSH public key
pwd | pbcopy                      # copy current path

# Linux equivalent
cat some-file.txt | xclip -selection clipboard
```

## Aliases Starter Pack (add to ~/.bashrc or ~/.zshrc)

```bash
# Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ~='cd ~'
alias -- -='cd -'          # go back to previous directory (browser back button)

# Listing
alias ll='ls -alF'
alias lt='ls -ltr'         # list by time, newest last

# Safety net (prompt before overwriting)
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Git shortcuts
alias gs='git status'
alias ga='git add .'
alias gc='git commit -m'
alias gp='git push'
alias gl='git log --oneline --graph --decorate'
alias gco='git checkout'

# Reload shell config
alias reload='source ~/.bashrc'   # or source ~/.zshrc

# After adding aliases, activate immediately:
# run: source ~/.bashrc  (or use the reload alias above)
```

### Shell Functions (more powerful than aliases — can take arguments)

```bash
# cd + ls in one command — add to ~/.bashrc or ~/.zshrc
function cl() {
    cd "$1" && ls -la
}
# Usage: cl my-project
```

## tmux — Persistent Terminal Sessions

Keeps processes alive when SSH drops. Reconnect and pick up exactly where you left off.

```bash
# Session management
tmux new -s myproject       # start a named session
tmux attach -t myproject    # reattach to existing session
tmux ls                     # list all sessions
Ctrl + B, then D            # detach (session keeps running)

# Pane splitting (inside tmux)
Ctrl + B, then %            # split vertically
Ctrl + B, then "            # split horizontally
Ctrl + B, then arrow        # switch between panes
```

## fzf — Fuzzy Finder

Install: `brew install fzf` (macOS) or `sudo apt install fzf` (Ubuntu/Debian)

Once installed, `Ctrl+R` becomes an interactive history search — see all matches at once, filter in real time.

```bash
vim $(fzf)                              # fuzzy-search and open a file
git checkout $(git branch | fzf)        # switch git branches interactively
kill $(ps aux | fzf | awk '{print $2}') # kill a process interactively
```