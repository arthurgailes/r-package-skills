# duckplyr Function Reference

Complete API reference for duckplyr - A DuckDB-backed version of dplyr for fast data manipulation.

## Data Frame Creation & Conversion

### `duckdb_tibble()`
**Create duckplyr data frame**

```r
duckdb_tibble(..., .prudence = c("lavish", "thrifty", "stingy"))
```

**Parameters:**
- `...`: Arguments forwarded to `tibble()`
- `.prudence`: Memory protection level (default: `"thrifty"`)
  - `"lavish"`: Converts regardless of size
  - `"thrifty"`: Max 1 million cells
  - `"stingy"`: Never converts intermediate results
  - Can also be named numeric vector with `cells` or `rows` limits

**Returns:** `duckplyr_df` object (tibble-like with DuckDB backend)

### `as_duckdb_tibble()`
**Convert existing object to duckplyr data frame**

```r
as_duckdb_tibble(x, ..., prudence = c("lavish", "thrifty", "stingy"))
```

**Parameters:**
- `x`: Object to convert (data.frame, tibble, etc.)
- `...`: Forwarded to methods
- `prudence`: Memory protection level (same as above)

**Returns:** `duckplyr_df` object

### `is_duckdb_tibble()`
**Test if object is duckplyr data frame**

```r
is_duckdb_tibble(x)
```

**Returns:** Logical scalar

## File Reading Functions

### `read_parquet_duckdb()`
**Read Parquet files using DuckDB**

```r
read_parquet_duckdb(
  path,
  ...,
  prudence = c("thrifty", "lavish", "stingy"),
  options = list()
)
```

**Parameters:**
- `path`: File path(s) - supports glob patterns (`*`, `?`), HTTP(S) URLs, S3 paths
- `...`: Reserved for future extensions (must be empty)
- `prudence`: Memory protection level (default: `"thrifty"`)
- `options`: Arguments passed to DuckDB's `read_parquet()` table function

**Returns:** `duckplyr_df` (lazy-evaluated)

### `read_csv_duckdb()`
**Read CSV files using DuckDB**

```r
read_csv_duckdb(
  path,
  ...,
  prudence = c("thrifty", "lavish", "stingy"),
  options = list()
)
```

**Parameters:** Same as `read_parquet_duckdb()`

**Returns:** `duckplyr_df` (lazy-evaluated)

### `read_json_duckdb()`
**Read JSON files using DuckDB**

```r
read_json_duckdb(
  path,
  ...,
  prudence = c("thrifty", "lavish", "stingy"),
  options = list()
)
```

**Parameters:** Same as `read_parquet_duckdb()`

**Returns:** `duckplyr_df` (lazy-evaluated)

### `read_file_duckdb()`
**Read files using DuckDB**

```r
read_file_duckdb(
  path,
  ...,
  prudence = c("thrifty", "lavish", "stingy"),
  options = list()
)
```

**Parameters:** Same as `read_parquet_duckdb()`

**Returns:** `duckplyr_df` (lazy-evaluated)

### `read_tbl_duckdb()` (experimental)
**Read table from DuckDB database file**

```r
read_tbl_duckdb(
  path,
  table_name,
  ...,
  schema = "main",
  prudence = c("thrifty", "lavish", "stingy")
)
```

**Parameters:**
- `path`: Path to DuckDB database file
- `table_name`: Name of table to read
- `...`: Reserved
- `schema`: Schema containing the table (default: `"main"`)
- `prudence`: Memory protection level

**Returns:** `duckplyr_df`

### `read_sql_duckdb()` (experimental)
**Return SQL query as duckdb_tibble**

```r
read_sql_duckdb(
  sql,
  ...,
  prudence = c("thrifty", "lavish", "stingy")
)
```

**Parameters:**
- `sql`: SQL query string
- `...`: Reserved
- `prudence`: Memory protection level

**Returns:** `duckplyr_df`

## Computing & Materialization

### `collect()`
**Force conversion to data frame**

```r
collect(x, ...)
```

**Parameters:**
- `x`: duckplyr_df object
- `...`: Additional arguments

**Returns:** Tibble (in-memory R data frame)

**Note:** Brings lazy-evaluated data into R memory

### `compute()`
**Compute results to temporary DuckDB table**

```r
compute(x, ..., prudence = c("thrifty", "lavish", "stingy"))
```

**Parameters:**
- `x`: duckplyr_df object
- `...`: Additional arguments
- `prudence`: Override memory protection for this operation

**Returns:** `duckplyr_df` (materialized in DuckDB, still lazy from R perspective)

