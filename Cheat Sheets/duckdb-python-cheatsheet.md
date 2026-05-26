# DuckDB Python Cheatsheet

A practical guide for using DuckDB in Python, with examples using lending/financial data scenarios.

---

## Common Workflows

### Method Chaining
```python
# Read, filter, aggregate, sort — all chained
df = (
    duckdb.sql("""
        SELECT
            originator,
            count(*)       AS loan_count,
            sum(balance)   AS total_balance,
            avg(balance)   AS avg_balance
        FROM read_parquet('data/loans_*.parquet')
        WHERE loan_status IN ('CURRENT', 'PAID_OFF')
        GROUP BY originator
        ORDER BY total_balance DESC
        LIMIT 10
    """)
    .df()
)
```

### Reusable Connection with CTEs
```python
con = duckdb.connect('analytics.duckdb')

# Create base table once
con.execute("CREATE TABLE IF NOT EXISTS loans AS SELECT * FROM 'loans.parquet'")

# Run analysis with CTEs
df = con.execute("""
    WITH current_loans AS (
        SELECT * FROM loans WHERE loan_status = 'CURRENT'
    ),
    originator_stats AS (
        SELECT
            originator,
            count(*)   AS loan_count,
            sum(balance) AS total_balance
        FROM current_loans
        GROUP BY originator
    )
    SELECT
        *,
        total_balance / sum(total_balance) OVER () AS pct_of_total
    FROM originator_stats
    ORDER BY total_balance DESC
""").fetchdf()
```

### Querying Pandas/Polars In-Place
```python
# Load with Polars, query with DuckDB, return to Polars
loans = pl.read_parquet('loans.parquet')

result = duckdb.sql("""
    SELECT originator, sum(balance) AS total
    FROM loans
    GROUP BY originator
""").pl()
```

### Iterating Over Large Results
```python
con.execute("SELECT * FROM large_loans_table")

# Fetch in chunks instead of all at once
chunk_size = 10_000
while True:
    chunk = con.fetchmany(chunk_size)
    if not chunk:
        break
    # process chunk...
```

### Attach Multiple Databases
```python
con = duckdb.connect('main.duckdb')
con.execute("ATTACH 'archive.duckdb' AS archive")
con.execute("ATTACH 'staging.duckdb' AS staging (READ_ONLY)")

# Query across databases
con.execute("""
    SELECT * FROM main.loans
    UNION ALL
    SELECT * FROM archive.loans
""")

# Detach when done
con.execute("DETACH archive")
```

### Performance Settings
```python
# Set number of threads (default: all cores)
duckdb.sql("SET threads = 4")

# Set memory limit
duckdb.sql("SET memory_limit = '8GB'")

# Enable progress bar for long queries
duckdb.sql("SET enable_progress_bar = true")

# Temp directory for spill-to-disk (for queries exceeding memory_limit)
duckdb.sql("SET temp_directory = '/tmp/duckdb_temp'")

# Or set on connection
con = duckdb.connect(config={
    'threads': 4,
    'memory_limit': '8GB'
})
```

---

## Setup & Imports

```python
import duckdb
import pandas as pd
import polars as pl

# In-memory database (temporary, lost on close)
con = duckdb.connect()

# File-based database (persistent)
con = duckdb.connect('mydb.duckdb')

# Read-only connection
con = duckdb.connect('mydb.duckdb', read_only=True)

# Default connection shorthand (no explicit connect needed)
duckdb.sql("SELECT 1")  # Uses built-in default in-memory connection
```

---

## Fetching Results

