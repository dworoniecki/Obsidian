---
tags: [cheat-sheet, dbt, sql, data-engineering]
---

# dbt Cheatsheet

Quick reference for dbt CLI commands, node selection, flags, and common workflows.

---

## Common Workflows

### Run a Single Model and Its Tests
```bash
dbt build -s my_model
```

### Run an Entire Folder
```bash
dbt run -s models/staging/
dbt run -s staging.*           # All models in the staging subdirectory
```

### Run All Models Downstream of a Source
```bash
dbt run -s source:raw_data+
```

### Full Refresh Incremental Models
```bash
dbt run --select "config.materialized:incremental" --full-refresh
```

### Run Only Failed Models from Last Run
```bash
dbt run -s "result:error+" --state ./target
```

### Pass Variables at Runtime
```bash
dbt run --vars '{"run_date": "2024-01-01", "env": "prod"}'

# In model SQL, reference with:
# WHERE date_col = '{{ var("run_date") }}'
```

### Tag-Based Scheduling
```bash
# In model config:
# {{ config(tags=["nightly", "finance"]) }}

dbt run --select "tag:nightly"
dbt run --select "tag:finance,tag:nightly"  # Models with BOTH tags
```

### Run a Macro Directly
```bash
dbt run-operation grant_select --args '{"role": "reporter"}'
dbt run-operation drop_old_relations --args '{"dry_run": false}'
```

### List All Models of a Type
```bash
dbt list -s "resource_type:model"
dbt list -s "resource_type:source"
dbt list -s "resource_type:test"
dbt list -s "resource_type:snapshot"
```

### Debug Connection Issues
```bash
dbt debug                          # Test connection and config
dbt debug --profiles-dir ~/.dbt/   # Test with specific profiles.yml location
```

---

## Main Commands

| Command | Description | Use Case |
|---------|-------------|----------|
| `dbt run` | Execute models in the project | Build tables/views in the warehouse |
| `dbt test` | Execute tests defined in the project | Validate data quality |
| `dbt build` | Run + test all selected resources (models, seeds, snapshots, tests) | Full build in one command |
| `dbt compile` | Compile models (generate SQL) without running them | Preview compiled SQL |
| `dbt seed` | Load CSV files from the `seeds/` folder into the database | Load reference/lookup data |
| `dbt snapshot` | Execute snapshot jobs to track slowly changing dimensions | Run SCD type 2 logic |
| `dbt source freshness` | Check if source data is fresh based on `loaded_at_field` | Validate source data timeliness |
| `dbt docs generate` | Generate the project documentation website | Build docs site |
| `dbt docs serve` | Start a local web server to browse documentation | View docs locally |
| `dbt deps` | Download and install packages from `packages.yml` | Install dbt packages |
| `dbt debug` | Test database connection and project config | Troubleshoot setup |
| `dbt init` | Initialize a new dbt project (Core only) | Start a new project |
| `dbt clean` | Delete folders in `clean-targets` list (e.g., `target/`, `dbt_packages/`) | Clear compiled artifacts |
| `dbt parse` | Parse the project and output timing info | Debug performance |
| `dbt list` | List all resources in the project (models, tests, seeds, etc.) | Explore project structure |
| `dbt show` | Preview rows from a model after transformation | Quick data preview |
| `dbt clone` | Clone selected nodes from a state to the target schema | Copy prod tables to dev |
| `dbt retry` | Retry the last dbt command from the point of failure | Resume after a partial failure |
| `dbt run-operation` | Invoke a macro directly | Run utility macros |

---

## Main Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--select` / `-s` | Select specific models/resources to run | `dbt run -s my_model` |
| `--exclude` | Exclude specific models from the selection | `dbt run -s finance.* --exclude finance.big_model` |
| `--selector` | Use a named selector from `selectors.yml` | `dbt run --selector nightly_models` |
| `--defer` | Use production artifacts for upstream models not in selection | `dbt run -s my_model --defer --state path/to/artifacts` |
| `--full-refresh` | Force recreate incremental models and seeds as tables | `dbt run --full-refresh` |
| `--threads` | Number of parallel threads to use | `dbt run --threads 8` |
| `--vars` | Pass variables to the project at runtime | `dbt run --vars '{"run_date": "2024-01-01"}'` |
| `--profiles-dir` | Override path to `profiles.yml` | `dbt run --profiles-dir ~/.dbt/` |
| `--project-dir` | Override path to `dbt_project.yml` | `dbt run --project-dir /path/to/project` |
| `--target` / `-t` | Override the active target (e.g., dev vs prod) | `dbt run --target prod` |
| `--fail-fast` | Stop immediately if any resource fails | `dbt run --fail-fast` |
| `--store-failures` | Store failed test rows in a table for inspection | `dbt test --store-failures` |
| `--warn-error` | Treat warnings as errors | `dbt run --warn-error` |
| `--log-level` | Set log verbosity (`debug`, `info`, `warn`, `error`) | `dbt run --log-level debug` |
| `--debug` | Show debug-level output in the terminal | `dbt run --debug` |
| `--no-compile` | Skip recompilation (with `dbt docs generate`) | `dbt docs generate --no-compile` |
| `--version` | Show installed dbt version | `dbt --version` |
| `--help` | Show available commands and flags | `dbt run --help` |