**Use case:** Cache intermediate results in pipeline to avoid re-computation

### `compute_parquet()`
**Compute results to Parquet file**

```r
compute_parquet(x, path, ..., prudence = NULL, options = NULL)
```

**Parameters:**
- `x`: duckplyr_df object (S3 generic also dispatches on `data.frame`)
- `path`: Output file path
- `prudence`: Override memory protection for this operation (default inherits from input)
- `options`: Named list of Parquet writer options forwarded to DuckDB's `COPY ... (FORMAT PARQUET, ...)`. Examples: `list(compression = "zstd", row_group_size = 100000L)`
- `...`: Reserved for future use

**Returns:** `duckplyr_df` pointing to the file

**Note:** Since 1.2.0, `compute_parquet()` is an S3 generic and accepts the `options` argument for format-specific writer settings.

### `compute_csv()`
**Compute results to CSV file**

```r
compute_csv(x, path, ..., prudence = NULL, options = NULL)
```

**Parameters:**
- `x`: duckplyr_df object (S3 generic also dispatches on `data.frame`)
- `path`: Output file path
- `prudence`: Override memory protection for this operation
- `options`: Named list of CSV writer options forwarded to DuckDB's `COPY ... (FORMAT CSV, ...)`. Examples: `list(delimiter = ";", header = TRUE, quote = "\"")`
- `...`: Reserved for future use

**Returns:** `duckplyr_df` pointing to the file

**Note:** Since 1.2.0, `compute_csv()` is an S3 generic and accepts the `options` argument.

### `as_tbl()` (experimental)
**Convert duckplyr frame to dbplyr table for SQL escape hatch**

```r
as_tbl(.data)
```

**Parameters:**
- `.data`: A duckplyr_df or data frame

**Returns:** A dbplyr `tbl_lazy` connected to duckplyr's internal DuckDB connection.

**Use case:** Drop down to dbplyr / hand-written SQL for operations duckplyr does not translate, then convert back with `as_duckdb_tibble()`.

```r
df <- duckdb_tibble(a = 1L)
df |>
  as_tbl() |>
  dplyr::mutate(b = dbplyr::sql("a + 1")) |>
  as_duckdb_tibble()
```

### `explain()`
**Explain query execution plan**

```r
explain(x, ...)
```

**Parameters:**
- `x`: duckplyr_df object
- `...`: Additional arguments

**Returns:** Character vector showing DuckDB query plan

**Use case:** Understand query optimization and execution

### `pull()`
**Extract single column**

```r
pull(x, var, ...)
```

**Parameters:**
- `x`: duckplyr_df object
- `var`: Column name or position
- `...`: Additional arguments

**Returns:** Vector (materializes that column)

## Row Operations

### `filter()`
**Keep rows matching condition**

```r
filter(.data, ..., .by = NULL)
```

**Parameters:**
- `.data`: duckplyr_df object
- `...`: Logical predicates
- `.by`: Optional grouping columns (alternative to `group_by()`)

**Returns:** `duckplyr_df` (filtered)

### `filter_out()` (experimental)
**Remove rows matching condition**

```r
filter_out(.data, ..., .by = NULL)
```

**Parameters:** Same as `filter()`

**Returns:** `duckplyr_df` (inverse filter)

**Note:** Marked experimental as of 1.2.0.

### `distinct()`
**Keep unique rows**

```r
distinct(.data, ..., .keep_all = FALSE)
```

**Parameters:**
- `.data`: duckplyr_df object
- `...`: Columns to use for uniqueness
- `.keep_all`: If TRUE, keep all columns; if FALSE, keep only specified columns

**Returns:** `duckplyr_df` (distinct rows)

### `arrange()`
**Order rows by column values**

```r
arrange(.data, ..., .by_group = FALSE)
```

**Parameters:**
- `.data`: duckplyr_df object
- `...`: Columns to sort by (use `desc()` for descending)
- `.by_group`: If TRUE, sort by groups first

**Returns:** `duckplyr_df` (sorted)

**Note:** Unlike dplyr, duckplyr does NOT automatically sort results

### `slice_head()`
**Subset rows using positions**

```r
slice_head(.data, ..., n, prop, by = NULL)
```

**Parameters:**
- `.data`: duckplyr_df object
- `n`: Number of rows
- `prop`: Proportion of rows
- `by`: Optional grouping

**Returns:** `duckplyr_df` (first n rows)

### `head()`
**Return first parts**

```r
head(x, n = 6L, ...)
```

**Parameters:**
- `x`: duckplyr_df object
- `n`: Number of rows (default: 6)
- `...`: Additional arguments