| Method | Returns | Example | Use Case |
|--------|---------|---------|----------|
| `.fetchall()` | List of tuples | `con.execute("SELECT ...").fetchall()` | Raw results, lightweight |
| `.fetchone()` | Single tuple | `con.execute("SELECT ...").fetchone()` | Scalar or single row |
| `.fetchdf()` | Pandas DataFrame | `con.execute("SELECT ...").fetchdf()` | Pandas workflows |
| `.df()` | Pandas DataFrame (alias) | `duckdb.sql("SELECT ...").df()` | Shorthand for fetchdf |
| `.pl()` | Polars DataFrame | `duckdb.sql("SELECT ...").pl()` | Polars workflows |
| `.fetchnumpy()` | Dict of NumPy arrays | `con.execute("SELECT ...").fetchnumpy()` | Numerical computation |
| `.description` | Column metadata | `con.execute("SELECT ...").description` | Get column names/types |

```python
# All fetch styles on the same query
result = duckdb.sql("SELECT loan_id, balance FROM loans WHERE balance > 10000")

result.fetchall()    # [('L001', 15000.0), ('L002', 25000.0), ...]
result.fetchdf()     # Pandas DataFrame
result.pl()          # Polars DataFrame

# Get column names from description
con.execute("SELECT loan_id, balance FROM loans")
cols = [desc[0] for desc in con.description]  # ['loan_id', 'balance']
```

---

## Reading Data

### CSV Files
```python
# Auto-detect from file extension
duckdb.sql("SELECT * FROM 'loans.csv'")

# Explicit read_csv with options
duckdb.sql("SELECT * FROM read_csv('loans.csv')")
duckdb.sql("SELECT * FROM read_csv('loans.csv', header=true)")
duckdb.sql("SELECT * FROM read_csv('loans.csv', delim='|')")            # Pipe-delimited
duckdb.sql("SELECT * FROM read_csv('loans.csv', skip=2)")               # Skip first 2 rows
duckdb.sql("SELECT * FROM read_csv('loans.csv', nullstr='N/A')")        # Treat N/A as NULL
duckdb.sql("SELECT * FROM read_csv('loans.csv', dateformat='%m/%d/%Y')") # Custom date format

# Specify column types
duckdb.sql("""
    SELECT * FROM read_csv('loans.csv',
        columns={'loan_id': 'VARCHAR', 'balance': 'DOUBLE', 'zip_code': 'VARCHAR'}
    )
""")

# Python API (returns relation)
rel = duckdb.read_csv('loans.csv')
df = rel.df()
```

### Parquet Files
```python
# Auto-detect from file extension
duckdb.sql("SELECT * FROM 'loans.parquet'")

# Explicit read_parquet
duckdb.sql("SELECT * FROM read_parquet('loans.parquet')")

# Select specific columns (more efficient - only reads needed columns)
duckdb.sql("SELECT loan_id, balance FROM read_parquet('loans.parquet')")

# Python API
rel = duckdb.read_parquet('loans.parquet')
```

### JSON Files
```python
# Auto-detect from file extension
duckdb.sql("SELECT * FROM 'data.json'")

# read_json with options
duckdb.sql("SELECT * FROM read_json('data.json')")
duckdb.sql("SELECT * FROM read_json('data.json', format='array')")       # JSON array at root
duckdb.sql("SELECT * FROM read_json('data.json', format='newline_delimited')") # NDJSON/JSONL
```

### Multiple Files (Glob Patterns)
```python
# Glob pattern - reads all matching files as one table
duckdb.sql("SELECT * FROM 'data/loans_*.parquet'")
duckdb.sql("SELECT * FROM read_parquet('data/*.parquet')")

# List of specific files
duckdb.sql("SELECT * FROM read_parquet(['jan.parquet', 'feb.parquet', 'mar.parquet'])")

# Add filename column to know which file each row came from
duckdb.sql("SELECT *, filename FROM read_parquet('data/*.parquet', filename=true)")
```

### From Python Objects
```python
# DuckDB can query pandas/polars DataFrames directly by variable name
pandas_df = pd.read_csv('loans.csv')
polars_df = pl.read_csv('loans.csv')

duckdb.sql("SELECT * FROM pandas_df WHERE balance > 10000")   # Reference by variable name
duckdb.sql("SELECT * FROM polars_df WHERE balance > 10000")

# Explicitly register with a custom name
con.register('loans', pandas_df)
con.register('loans_polars', polars_df)
con.execute("SELECT * FROM loans")
```