---

## Node Selection

### Commands that Support `--select`

| Command | Supported Flags |
|---------|----------------|
| `dbt run` | `--select`, `--exclude`, `--selector`, `--defer` |
| `dbt test` | `--select`, `--exclude`, `--selector`, `--defer` |
| `dbt build` | `--select`, `--exclude`, `--selector`, `--resource-type`, `--defer` |
| `dbt seed` | `--select`, `--exclude`, `--selector` |
| `dbt snapshot` | `--select`, `--exclude`, `--selector` |
| `dbt list` | `--select`, `--exclude`, `--selector`, `--resource-type` |
| `dbt compile` | `--select`, `--exclude`, `--selector` |
| `dbt source freshness` | `--select`, `--exclude`, `--selector` |
| `dbt docs generate` | `--select`, `--exclude`, `--selector` |

---

## Graph Operators

| Operator | Syntax | Description | Example |
|----------|--------|-------------|---------|
| **Plus (children)** | `model+` | Select model and all downstream children | `dbt run -s my_model+` |
| **Plus (parents)** | `+model` | Select model and all upstream parents | `dbt run -s +my_model` |
| **Plus (both)** | `+model+` | Select model, all parents, and all children | `dbt run -s +my_model+` |
| **N-plus (children)** | `model+n` | Select model and n degrees of children | `dbt run -s my_model+1` |
| **N-plus (parents)** | `n+model` | Select model and n degrees of parents | `dbt run -s 2+my_model` |
| **N-plus (both)** | `n+model+n` | Select model with n parents and n children | `dbt run -s 3+my_model+4` |
| **At (@)** | `@model` | Select model, its children, and parents of children | `dbt run -s @my_model` |
| **Star (*)** | `dir.*` | Select all models in a directory | `dbt run -s finance.base.*` |

---

## Selection Methods

| Method | Syntax | Description | Example |
|--------|--------|-------------|---------|
| `tag` | `tag:<name>` | Models with a specific tag | `dbt run -s "tag:nightly"` |
| `source` | `source:<name>+` | Source and its downstream models | `dbt run -s "source:snowplow+"` |
| `resource_type` | `resource_type:<type>` | All resources of a type | `dbt list -s "resource_type:test"` |
| `path` | `path:<dir>` | Models in a directory path | `dbt run -s "models/staging/github"` |
| `package` | `package:<name>` | Models from a package | `dbt run -s "package:snowplow"` |
| `config` | `config.<key>:<value>` | Models with a specific config value | `dbt run -s "config.materialized:incremental"` |
| `test_type` | `test_type:<generic\|singular>` | Generic or singular tests | `dbt test -s "test_type:generic"` |
| `test_name` | `test_name:<name>` | Tests of a specific type | `dbt test -s "test_name:unique"` |
| `state` | `state:<modified\|new\|old>` | Models changed since last state | `dbt run -s "state:modified" --state path/to/artifacts` |
| `exposure` | `exposure:<name>` | Models upstream of an exposure | `dbt run -s "+exposure:weekly_kpis"` |
| `metric` | `metric:<name>` | Models upstream of a metric | `dbt build -s "+metric:weekly_active_users"` |
| `result` | `result:<status>` | Models with a result status from last run | `dbt run -s "result:error" --state path/to/artifacts` |
| `source_status` | `source_status:<fresher\|stale>` | Sources by freshness status | `dbt build -s "source_status:fresher+"` |
| `group` | `group:<name>` | Models in a group | `dbt run -s "group:finance"` |
| `access` | `access:<public\|private\|protected>` | Models by access level | `dbt list -s "access:public"` |
| `version` | `version:<latest\|prerelease>` | Versioned models | `dbt list -s "version:latest"` |
| `unit_test` | `unit_test:<name>` | Specific unit tests | `dbt list -s "unit_test:*"` |

---

## Set Operators

### Union — run both sets (space-separated)
```bash
# Run snowplow_sessions + all its ancestors, AND fct_orders + all its ancestors
dbt run --select +snowplow_sessions +fct_orders
```

### Intersection — only models in BOTH sets (comma-separated)
```bash
# Only models that are ancestors of BOTH snowplow_sessions AND fct_orders
dbt run --select +snowplow_sessions,+fct_orders

# Models in marts/finance subdirectory AND tagged nightly
dbt run --select marts.finance,tag:nightly
```

### Exclude
```bash
# Run all models in my_package, except a_big_model and its children
dbt run --select my_package.*+ --exclude my_package.a_big_model+
```

---

## State & Defer

```bash
# State: compare against a previous manifest to find modified models
dbt run --select "state:modified+" --state path/to/artifacts

# Defer: build only changed models, use prod for upstream dependencies
# (commonly used for Slim CI to avoid rebuilding the whole project)
dbt build --select "state:modified+" --defer --state path/to/artifacts

# Common CI pattern: run + test only what changed
dbt build -s "state:modified+" --defer --state ./prod-artifacts
```

---

## dbt docs

```bash
# Generate the docs site (compiles project first by default)
dbt docs generate

# Generate without recompiling
dbt docs generate --no-compile

# Serve docs locally (default port 8080)
dbt docs serve

# Serve on a custom port
dbt docs serve --port 9090
```

