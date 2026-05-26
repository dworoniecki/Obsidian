# Snowflake Hidden Gems — Functions & Features Most Users Don't Know About

---

## Data Transformation

| Function | Description |
|----------|-------------|
| `FLATTEN()` | Explodes semi-structured data (JSON/arrays) into rows |
| `OBJECT_CONSTRUCT()` | Build JSON objects from relational data |
| `ARRAY_AGG()` | Aggregate values into an array |
| `TRY_CAST()` / `TRY_TO_NUMBER()` | Safe casting -- returns NULL instead of errors |
| `NULLIFZERO()` | Returns NULL if the value is zero (great for avoiding divide-by-zero) |
| `ZEROIFNULL()` | Returns 0 if the value is NULL (clean numeric defaults) |

---

## Text & Pattern Matching

| Function | Description |
|----------|-------------|
| `REGEXP_SUBSTR_ALL()` | Extract all regex matches as an array |
| `EDITDISTANCE()` | Levenshtein distance for fuzzy string matching |
| `JAROWINKLER_SIMILARITY()` | Similarity scoring for deduplication |
| `SPLIT_PART()` | Extract Nth element from a delimited string |
| `STRTOK_TO_ARRAY()` | Split string into an array |
| `INITCAP()` | Title-case strings (locale-aware) |
| `PARSE_JSON()` / `PARSE_XML()` | Convert strings to structured types inline |

---

## Aggregation & Grouping

| Function | Description |
|----------|-------------|
| `GROUPING SETS` / `ROLLUP` / `CUBE` | Multiple grouping levels in one pass |
| `LISTAGG()` | Concatenate values into a delimited string |
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

---

## Cortex AI / ML

| Function | Description |
|----------|-------------|
| `SNOWFLAKE.CORTEX.COMPLETE()` | LLM inference directly in SQL |
| `SNOWFLAKE.CORTEX.SENTIMENT()` | Sentiment analysis |
| `SNOWFLAKE.CORTEX.SUMMARIZE()` | Text summarization |
| `SNOWFLAKE.CORTEX.TRANSLATE()` | Language translation |
| `SNOWFLAKE.CORTEX.EMBED_TEXT_768()` | Generate embeddings for vector search |
| `SNOWFLAKE.CORTEX.EXTRACT_ANSWER()` | Extract answers from documents |
| `SNOWFLAKE.CORTEX.CLASSIFY_TEXT()` | Zero-shot text classification |
| `MODEL_PREDICT()` | Call ML models from the Model Registry in SQL |

---

## Advanced SQL Patterns

| Function | Description |
|----------|-------------|
| `MATCH_RECOGNIZE` | Pattern matching over ordered rows (sessionization, funnel analysis) |
| `CONNECT BY` | Recursive/hierarchical queries (alternative to recursive CTEs) |
| `PIVOT` / `UNPIVOT` | Native row/column transformations |
| `EXECUTE IMMEDIATE` | Dynamic SQL inside stored procedures/scripts |

---

## Top "I Wish I Knew This Sooner" Picks

1. **`QUALIFY`** -- eliminates subqueries for window function filtering
2. **`MATCH_RECOGNIZE`** -- replaces complex sessionization logic
3. **`CHANGES`** -- ad-hoc change tracking without setting up streams
4. **`TRY_CAST()`** -- no more failed pipelines from bad data
5. **`NULLIFZERO()` / `ZEROIFNULL()`** -- tiny functions that prevent so many bugs
6. **`GENERATOR()` + `SEQ()`** -- instant test data without external tools