### Remote Files (S3 / HTTP)
```python
# Install and load httpfs extension (one-time setup)
duckdb.sql("INSTALL httpfs")
duckdb.sql("LOAD httpfs")

# S3
duckdb.sql("SET s3_region='us-east-1'")
duckdb.sql("SET s3_access_key_id='AKIAIOSFODNN7EXAMPLE'")
duckdb.sql("SET s3_secret_access_key='wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY'")

duckdb.sql("SELECT * FROM 's3://my-bucket/loans.parquet'")
duckdb.sql("SELECT * FROM 's3://my-bucket/data/*.parquet'")

# HTTP
duckdb.sql("SELECT * FROM 'https://example.com/loans.parquet'")
```

---

## Writing Data

```python
# Write to Parquet
duckdb.sql("COPY (SELECT * FROM loans) TO 'output.parquet' (FORMAT PARQUET)")
duckdb.sql("COPY (SELECT * FROM loans) TO 'output.parquet' (FORMAT PARQUET, COMPRESSION ZSTD)")

# Write to CSV
duckdb.sql("COPY (SELECT * FROM loans) TO 'output.csv' (FORMAT CSV, HEADER)")
duckdb.sql("COPY (SELECT * FROM loans) TO 'output.csv' (FORMAT CSV, HEADER, DELIMITER '|')")

# Write to JSON
duckdb.sql("COPY (SELECT * FROM loans) TO 'output.json' (FORMAT JSON)")

# Write to multiple partitioned Parquet files
duckdb.sql("""
    COPY (SELECT * FROM loans) TO 'output/'
    (FORMAT PARQUET, PARTITION_BY (originator, loan_status))
""")

# Export entire database
duckdb.sql("EXPORT DATABASE 'backup/' (FORMAT PARQUET)")

# To Python objects
pandas_df = duckdb.sql("SELECT * FROM loans").df()
polars_df = duckdb.sql("SELECT * FROM loans").pl()
```

---

## Creating and Managing Tables

```python
# Create table from query
con.execute("CREATE TABLE loans AS SELECT * FROM 'loans.csv'")
con.execute("CREATE TABLE loans AS SELECT * FROM read_parquet('loans.parquet')")

# Create empty table with schema
con.execute("""
    CREATE TABLE loans (
        loan_id     VARCHAR,
        balance     DOUBLE,
        originator  VARCHAR,
        loan_status VARCHAR,
        orig_date   DATE
    )
""")

# Create if not exists
con.execute("CREATE TABLE IF NOT EXISTS loans AS SELECT * FROM 'loans.csv'")

# Create or replace
con.execute("CREATE OR REPLACE TABLE loans AS SELECT * FROM 'loans.csv'")

# Insert data
con.execute("INSERT INTO loans SELECT * FROM 'new_loans.csv'")
con.execute("INSERT INTO loans VALUES ('L999', 5000.0, 'ACME', 'CURRENT', '2024-01-01')")

# Drop table
con.execute("DROP TABLE loans")
con.execute("DROP TABLE IF EXISTS loans")

# List all tables
duckdb.sql("SHOW TABLES").fetchall()
duckdb.sql("SHOW ALL TABLES").fetchall()  # Includes attached databases
```

---

## Parameterized Queries

```python
# Positional parameters with ?
con.execute("SELECT * FROM loans WHERE balance > ?", [10000])
con.execute("SELECT * FROM loans WHERE originator = ? AND balance > ?", ['ACME', 5000])

# Named parameters with $name
con.execute(
    "SELECT * FROM loans WHERE originator = $orig AND balance > $min_bal",
    {"orig": "ACME", "min_bal": 5000}
)

# With fetchdf
df = con.execute("SELECT * FROM loans WHERE loan_status = ?", ['CURRENT']).fetchdf()
```

