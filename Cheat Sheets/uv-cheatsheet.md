---
tags: [cheat-sheet, python, uv, package-management]
---

# uv Cheatsheet

Quick reference for `uv` — a fast Python package and project manager written in Rust.

---

## Common Workflows

### Start a New Project
```bash
uv init my-etl-project
cd my-etl-project
uv python pin 3.12

uv add pandas polars duckdb
uv add --dev pytest ruff

uv run python -m pytest
```

### Set Up an Existing Project (from repo)
```bash
git clone https://github.com/org/project
cd project
uv sync                          # Reads uv.lock, installs everything
uv run python main.py
```

### One-Off Tool Execution (no install)
```bash
uvx ruff check .                 # Lint without installing ruff globally
uvx black --check .              # Format check without installing black
uvx mypy src/                    # Type check
```

### Audit and Upgrade All Outdated Dependencies
```bash
uv tree --outdated --depth=1      # See which direct deps have newer versions
uv lock --upgrade                 # Upgrade all packages in the lockfile
uv tree --outdated --depth=1      # Confirm what's still behind (intentional pins, etc.)
uv sync                           # Install the upgraded packages
```

### Upgrade a Specific Dependency
```bash
uv lock --upgrade-package pandas  # Update lockfile for pandas only
uv sync                           # Install the updated packages
```

### Export Requirements for Docker / CI
```bash
uv pip compile pyproject.toml -o requirements.txt      # Pinned prod deps
uv pip compile pyproject.toml --extra dev -o requirements-dev.txt  # With dev deps
```

### Replace a `requirements.txt` + `venv` Workflow
```bash
# Before (old way)
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# After (uv way)
uv venv && uv pip install -r requirements.txt
# Or better — use a proper project:
uv init && uv add pandas polars   # uv manages everything
```

### Use uv in a Makefile
```makefile
.PHONY: install test lint

install:
	uv sync

test:
	uv run python -m pytest

lint:
	uvx ruff check .
	uvx ruff format --check .
```

---

## Installation

| Command | Description | Example | Use Case |
|---------|-------------|---------|----------|
| `curl -LsSf https://astral.sh/uv/install.sh \| sh` | Install uv (macOS/Linux) | — | First-time setup |
| `brew install uv` | Install via Homebrew | `brew install uv` | Mac preferred install |
| `uv self update` | Update uv to latest version | `uv self update` | Keep uv current |
| `uv --version` | Check installed version | `uv --version` | Verify install |

---

## Project Management

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `uv init` | Create a new project with `pyproject.toml` | `--name` (project name), `--python` (version), `--no-readme` | `uv init my-project` | Start a new Python project |
| `uv init --app` | Create an application project | `--python` (version) | `uv init --app etl-pipeline` | Project meant to be run, not imported |
| `uv init --lib` | Create a library project | `--python` (version) | `uv init --lib my-utils` | Importable/publishable package |
| `uv add` | Add a dependency and install it | `--dev` (dev dep), `--optional` (extra), `--editable` | `uv add pandas` | Add package to project |
| `uv add --dev` | Add a development-only dependency | N/A | `uv add --dev pytest ruff` | Add test/lint tools |
| `uv remove` | Remove a dependency | N/A | `uv remove requests` | Uninstall and remove from config |
| `uv sync` | Install all dependencies from lockfile | `--frozen` (no update), `--no-dev` (skip dev deps) | `uv sync` | Set up environment from lockfile |
| `uv lock` | Generate or update `uv.lock` without installing | `--upgrade` (all), `--upgrade-package` (one) | `uv lock --upgrade` | Refresh lockfile |
| `uv run` | Run a command inside the project environment | `--python` (version), `--with` (extra package) | `uv run python script.py` | Execute without activating env |
| `uv tree` | Show dependency tree | `--depth` (max levels) | `uv tree` | Inspect package relationships |

---

## Python Version Management

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `uv python list` | List available Python versions | `--all-versions` | `uv python list` | See what's installable |
| `uv python install` | Download and install a Python version | N/A | `uv python install 3.12` | Install specific Python |
| `uv python pin` | Pin the Python version for the project | N/A | `uv python pin 3.11` | Lock project to a version |
| `uv python find` | Show path to a Python executable | N/A | `uv python find 3.12` | Find where Python lives |

---

## Virtual Environments

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `uv venv` | Create a virtual environment | `--python` (version), `-p` (short form), `--seed` (add pip/wheel) | `uv venv --python 3.12` | Create `.venv` in current dir |
| `uv venv .venv` | Create venv at a specific path | N/A | `uv venv .venv` | Explicit path (same as default) |
| `source .venv/bin/activate` | Activate the virtual environment | N/A | `source .venv/bin/activate` | Enter the env manually |
| `deactivate` | Deactivate the active environment | N/A | `deactivate` | Leave the env |

> **Tip:** In projects, `uv run` handles activation automatically — no need to manually activate.

---

## Package Installation (pip-compatible interface)

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `uv pip install` | Install packages (like `pip install`) | `-r` (requirements file), `--upgrade`, `--editable` | `uv pip install pandas polars` | Install packages into active env |
| `uv pip install -r` | Install from requirements file | N/A | `uv pip install -r requirements.txt` | Bulk install from file |
| `uv pip install -e .` | Install current project in editable mode | N/A | `uv pip install -e .` | Dev install of local package |
| `uv pip uninstall` | Remove packages | `-r` (from requirements file) | `uv pip uninstall pandas` | Remove packages from env |
| `uv pip list` | List installed packages | `--format` (json/columns/freeze) | `uv pip list` | See what's installed |
| `uv pip freeze` | Output installed packages in requirements format | N/A | `uv pip freeze > requirements.txt` | Export current env |
| `uv pip show` | Show details about a package | N/A | `uv pip show pandas` | Version, location, deps |
| `uv pip check` | Verify dependency compatibility | N/A | `uv pip check` | Find conflicts |
| `uv pip compile` | Resolve deps to a lockfile (like `pip-compile`) | `-o` (output file), `--upgrade` | `uv pip compile pyproject.toml -o requirements.txt` | Generate pinned requirements |

---

## Tool Management (Isolated CLI Tools)

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `uvx` | Run a tool in a temporary isolated env | `--from` (package name), `--python` | `uvx ruff check .` | One-off tool run, no install |
| `uv tool install` | Install a tool globally (persistent) | `--python` (version) | `uv tool install ruff` | Install CLI tool for repeated use |
| `uv tool run` | Run a tool (same as `uvx`) | N/A | `uv tool run black .` | Explicit form of `uvx` |
| `uv tool list` | List installed global tools | N/A | `uv tool list` | See what's globally installed |
| `uv tool uninstall` | Remove a globally installed tool | N/A | `uv tool uninstall ruff` | Remove global tool |
| `uv tool upgrade` | Upgrade a globally installed tool | `--all` (all tools) | `uv tool upgrade --all` | Update all global tools |

---

## Running Scripts

| Command | Description | Common Options | Example | Use Case |
|---------|-------------|----------------|---------|----------|
| `uv run script.py` | Run a Python script in project env | N/A | `uv run etl.py` | Run without activating env |
| `uv run --with pandas script.py` | Run script with an extra package (no project needed) | `--with` (extra dep), `--python` | `uv run --with polars analyze.py` | Inject a dep for one run |
| `uv run python -m pytest` | Run a module inside project env | N/A | `uv run python -m pytest` | Run tests without activating |
| `uv run --python 3.11 script.py` | Run script with a specific Python version | N/A | `uv run --python 3.11 script.py` | Test against a specific version |
