---
tags: [cheat-sheet, snowflake, sql, data-warehouse]
---

# Snowflake Hidden Gems — Functions & Features Most Users Don't Know About

## Contents

- [[#Common Workflows]]
- [[#Data Transformation]]
- [[#Text & Pattern Matching]]
- [[#Aggregation & Grouping]]
- [[#Window Functions]]
- [[#Date & Time]]
- [[#Semi-Structured & Nested Data]]
- [[#Data Quality & Debugging]]
- [[#Security & Governance]]
- [[#Sequences, Random Data & Test Generation]]
- [[#Performance & Monitoring]]
- [[#Cortex AI / ML]]
- [[#Advanced SQL Patterns]]
- [[#Top "I Wish I Knew This Sooner" Picks]]

---

## Common Workflows

### Deduplicate rows — keep latest per key
```sql
-- Keep the most recent record per loan_id; QUALIFY avoids a subquery
SELECT * EXCEPT (rn)
FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY loan_id ORDER BY updated_at DESC) AS rn
    FROM loans
)
QUALIFY rn = 1
```

### Flatten a JSON array into rows
```sql
SELECT
    l.loan_id,
    p.value:payment_date::DATE   AS payment_date,
    p.value:amount::NUMBER(18,2) AS amount
FROM loans l,
LATERAL FLATTEN(input => l.payments_json) p
```

### Pivot long to wide — one column per loan status
```sql
PIVOT loans
    ON loan_status
    USING sum(balance)
    GROUP BY originator
```

### Time travel — query a table as it was N minutes ago
```sql
SELECT * FROM loans AT (OFFSET => -1800)  -- 1800 seconds ago

-- Or by exact timestamp:
SELECT * FROM loans AT (TIMESTAMP => '2024-01-15 09:00:00'::TIMESTAMP_NTZ)
```

### Safe type conversion — don't let bad data break the pipeline
```sql
SELECT
    loan_id,
    TRY_CAST(raw_balance AS NUMBER(18,2))           AS balance,
    ZEROIFNULL(TRY_CAST(raw_balance AS NUMBER(18,2))) AS balance_or_zero
FROM raw_loans
```

### Fingerprint a table to detect unexpected drift
```sql
-- Run before and after a load; matching hashes means no data changed
SELECT HASH_AGG(*) AS fingerprint, COUNT(*) AS row_count FROM loans
```

---

## Data Transformation

| Function | Description |
|----------|-------------|
| `SELECT * EXCLUDE (col1, col2)` | Select all columns except the listed ones -- avoids rewriting 95 column names |
| `SELECT * RENAME (col AS alias)` | Rename one or more columns within a `SELECT *` without listing every other column |
| `SELECT * REPLACE (expr AS col)` | Override the value of a column within `SELECT *` (e.g. `UPPER(name) AS name`) without listing every column |
| `SELECT * ILIKE '%pattern%'` | Filter which columns are returned by matching their names against a pattern -- case-insensitive |
| `FLATTEN()` | Explodes semi-structured data (JSON/arrays) into rows |
| `OBJECT_CONSTRUCT()` | Build JSON objects from relational data |
| `ARRAY_AGG()` | Aggregate values into an array |
| `TRY_CAST()` / `TRY_TO_NUMBER()` | Safe casting -- returns NULL instead of errors |
| `ROUND(val, scale, 'HALF_TO_EVEN')` | Banker's rounding -- rounds 2.5 to 2 and 3.5 to 4 (nearest even), use `'HALF_AWAY_FROM_ZERO'` for standard rounding |
| `NULLIFZERO()` | Returns NULL if the value is zero (great for avoiding divide-by-zero) |
| `ZEROIFNULL()` | Returns 0 if the value is NULL (clean numeric defaults) |
| `**array_col` | Spread operator -- expands an array column into individual scalar values inline within a SELECT |

---

## Text & Pattern Matching

| Function | Description |
|----------|-------------|
| `REGEXP_SUBSTR_ALL()` | Extract all regex matches as an array |
| `REGEXP_LIKE(col, pattern)` | Boolean pattern match -- useful for data validation (emails, phone numbers) |
| `REGEXP_REPLACE(col, pattern, replacement)` | Find-and-replace using a regex pattern |
| `EDITDISTANCE()` | Levenshtein distance for fuzzy string matching |
| `JAROWINKLER_SIMILARITY()` | Similarity scoring for deduplication |
| `SPLIT_PART()` | Extract Nth element from a delimited string |
| `STRTOK(str, delim, n)` | Extract the Nth token from a delimited string (`STRTOK(email, '@', 1)` for username) |
| `STRTOK_TO_ARRAY()` | Split string into an array |
| `LIKE ANY` | Match against multiple patterns in one clause (`WHERE name LIKE ANY ('%snow%', '%flake%')`) |
| `INITCAP()` | Title-case strings (locale-aware) |
| `PARSE_JSON()` / `PARSE_XML()` | Convert strings to structured types inline |
| `SEARCH(cols, 'token1 token2', 'AND')` | Full-text search requiring all tokens to match -- third argument controls AND vs OR token logic |

---

## Aggregation & Grouping

| Function | Description |
|----------|-------------|
| `GROUPING SETS` / `ROLLUP` / `CUBE` | Multiple grouping levels in one pass |
| `LISTAGG()` | Concatenate values into a delimited string |
| `GROUP BY ALL` | Group by all non-aggregated columns automatically -- no need to repeat them |
| `MIN_BY(value_col, order_col)` / `MAX_BY(...)` | Return the value of one column at the row where another column is min/max -- avoids self-joins and subqueries |
| `ORDER BY ALL` | Sort by all selected columns without listing each one |
| `APPROX_COUNT_DISTINCT()` | HyperLogLog estimate, way faster on huge tables |
| `MINHASH()` / `APPROXIMATE_JACCARD_INDEX()` | Approximate set similarity at scale |
| `BITAND_AGG()` / `BITOR_AGG()` | Bitwise aggregations for flag columns |
| `HASH_AGG()` | Single hash fingerprint of a table/result for drift detection |

---

## Window Functions

| Function | Description |
|----------|-------------|
| `QUALIFY` | Filter window function results directly (no subquery needed) |
| `CONDITIONAL_CHANGE_EVENT()` | Detects value changes in ordered data |
| `NTH_VALUE()` | Get the Nth row in a window (not just first/last) |
| `RATIO_TO_REPORT()` | Percentage of total without a self-join |
| `CUME_DIST()` / `PERCENT_RANK()` | Percentile positioning |
| `IGNORE NULLS` on `LAG()`/`LEAD()` | Skip NULLs when looking forward/back |
| `WIDTH_BUCKET()` | Histogram binning in one function |

---

## Date & Time

| Function | Description |
|----------|-------------|
| `TIME_SLICE()` | Snap timestamps to uniform intervals (5-min buckets, etc.) |
| `LAST_DAY()` | End of month/week/year without manual calculation |
| `DATEDIFF()` with `WEEK`, `QUARTER` | Often forgotten granularities |
| `RESAMPLE(ts, 'DAY', 1) FILL (METHOD => 'BFILL')` | Time-series gap filling with interpolation -- pads missing intervals using bfill, ffill, or a constant |

---

## Semi-Structured & Nested Data

| Function | Description |
|----------|-------------|
| `LATERAL FLATTEN` + `RECURSIVE => TRUE` | Recursively explode nested JSON |
| `STRIP_NULL_VALUE()` | Remove JSON nulls that otherwise become the string `"null"` |
| `GET_PATH()` / `XMLGET()` | Navigate nested structures with dot-notation strings |
| `OBJECT_KEYS()` | List all keys in a VARIANT object |
| `ARRAY_INTERSECTION()` / `ARRAY_EXCEPT()` | Set operations on arrays |
| `ARRAY_COMPACT()` | Remove NULLs from arrays |
| `ARRAY_SORT(arr, asc)` | Sort an array ascending or descending |
| `ARRAY_MIN(arr)` / `ARRAY_MAX(arr)` | Min/max value of an array, NULL-safe |
| `AS_*()` family (`AS_INTEGER`, `AS_VARCHAR`) | Type-safe VARIANT extraction |

---

## Data Quality & Debugging

| Function | Description |
|----------|-------------|
| `SYSTEM$CLUSTERING_INFORMATION()` | Check if your table needs re-clustering |
| `GET_DDL()` | Reverse-engineer DDL for any object |
| `RESULT_SCAN(LAST_QUERY_ID())` | Query results of your previous query as a table |
| `SYSTEM$EXPLAIN_PLAN_JSON()` | Get query plan without running it |
| `SYSTEM$TYPEOF()` | Inspect runtime data types |
| `EQUAL_NULL()` | NULL-safe equality (treats NULL = NULL as TRUE) |

---

## Security & Governance

| Function | Description |
|----------|-------------|
| `SYSTEM$GET_TAG()` | Read tag values on objects |
| `POLICY_REFERENCES()` | Find which policies apply to what |
| `IS_ROLE_IN_SESSION()` | Check active roles without context functions |

---

## Sequences, Random Data & Test Generation

| Function | Description |
|----------|-------------|
| `GENERATOR()` | Create synthetic rows (`TABLE(GENERATOR(ROWCOUNT => 1000))`) |
| `SEQ1()` ... `SEQ8()` | Sequence generators inside `GENERATOR()` |
| `UNIFORM()` / `RANDOM()` | Random number generation |
| `ZIPF()` / `NORMAL()` | Statistical distributions for test data |
| `UUID_STRING()` | Generate UUIDs natively |

---

## Performance & Monitoring

| Function | Description |
|----------|-------------|
| `SAMPLE` / `TABLESAMPLE` | Random row sampling without scanning full table |
| `CHANGES` clause | Query change history (streams alternative for ad-hoc use) |
| `AT` / `BEFORE` | Time Travel queries (`SELECT * FROM t AT(OFFSET => -300)`) |
| `SYSTEM$PIPE_STATUS()` | Monitor Snowpipe health |
| `JOIN DIRECTED` | Explicit join order hint -- tells the optimizer which table to use as the build side, overriding its default plan |

---

## Cortex AI / ML

| Function | Description |
|----------|-------------|
| `SNOWFLAKE.CORTEX.COMPLETE()` | LLM inference directly in SQL (text gen, sentiment, translation, classification, entity extraction) |
| `SNOWFLAKE.CORTEX.FINETUNE()` | Fine-tune a base LLM on your own data (`'CREATE'` / `'SHOW'` / `'DESCRIBE'` actions) |
| `SNOWFLAKE.CORTEX.SENTIMENT()` | Sentiment analysis |
| `SNOWFLAKE.CORTEX.SUMMARIZE()` | Text summarization |
| `SNOWFLAKE.CORTEX.TRANSLATE()` | Language translation |
| `SNOWFLAKE.CORTEX.EMBED_TEXT_768()` | Generate embeddings for vector search |
| `SNOWFLAKE.CORTEX.EXTRACT_ANSWER()` | Extract answers from documents |
| `SNOWFLAKE.CORTEX.CLASSIFY_TEXT()` | Zero-shot text classification |
| `MODEL_PREDICT()` | Call ML models from the Model Registry in SQL |
| `AI_FILTER(col, 'prompt')` / `AI_CLASSIFY(col, labels)` / `AI_AGG(col, 'prompt')` | Cortex AI SQL operators for row-level filtering, classification, and aggregation using natural language prompts |

---

## Advanced SQL Patterns

| Function | Description |
|----------|-------------|
| `MATCH_RECOGNIZE` | Pattern matching over ordered rows (sessionization, funnel analysis) |
| `CONNECT BY` | Recursive/hierarchical queries (alternative to recursive CTEs) |
| `PIVOT` / `UNPIVOT` | Native row/column transformations |
| `EXECUTE IMMEDIATE` | Dynamic SQL inside stored procedures/scripts |
| `TO_QUERY(sql_string)` | Turn a dynamically constructed SQL string into a table reference -- useful for variable-driven filtering and aggregation |
| `TABLE tablename` | Shorthand for `SELECT * FROM tablename` |
| `ALTER TABLE ... ADD COLUMN IF NOT EXISTS` | Add a column only if it doesn't already exist |
| `ALTER TABLE ... DROP COLUMN IF EXISTS` | Drop a column only if it exists -- safe for scripted schema changes |
| `ASOF JOIN` | Join on the nearest timestamp instead of exact match (`MATCH_CONDITION(a.ts >= b.ts)`) |
| `MERGE ... UPDATE ALL BY NAME` | Update all matching columns by name automatically -- no manual column mapping needed |
| `UNION BY NAME` | Merge result sets by matching column names instead of position -- tolerates different column orderings across queries |
| `query1 ->> query2` | Chain query output directly into the next query as its input -- inline alternative to CTEs or subqueries |

---

## Top "I Wish I Knew This Sooner" Picks

1. **`QUALIFY`** -- eliminates subqueries for window function filtering
2. **`MATCH_RECOGNIZE`** -- replaces complex sessionization logic
3. **`CHANGES`** -- ad-hoc change tracking without setting up streams
4. **`TRY_CAST()`** -- no more failed pipelines from bad data
5. **`NULLIFZERO()` / `ZEROIFNULL()`** -- tiny functions that prevent so many bugs
6. **`GENERATOR()` + `SEQ()`** -- instant test data without external tools