---

## Inspecting Data

```python
# DESCRIBE - show schema/column types
duckdb.sql("DESCRIBE loans")
duckdb.sql("DESCRIBE SELECT * FROM 'loans.parquet'")

# SUMMARIZE - descriptive stats for all columns
duckdb.sql("SUMMARIZE loans")
duckdb.sql("SUMMARIZE SELECT * FROM 'loans.parquet'")

# Show first rows
duckdb.sql("SELECT * FROM loans LIMIT 10")

# Count rows
duckdb.sql("SELECT count(*) FROM loans").fetchone()[0]

# Check column names and types
duckdb.sql("DESCRIBE loans").fetchdf()
```

---

## DuckDB-Specific SQL Extensions

### EXCLUDE and REPLACE

```python
# Select all columns except specific ones
duckdb.sql("SELECT * EXCLUDE (internal_id, created_at) FROM loans")

# Select all columns but override specific ones
duckdb.sql("SELECT * REPLACE (balance / 100 AS balance) FROM loans")

# Combine both
duckdb.sql("SELECT * EXCLUDE (raw_date) REPLACE (balance / 100 AS balance) FROM loans")
```

### COLUMNS() - Pattern-Based Column Selection

```python
# Select columns matching a regex pattern
duckdb.sql("SELECT COLUMNS('balance_.*') FROM loans")           # All balance_ columns
duckdb.sql("SELECT COLUMNS('.*_date') FROM loans")              # All _date columns

# Apply function to all matching columns
duckdb.sql("SELECT sum(COLUMNS('balance_.*')) FROM loans")       # Sum all balance columns

# Apply to all numeric columns using column types
duckdb.sql("SELECT COLUMNS(c -> c LIKE '%balance%') FROM loans") # Lambda filter on name
```

### PIVOT and UNPIVOT

```python
# PIVOT - long to wide
duckdb.sql("""
    PIVOT loans
    ON loan_status
    USING sum(balance)
    GROUP BY originator
""")

# UNPIVOT - wide to long (like melt in pandas)
# Wide: loan_id | jan_balance | feb_balance | mar_balance
duckdb.sql("""
    UNPIVOT loans
    ON jan_balance, feb_balance, mar_balance
    INTO
        NAME  month
        VALUE balance
""")
```

---

## Aggregations

```python
# Standard aggregations
duckdb.sql("""
    SELECT
        originator,
        count(*)                     AS loan_count,
        sum(balance)                 AS total_balance,
        avg(balance)                 AS avg_balance,
        min(balance)                 AS min_balance,
        max(balance)                 AS max_balance,
        stddev(balance)              AS std_balance,
        median(balance)              AS median_balance,
        percentile_cont(0.75)
            WITHIN GROUP (ORDER BY balance) AS p75_balance
    FROM loans
    GROUP BY originator
""")

# FILTER clause on aggregations
duckdb.sql("""
    SELECT
        count(*) FILTER (WHERE loan_status = 'CURRENT')   AS current_count,
        count(*) FILTER (WHERE loan_status = 'DEFAULTED') AS default_count,
        sum(balance) FILTER (WHERE loan_status = 'CURRENT') AS current_balance
    FROM loans
""")

# List aggregation - collect values into a list
duckdb.sql("SELECT originator, list(loan_id) AS loan_ids FROM loans GROUP BY originator")

# String aggregation
duckdb.sql("SELECT originator, string_agg(loan_id, ', ') AS loan_list FROM loans GROUP BY originator")
duckdb.sql("SELECT originator, string_agg(loan_id, ', ' ORDER BY balance DESC) FROM loans GROUP BY originator")
```

---

## Window Functions

