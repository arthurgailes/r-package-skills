---
name: r-duckplyr
description: Use when code loads or uses duckplyr (library(duckplyr), duckplyr::), processing large datasets with dplyr syntax, working with Parquet files in R, or needing lazy evaluation for bigger-than-memory data
---

# duckplyr: DuckDB-Backed dplyr

## Overview

**duckplyr is a drop-in replacement for dplyr powered by DuckDB for speed and memory efficiency.** It uses identical syntax but lazy evaluation - operations execute only when results are needed, enabling processing of datasets larger than available RAM.

## References

Read `references/API.md` before writing code.

- `references/API.md` - Complete function reference and lazy evaluation patterns

## When to Use duckplyr vs Alternatives

| Use duckplyr when...       | Use dplyr when...      | Use duckspatial when...  | Use data.table when...       |
| -------------------------- | ---------------------- | ------------------------ | ---------------------------- |
| Data >100k rows            | Small datasets (<100k) | Spatial operations       | In-place modification (`:=`) |
| Larger-than-memory files   | All data fits in RAM   | Geospatial joins/buffers | Reference semantics          |
| Parquet/CSV on disk        | Already in memory      | DuckDB + spatial queries | Non-equi joins               |
| Lazy pipeline optimization | Immediate results      | PMTiles, vector tiles    | Keyed/rolling joins          |

**Key insight:** duckplyr works on files without loading into R - queries Parquet/CSV directly from disk or URLs.

## Quick Start

```r
library(duckplyr)

# Convert existing data frame
df <- as_duckdb_tibble(my_data)

# Or read files directly (lazy evaluation)
df <- read_parquet_duckdb("large_file.parquet")

# Standard dplyr syntax
result <- df |>
  filter(year == 2024) |>
  group_by(category) |>
  summarise(total = sum(value)) |>
  collect()  # Materializes result
```

## Critical Differences from dplyr

| Difference          | dplyr                   | duckplyr                                     |
| ------------------- | ----------------------- | -------------------------------------------- |
| **Function name**   | N/A                     | `as_duckdb_tibble()` (not `as_duck_frame()`) |
| **Evaluation**      | Eager (immediate)       | Lazy (until `collect()`)                     |
| **Sorting**         | Auto-sorts groups       | NO auto-sort - use `arrange()`               |
| **NULL handling**   | `na.rm = FALSE` default | Excludes NULLs by default                    |
| **Materialization** | Always in memory        | Controlled by `prudence` parameter           |

### Prudence Levels (Memory Protection)

- `"lavish"`: Converts regardless of size (may OOM)
- `"thrifty"`: Max 1 million cells (default)
- `"stingy"`: Never auto-converts (safest for large data)

```r
read_parquet_duckdb("file.parquet", prudence = "stingy")
```

## Quick Reference

| Task                       | Function                                                       |
| -------------------------- | -------------------------------------------------------------- |
| Read Parquet               | `read_parquet_duckdb(path, prudence = "stingy")`               |
| Read CSV/JSON              | `read_csv_duckdb()`, `read_json_duckdb()`                      |
| Multiple files             | `read_parquet_duckdb("data_*.parquet")` (globs)                |
| Read DuckDB table          | `read_tbl_duckdb(path, table_name, schema = "main")`           |
| Convert data frame         | `as_duckdb_tibble(df)`                                         |
| Bring to R                 | `collect()` (materializes in R memory)                         |
| Cache in DuckDB            | `compute()` (temp table)                                       |
| Write Parquet with options | `compute_parquet(df, "out.parquet", options = list(compression = "zstd"))` |
| Write CSV with options     | `compute_csv(df, "out.csv", options = list(delimiter = ";"))`  |
| Call DuckDB function       | `mutate(x = dd$UPPER(name))` (1.1.0+, skips fallback)          |
| SQL escape hatch           | `as_tbl(df) \|> mutate(... sql("...")) \|> as_duckdb_tibble()` |
| Remote data (HTTP/S3)      | `db_exec("INSTALL httpfs")`, then use URLs                     |
| Query plan                 | `explain(df \|> filter(...))`                                  |
| Memory limit               | `db_exec("PRAGMA memory_limit = '4GB'")`                       |
| Track fallbacks            | `fallback_sitrep()`, `fallback_review()`                       |

## Common Mistakes

| Mistake                          | Fix                                                            |
| -------------------------------- | -------------------------------------------------------------- |
| `as_duck_frame()`                | Use `as_duckdb_tibble()`                                       |
| `as_duckplyr_tibble/df()`        | Deprecated since 1.0.0 - use `as_duckdb_tibble()`              |
| Early `collect()`                | Keep lazy until end                                            |
| No prudence setting              | Set `prudence = "stingy"` for large files                      |
| Expecting auto-sort              | Use explicit `arrange()`                                       |
| arrow/readr instead              | Use `read_*_duckdb()` functions                                |
| Missing httpfs                   | `db_exec("INSTALL httpfs")` for URLs                           |
| No `compute()` caching           | Cache expensive intermediates                                  |
| Triggering fallback unnecessarily| Use `dd$FUNC()` to call DuckDB functions directly (1.1.0+)     |
| Need raw SQL                     | Use `as_tbl()` to drop into dbplyr, then `as_duckdb_tibble()`  |

## When NOT to Use

- Small data (<100k rows) - dplyr, collapse, data.table faster
- Spatial operations - use duckspatial
- In-place modification - use data.table or collapse

**See references/API.md for complete function reference**