**Returns:** `duckplyr_df` (first n rows)

## Column Operations

### `select()`
**Keep or drop columns**

```r
select(.data, ...)
```

**Parameters:**
- `.data`: duckplyr_df object
- `...`: Column names or selection helpers

**Returns:** `duckplyr_df` (selected columns)

**Selection helpers:** `starts_with()`, `ends_with()`, `contains()`, `matches()`, `where()`, `everything()`, `last_col()`

### `rename()`
**Rename columns**

```r
rename(.data, ...)
```

**Parameters:**
- `.data`: duckplyr_df object
- `...`: `new_name = old_name` pairs

**Returns:** `duckplyr_df` (renamed columns)

### `relocate()`
**Change column order**

```r
relocate(.data, ..., .before = NULL, .after = NULL)
```

**Parameters:**
- `.data`: duckplyr_df object
- `...`: Columns to move
- `.before`: Column to insert before
- `.after`: Column to insert after

**Returns:** `duckplyr_df` (reordered columns)

### `mutate()`
**Create, modify, delete columns**

```r
mutate(.data, ..., .by = NULL, .keep = c("all", "used", "unused", "none"))
```

**Parameters:**
- `.data`: duckplyr_df object
- `...`: Name-value pairs of expressions
- `.by`: Optional grouping (alternative to `group_by()`)
- `.keep`: Which columns to keep

**Returns:** `duckplyr_df` (with mutations)

### `transmute()` (superseded)
**Create columns and drop others**

```r
transmute(.data, ...)
```

**Parameters:**
- `.data`: duckplyr_df object
- `...`: Name-value pairs

**Returns:** `duckplyr_df` (only new columns)

**Note:** Superseded by `mutate(.keep = "none")`. Since 1.2.0, expressions can reference newly created variables defined earlier in the same call (matching dplyr 1.2.0 behavior).

## Grouping & Summarization

### `count()`
**Count observations in each group**

```r
count(x, ..., wt = NULL, sort = FALSE, name = NULL)
```

**Parameters:**
- `x`: duckplyr_df object
- `...`: Grouping columns
- `wt`: Weight column
- `sort`: If TRUE, sort by count
- `name`: Output column name (default: `"n"`)

**Returns:** `duckplyr_df` (counts per group)

### `summarise()` / `summarize()`
**Summarise each group to one row**

```r
summarise(.data, ..., .by = NULL, .groups = NULL)
```

**Parameters:**
- `.data`: duckplyr_df object
- `...`: Name-value pairs of summary functions
- `.by`: Optional grouping (alternative to `group_by()`)
- `.groups`: Grouping structure of result
  - `NULL`: Default behavior
  - `"drop_last"`: Drop last grouping level
  - `"drop"`: Drop all groups
  - `"keep"`: Keep grouping
  - `"rowwise"`: Make each row a group

**Returns:** `duckplyr_df` (summarized)