```python
duckdb.sql("""
    SELECT
        loan_id,
        originator,
        balance,

        -- Ranking
        row_number()  OVER (PARTITION BY originator ORDER BY balance DESC) AS row_num,
        rank()        OVER (PARTITION BY originator ORDER BY balance DESC) AS rank,
        dense_rank()  OVER (PARTITION BY originator ORDER BY balance DESC) AS dense_rank,
        percent_rank() OVER (PARTITION BY originator ORDER BY balance DESC) AS pct_rank,
        ntile(4)      OVER (PARTITION BY originator ORDER BY balance DESC) AS quartile,

        -- Offset
        lag(balance, 1)  OVER (PARTITION BY loan_id ORDER BY payment_date) AS prev_balance,
        lead(balance, 1) OVER (PARTITION BY loan_id ORDER BY payment_date) AS next_balance,
        first_value(balance) OVER (PARTITION BY originator ORDER BY orig_date) AS first_balance,
        last_value(balance)  OVER (PARTITION BY originator ORDER BY orig_date) AS last_balance,

        -- Aggregation over window (broadcast to all rows in partition)
        sum(balance)   OVER (PARTITION BY originator) AS originator_total,
        avg(balance)   OVER (PARTITION BY originator) AS originator_avg,
        count(*)       OVER (PARTITION BY originator) AS originator_count,

        -- Cumulative
        sum(balance)   OVER (PARTITION BY originator ORDER BY orig_date
                             ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_balance,

        -- Rolling 3-row window
        avg(balance)   OVER (ORDER BY payment_date
                             ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS rolling_avg_3

    FROM loans
""")
```

---

## Joins

```python
# Standard joins
duckdb.sql("SELECT * FROM loans l LEFT JOIN payments p ON l.loan_id = p.loan_id")
duckdb.sql("SELECT * FROM loans l INNER JOIN originators o ON l.originator = o.name")
duckdb.sql("SELECT * FROM loans l FULL OUTER JOIN payments p ON l.loan_id = p.loan_id")

# Anti join - rows in loans NOT in payments
duckdb.sql("""
    SELECT * FROM loans l
    WHERE NOT EXISTS (SELECT 1 FROM payments p WHERE p.loan_id = l.loan_id)
""")

# ASOF join - match each row to the nearest row in another table (by time)
duckdb.sql("""
    SELECT * FROM loan_events
    ASOF JOIN interest_rates
    ON loan_events.loan_id = interest_rates.loan_id
    AND loan_events.event_date >= interest_rates.effective_date
""")

# LATERAL join - reference left-side columns in a subquery
duckdb.sql("""
    SELECT o.originator, top_loans.*
    FROM originators o,
    LATERAL (
        SELECT * FROM loans l
        WHERE l.originator = o.originator
        ORDER BY balance DESC
        LIMIT 3
    ) top_loans
""")
```

---

## Date & Time Functions

```python
duckdb.sql("""
    SELECT
        orig_date,

        -- Truncate to period
        date_trunc('month', orig_date)    AS month_start,
        date_trunc('quarter', orig_date)  AS quarter_start,
        date_trunc('year', orig_date)     AS year_start,

        -- Extract parts
        extract('year'  FROM orig_date)   AS year,
        extract('month' FROM orig_date)   AS month,
        extract('day'   FROM orig_date)   AS day,
        extract('dow'   FROM orig_date)   AS day_of_week,   -- 0=Sunday
        extract('doy'   FROM orig_date)   AS day_of_year,
        extract('quarter' FROM orig_date) AS quarter,

        -- Date differences
        date_diff('day',   orig_date, current_date) AS age_days,
        date_diff('month', orig_date, current_date) AS age_months,
        date_diff('year',  orig_date, current_date) AS age_years,

        -- Arithmetic
        orig_date + INTERVAL '30 days'  AS plus_30_days,
        orig_date + INTERVAL '3 months' AS plus_3_months,
        orig_date + INTERVAL '1 year'   AS plus_1_year,

        -- Formatting
        strftime(orig_date, '%Y-%m')         AS year_month,
        strftime(orig_date, '%Y-%m-%d')      AS date_str,
        strftime(orig_date, '%B %d, %Y')     AS long_date,

        -- Current date/time
        current_date                          AS today,
        current_timestamp                     AS now_ts,
        now()                                 AS now_fn

    FROM loans
""")

# Parse strings to dates
duckdb.sql("SELECT strptime('2024-01-15', '%Y-%m-%d') AS parsed_date")
duckdb.sql("SELECT '2024-01-15'::DATE AS parsed_date")
duckdb.sql("SELECT CAST('2024-01-15' AS DATE) AS parsed_date")
```

---

## String Functions

```python
duckdb.sql("""
    SELECT
        originator,

        -- Case
        lower(originator)       AS lower_name,
        upper(originator)       AS upper_name,

        -- Trimming
        trim(originator)        AS trimmed,
        ltrim(originator)       AS left_trimmed,
        rtrim(originator)       AS right_trimmed,

        -- Substring
        left(originator, 3)     AS first_3,
        right(originator, 3)    AS last_3,
        substr(originator, 2, 4) AS substr_2_4,  -- Start at pos 2, length 4

        -- Length
        length(originator)       AS char_length,

        -- Contains / matching
        contains(originator, 'ACME')               AS has_acme,
        originator LIKE '%ACME%'                   AS like_acme,
        originator ILIKE '%acme%'                  AS ilike_acme,       -- Case-insensitive
        regexp_matches(originator, '^[A-Z]+$')     AS all_caps,

        -- Replace
        replace(originator, '_', '-')              AS dashes,
        regexp_replace(originator, '[0-9]', 'N')   AS no_numbers,

        -- Extract
        regexp_extract(loan_id, '([A-Z]+)', 1)     AS alpha_prefix,

        -- Split
        string_split(loan_id, '-')                 AS parts,           -- Returns list
        string_split(loan_id, '-')[1]              AS first_part,      -- 1-indexed

        -- Concat
        originator || '-' || loan_status           AS combined,
        concat(originator, '-', loan_status)       AS combined2,
        concat_ws('-', originator, loan_status)    AS combined3,       -- With separator

        -- Pad
        lpad(loan_id, 10, '0')                     AS zero_padded,
        rpad(originator, 20, ' ')                  AS right_padded

    FROM loans
""")
```

---

## List / Array Functions

```python
# Create lists via aggregation
duckdb.sql("SELECT originator, list(loan_id) AS loan_ids FROM loans GROUP BY originator")
duckdb.sql("SELECT originator, list(balance ORDER BY balance DESC) AS balances FROM loans GROUP BY originator")

# Work with list columns
duckdb.sql("""
    SELECT
        loan_ids,
        len(loan_ids)                   AS count,
        loan_ids[1]                     AS first_id,           -- 1-indexed
        list_contains(loan_ids, 'L001') AS has_l001,
        list_sort(loan_ids)             AS sorted_ids,
        list_distinct(loan_ids)         AS unique_ids,
        list_reverse(loan_ids)          AS reversed_ids,

        -- Transform each element (lambda)
        list_transform(balances, x -> x / 100)     AS balances_dollars,

        -- Filter elements (lambda)
        list_filter(balances, x -> x > 10000)      AS large_balances,

        -- Reduce to scalar (lambda)
        list_reduce(balances, (acc, x) -> acc + x) AS manual_sum,

        -- Slice
        loan_ids[2:4]                   AS elements_2_to_4,

        -- Flatten nested lists
        flatten([[1,2], [3,4]])         AS flat_list

    FROM originator_groups
""")

# UNNEST - expand list into rows
duckdb.sql("""
    SELECT originator, unnest(loan_ids) AS loan_id
    FROM originator_groups
""")

# Generate a series
duckdb.sql("SELECT unnest(generate_series(1, 12)) AS month")
```

---

## Struct / Map Functions