**Note:** DuckDB excludes NULLs by default (unlike R's `na.rm = FALSE`)

## Multi-Table Operations

### `left_join()`
**Left join**

```r
left_join(x, y, by = NULL, copy = FALSE, suffix = c(".x", ".y"), ..., keep = NULL)
```

**Parameters:**
- `x`, `y`: duckplyr_df objects to join
- `by`: Join columns (character vector, named vector, or `join_by()`)
- `copy`: Ignored (for compatibility)
- `suffix`: Suffix for duplicate column names
- `keep`: Keep join columns from both tables
- `...`: Additional arguments

**Returns:** `duckplyr_df` (joined)

### `right_join()`, `inner_join()`, `full_join()`
**Other join types**

Same signature as `left_join()`

### `semi_join()`, `anti_join()`
**Filtering joins**

```r
semi_join(x, y, by = NULL, ...)
anti_join(x, y, by = NULL, ...)
```

**Parameters:**
- `x`, `y`: duckplyr_df objects
- `by`: Join columns
- `...`: Additional arguments

**Returns:** `duckplyr_df` (filtered by join condition)

## Set Operations

### `intersect()`, `union()`, `union_all()`, `setdiff()`, `symdiff()`
**Set operations on rows**

```r
intersect(x, y, ...)
union(x, y, ...)
union_all(x, y, ...)
setdiff(x, y, ...)
symdiff(x, y, ...)
```

**Parameters:**
- `x`, `y`: duckplyr_df objects
- `...`: Additional arguments

**Returns:** `duckplyr_df` (result of set operation)

## Configuration & Methods

### `methods_overwrite()`
**Forward all dplyr methods to duckplyr**

```r
methods_overwrite()
```

**Effect:** Overwrites dplyr methods for the session - all data frames use duckplyr

**Note:** Called automatically by `library(duckplyr)`

### `methods_restore()`
**Restore original dplyr methods**

```r
methods_restore()
```

**Effect:** Reverts to standard dplyr behavior

### `fallback_sitrep()`
**Display fallback status report**

```r
fallback_sitrep()
```

**Returns:** Status of fallback configuration

### `fallback_config()`
**Configure fallback behavior**

```r
fallback_config(...)
```

**Parameters:**
- `...`: Configuration options

### `fallback_review()`
**Review fallback occurrences**

```r
fallback_review()
```

**Returns:** Information about when duckplyr fell back to dplyr

### `fallback_upload()`
**Upload fallback log to telemetry endpoint (opt-in)**

```r
fallback_upload()
```

**Effect:** Sends collected fallback details upstream for the duckplyr team to triage missing translations. Opt-in only.

### `fallback_purge()`
**Delete locally collected fallback log entries**

```r
fallback_purge()
```

### `config`
**Configuration options**

Access current configuration settings

### `stats_show()`
**Display computation statistics**

```r
stats_show()
```

**Returns:** Performance statistics for DuckDB operations

### `last_rel()`
**Retrieve recent computation details**

```r
last_rel()
```

**Returns:** The most recent DuckDB relation built by duckplyr. Useful for debugging - inspect what relation tree was constructed before materialization.

### `flights_df()`
**Sample dataset (NYC flights)**

```r
flights_df()
```

**Returns:** Flights data frame used in duckplyr examples and vignettes.

## Database Utilities

### `db_exec()`
**Execute statement on default connection**

```r
db_exec(sql)
```

**Parameters:**
- `sql`: SQL statement string

**Returns:** Result of execution

**Common uses:**
- Install extensions: `db_exec("INSTALL httpfs")`
- Load extensions: `db_exec("LOAD httpfs")`
- Set memory limit: `db_exec("PRAGMA memory_limit = '1GB'")`
- Configure settings: `db_exec("PRAGMA threads = 4")`

## DuckDB Function Passthrough (`dd$`)

Since 1.1.0, prefix any DuckDB scalar/aggregate function with `dd$` inside `mutate()`, `filter()`, `summarise()`, etc. to call it directly without waiting for a duckplyr translation. This bypasses the translation layer and avoids fallback to dplyr.

```r
df |>
  mutate(
    row_id   = dd$ROW_NUMBER(),
    upper    = dd$UPPER(name),
    md5_hash = dd$MD5(name)
  ) |>
  filter(dd$REGEXP_MATCHES(name, "^A"))
```

**Use cases:**
- Access DuckDB-only functions (regex, JSON, list, struct, geo) before duckplyr translates them
- Avoid the fallback-to-dplyr penalty for unsupported functions
- Call any DuckDB function listed in the DuckDB SQL function reference

**Note:** Names after `dd$` are passed through verbatim - they are not R names. Function call syntax must be valid (R parses `dd$ROW_NUMBER()` as `(dd$ROW_NUMBER)()`).

## Key Concepts

### Prudence Levels

Control automatic materialization of intermediate results:

- **"lavish"**: No protection - converts regardless of size (may OOM)
- **"thrifty"** (default): Converts only if <= 1 million cells
- **"stingy"**: Never converts intermediate results automatically

Can override per-operation: `compute(df, prudence = "lavish")`

### Lazy Evaluation

duckplyr uses lazy evaluation - operations aren't executed until:
- `collect()` is called
- Data is printed
- Results are needed for downstream R operations

This enables query optimization across entire pipeline.

### Fallback Mechanism

When operations aren't supported by DuckDB:
1. duckplyr attempts to materialize data (respecting prudence)
2. Falls back to standard dplyr
3. Can be tracked with `fallback_review()`

### Remote Data Access

For HTTP(S) and S3 sources:

```r
db_exec("INSTALL httpfs")
db_exec("LOAD httpfs")
```

Then use regular read functions with URLs:
```r
read_parquet_duckdb("https://example.com/data.parquet")
```

### Performance Tips

1. **Chain operations before materializing** - lazy evaluation optimizes entire pipeline
2. **Use `compute()` strategically** - cache expensive intermediate results
3. **Prefer Parquet over CSV** - columnar format much faster
4. **Set memory limits** - `db_exec("PRAGMA memory_limit = '4GB'")`
5. **Avoid early `collect()`** - keep operations in DuckDB as long as possible