```python
# Create structs
duckdb.sql("""
    SELECT
        {'id': loan_id, 'bal': balance, 'status': loan_status} AS loan_info
    FROM loans
""")

# Access struct fields
duckdb.sql("SELECT loan_info.id, loan_info.bal FROM loans_with_struct")
duckdb.sql("SELECT loan_info['id'] FROM loans_with_struct")

# struct_pack
duckdb.sql("SELECT struct_pack(id := loan_id, bal := balance) AS s FROM loans")

# MAP type
duckdb.sql("SELECT MAP {'a': 1, 'b': 2} AS my_map")
duckdb.sql("SELECT map_extract(my_map, 'a') AS value FROM map_table")
```

---

## JSON Functions

```python
# Read JSON
duckdb.sql("SELECT * FROM read_json('data.json')")
duckdb.sql("SELECT * FROM read_json('data.ndjson', format='newline_delimited')")

# Extract from JSON column
duckdb.sql("""
    SELECT
        json_extract(payload, '$.loan_id')         AS loan_id,        -- Returns JSON
        json_extract_string(payload, '$.loan_id')  AS loan_id_str,    -- Returns VARCHAR
        json_extract(payload, '$.payments[0]')     AS first_payment,

        -- Array operations
        json_array_length(payload, '$.payments')   AS payment_count,

        -- Type checks
        json_type(payload, '$.balance')            AS balance_type    -- 'NUMBER', 'STRING', etc.

    FROM raw_events
""")

# Convert to/from JSON
duckdb.sql("SELECT to_json({'loan_id': 'L001', 'balance': 5000})")
```

---

## Extensions

| Extension | Purpose | Install & Load |
|-----------|---------|----------------|
| `httpfs` | Read/write S3, HTTP, GCS | `INSTALL httpfs; LOAD httpfs` |
| `spatial` | Geospatial / GIS functions | `INSTALL spatial; LOAD spatial` |
| `json` | Enhanced JSON support | `INSTALL json; LOAD json` |
| `excel` | Read `.xlsx` files | `INSTALL excel; LOAD excel` |
| `sqlite` | Query SQLite databases | `INSTALL sqlite; LOAD sqlite` |
| `postgres` | Query PostgreSQL directly | `INSTALL postgres; LOAD postgres` |
| `iceberg` | Read Apache Iceberg tables | `INSTALL iceberg; LOAD iceberg` |
| `delta` | Read Delta Lake tables | `INSTALL delta; LOAD delta` |

```python
# Install and load any extension
duckdb.sql("INSTALL httpfs")
duckdb.sql("LOAD httpfs")

# Read Excel files (after installing excel extension)
duckdb.sql("SELECT * FROM read_xlsx('loans.xlsx')")

# Attach a SQLite database (after installing sqlite extension)
duckdb.sql("ATTACH 'legacy.sqlite' AS sqlite_db (TYPE SQLITE)")
duckdb.sql("SELECT * FROM sqlite_db.loans")
```

---

## Key Differences vs Standard SQL

| Feature | Standard SQL | DuckDB |
|---------|-------------|--------|
| **Select all except** | Not supported | `SELECT * EXCLUDE (col1, col2)` |
| **Override one column** | Not supported | `SELECT * REPLACE (col/100 AS col)` |
| **Column patterns** | Not supported | `SELECT COLUMNS('prefix_.*')` |
| **Quick stats** | Not supported | `SUMMARIZE table_name` |
| **Read file directly** | Not supported | `SELECT * FROM 'file.parquet'` |
| **Aggregate filter** | Not standard | `count(*) FILTER (WHERE status='A')` |
| **List agg** | `array_agg` (varies) | `list(col)` |
| **ASOF join** | Not supported | `ASOF JOIN ... ON a >= b` |
| **Lambda functions** | Not supported | `list_transform(lst, x -> x * 2)` |
| **PIVOT** | Not supported | `PIVOT tbl ON col USING agg(val)` |
