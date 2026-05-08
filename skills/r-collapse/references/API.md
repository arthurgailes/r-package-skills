# collapse API Reference

Complete function reference for the collapse package (v2.1.6).

## Fast Statistical Functions

Core grouped/weighted statistics. All accept `g` (groups), `w` (weights), `TRA` (transformation), `na.rm`, and `nthreads` arguments.

| Function | Purpose |
|----------|---------|
| `fmean()` | Fast (grouped, weighted) mean |
| `fmedian()` | Fast (grouped, weighted) median |
| `fmode()` | Fast (grouped, weighted) mode |
| `fsum()` | Fast (grouped, weighted) sum |
| `fprod()` | Fast (grouped, weighted) product |
| `fsd()`, `fvar()` | Fast standard deviation/variance |
| `fmin()`, `fmax()` | Fast minimum/maximum |
| `fnth()` | Fast nth element/quantile |
| `fquantile()` | Fast quantile computation |
| `frange()` | Fast range (min/max pair) |
| `ffirst()`, `flast()` | Fast first/last value |
| `fnobs()` | Fast number of non-missing observations |
| `fndistinct()` | Fast number of distinct values |

All support:
- Grouped operations via `g` parameter
- Weighted operations via `w` parameter
- Transformation via `TRA` parameter ("replace", "-", "/", etc.)
- Column-wise (data frames) or vector input

## Fast Grouping and Ordering

| Function | Purpose |
|----------|---------|
| `GRP()` | Create grouping object for fast operations |
| `is_GRP()` | Test for GRP class |
| `GRPN()`, `GRPid()`, `GRPnames()` | Group sizes / id vector / level names |
| `as_factor_GRP()` | Convert GRP to factor |
| `gsplit()` | Fast split by GRP object |
| `greorder()` | Reorder values back to original order after `gsplit` |
| `fgroup_by()` | dplyr-style grouping, returns grouped_df |
| `fungroup()`, `fgroup_vars()`, `group_by_vars()` | Inspect / strip grouping |
| `radixorder()`, `radixorderv()` | Fast radix-based ordering |
| `group()`, `groupv()` | Hash-based grouping (no sort, fastest) |
| `fmatch()`, `ckmatch()` | Fast `match()` with optional check |
| `funique()`, `fnunique()` | Fast unique values / count of uniques |
| `fduplicated()`, `any_duplicated()` | Fast duplicate detection |
| `fcount()`, `fcountv()` | Fast row counts by group (like dplyr count) |
| `qF()`, `qG()` | Quick factor / quick group factor |
| `is_qG()`, `as_factor_qG()` | qG predicates and converters |
| `finteraction()` | Fast factor interaction |
| `fdroplevels()` | Drop unused factor levels |
| `groupid()`, `seqid()`, `timeid()` | Run-length / sequence / time-based ids |

## Fast Data Manipulation

| Function | Purpose |
|----------|---------|
| `fselect()`, `get_vars()` | Fast column selection |
| `add_vars()` | Add columns to a data frame |
| `num_vars()`, `cat_vars()`, `char_vars()`, `fact_vars()`, `logi_vars()`, `date_vars()` | Column-class subsets |
| `fsubset()`, `ss()` | Fast row + column subsetting |
| `fslice()`, `fslicev()` | Fast row slicing by position (added v2.1.0; sf-supported v2.1.1) |
| `fsummarise()` / `fsummarize()` | Fast dplyr-style summarise |
| `fmutate()`, `ftransform()`, `ftransformv()` | Fast transformation/mutate |
| `settransform()`, `settransformv()` | Same, by-reference (modifies in place) |
| `fcompute()`, `fcomputev()` | Compute new columns and drop the rest |
| `across()` | Apply functions across columns inside f-verbs |
| `roworder()`, `roworderv()` | Reorder rows |
| `colorder()`, `colorderv()` | Reorder columns |
| `frename()`, `setrename()` | Fast rename (copy / by reference) |
| `relabel()`, `setrelabel()` | Set variable labels (copy / by reference) |
| `rowbind()` | Fast `rbind()` alternative; handles ragged columns |
| `join()` | Data frame joins (`how =` "left", "right", "inner", "full", "semi", "anti"); `require =` for validation |
| `pivot()` | Reshape long/wide/recast (replaces tidyr pivot_longer/wider) |

## Quick Data Conversion

| Function | Purpose |
|----------|---------|
| `qDF()` | Quick data frame |
| `qDT()` | Quick data.table |
| `qTBL()` | Quick tibble |
| `qM()` | Quick matrix |
| `mctl()`, `mrtl()` | Matrix to list (column-wise / row-wise) |
| `as_numeric_factor()`, `as_integer_factor()`, `as_character_factor()` | Factor conversions |

## Advanced Data Aggregation

| Function | Purpose |
|----------|---------|
| `collap()` | Multi-type, multi-function aggregation (formula `by =`) |
| `collapv()` | Programmer version (column names/indices for `by`) |
| `collapg()` | Grouped-data version (operates on `grouped_df`) |
| `BY()` | Apply arbitrary function by groups |

**collap() features:**
- Multiple aggregation functions per variable
- Separate FUN for numeric vs categorical (`catFUN`)
- Weighted aggregation
- Multithreaded (`parallel = TRUE`, `mc.cores =`)
- Preserves attributes/labels

## Data Transformations

| Function | Purpose |
|----------|---------|
| `dapply()` | Data-apply (column or row-wise) |
| `BY()` | Apply function by groups |
| `TRA()`, `setTRA()` | Apply TRA transformation (copy / by reference) |
| `fscale()`, `STD()` | Scaling and centering (vector and standardize wrapper) |
| `fwithin()`, `W()` | Within-transformation (demeaning); `W()` is the alias |
| `fbetween()`, `B()` | Between-transformation (group means); `B()` is the alias |
| `fhdwithin()`, `HDW()` | Higher-dimensional within (multi-factor demean / residualize) |
| `fhdbetween()`, `HDB()` | Higher-dimensional between (multi-factor fitted values) |

## Linear Models

| Function | Purpose |
|----------|---------|
| `flm()` | Fast OLS regression (multiple methods: `qr`, `chol`, `solve`, `eigen`) |
| `fFtest()` | Fast F-test for nested model comparison |

## Time Series and Panel Data

### Indexing

| Function | Purpose |
|----------|---------|
| `findex_by()` | Create indexed panel data frame |
| `findex()` | Retrieve index from panel data frame |
| `unindex()` | Strip index attributes |
| `reindex()` | Re-attach index to data frame |
| `is_irregular()` | Check if panel is irregularly spaced |
| `to_plm()` | Convert to plm `pdata.frame` |

### Lags / Diffs / Growth

| Function | Purpose |
|----------|---------|
| `flag()`, `L()`, `F()` | Lag / lead values (`L()` lag, `F()` lead) |
| `fdiff()`, `D()`, `Dlog()` | Differences / log-differences |
| `fgrowth()`, `G()` | Growth rates (percent or log) |
| `fcumsum()` | Fast cumulative sum |

### Panel Series

| Function | Purpose |
|----------|---------|
| `psmat()` | Panel series matrix (`fill =` for empty slots, added v2.1.0) |
| `psacf()`, `pspacf()` | Panel autocorrelation / partial autocorrelation |
| `psccf()` | Panel cross-correlation |

## List Processing

| Function | Purpose |
|----------|---------|
| `is_unlistable()` | Check if list is unlistable |
| `ldepth()` | Depth of nested list |
| `atomic_elem()`, `list_elem()` | Extract atomic / list elements |
| `reg_elem()`, `irreg_elem()` | Regular / irregular elements |
| `get_elem()`, `has_elem()` | Recursive element access |
| `rsplit()` | Recursive split |
| `t_list()` | Transpose list of lists |
| `rapply2d()` | Recursive apply to 2D structures |
| `unlist2d()` | Unlist nested list to 2D data frame |

## Summary Statistics

| Function | Purpose |
|----------|---------|
| `qsu()` | Quick summary statistics (grouped, weighted, panel) |
| `qtab()`, `qtable()` | Quick cross-tabulation (with weights / proportions) |
| `descr()` | Detailed descriptive statistics by column |
| `pwcor()`, `pwcov()` | Pairwise correlation / covariance |
| `pwnobs()` | Pairwise number of observations |
| `varying()` | Detect time-varying / between-group-varying columns |

## Other Statistical

| Function | Purpose |
|----------|---------|
| `fdist()` | Fast distance computation (matrix or pairwise) |
| `fquantile()` | Fast quantile computation |
| `frange()` | Fast min/max pair |

## Recode and Replace Values

| Function | Purpose |
|----------|---------|
| `recode_num()` | Recode numeric values |
| `recode_char()` | Recode character values |
| `replace_na()` | Replace NA values |
| `replace_inf()` | Replace `Inf` / `-Inf` |
| `replace_outliers()` | Replace outliers (limit, threshold, NA, clip) |
| `pad()` | Pad panel/time series to balanced |

## Missing Value Handling

| Function | Purpose |
|----------|---------|
| `na_rm()` | Remove NAs (faster `na.omit()`) |
| `na_omit()` | Omit incomplete cases (data frames / matrices) |
| `na_locf()` | Last-observation-carried-forward fill |
| `na_focb()` | First-observation-carried-backward fill |
| `na_insert()` | Insert random NAs (test helper); `set =` for by-reference (added v2.1.2) |
| `missing_cases()` | Logical index of incomplete cases |
| `allNA()` | All-NA test |

## Memory-Efficient Programming

| Function | Purpose |
|----------|---------|
| `anyv()`, `allv()` | Any / all equal a value |
| `whichv()`, `whichNA()` | Position of value / NA |
| `alloc()` | Fast `rep()` (note: v2.1.1 changed list behavior) |
| `copyv()`, `setv()` | Copy / set values by index, by reference |
| `setop()` | In-place arithmetic (`%+=%`, `%-=%`, `%*=%`, `%/=%` operators) |
| `vlengths()`, `vtypes()`, `vclasses()` | Vectorized lengths/types/classes over list |
| `vgcd()` | Greatest common divisor |
| `fnlevels()`, `fnrow()`, `fncol()`, `fdim()` | Fast nlevels/nrow/ncol/dim |
| `seq_row()`, `seq_col()` | Row / column sequences |
| `vec()` | Stack data frame / matrix to vector |
| `cinv()` | Cholesky-based matrix inverse |

## Small Helper Functions

| Function | Purpose |
|----------|---------|
| `.c()` | Quote and concatenate names |
| `massign()` | Multiple assignment |
| `vlabels()`, `setLabels()` | Get / set variable labels |
| `namlab()` | Names + labels summary |
| `add_stub()`, `rm_stub()` | Add / remove name prefix or suffix |
| `all_identical()`, `all_obj_equal()`, `all_funs()` | Identity / equality / function-list checks |
| `setRownames()`, `setColnames()`, `setDimnames()` | Set names by reference |
| `unattrib()`, `setAttrib()`, `setattrib()`, `copyAttrib()`, `copyMostAttrib()` | Attribute helpers |
| `is_categorical()`, `is_date()` | Type predicates |

## Package Options

| Function | Purpose |
|----------|---------|
| `set_collapse()` | Set global collapse options (`nthreads`, `na.rm`, `sort`, `mask`, `stable.algo`, `digits`, `verbose`) |
| `get_collapse()` | Read current option values |

`mask =` accepts `"manip"`, `"helper"`, `"fast-fun"`, `"fast-stat"`, `"fast-trfm"`, `"all"` to expose unprefixed aliases (e.g. `select`, `group_by`, `summarise`).

## Built-in Data

| Object | Description |
|--------|-------------|
| `wlddev` | World Bank development indicators panel (1960-2020) |
| `GGDC10S` | Groningen Growth and Development Centre 10-sector database |

## TRA Argument

The `TRA` parameter enables in-place transformations in statistical functions. Either an integer code or string:

| Value | Transformation |
|-------|----------------|
| `"replace_fill"` / `"fill"` | Replace with statistic, fill missing |
| `"replace"` / `"replace_na"` | Replace with statistic, keep NAs |
| `"-"` | Subtract statistic (demean) |
| `"-+"` | Demean and add overall statistic |
| `"+"` | Add statistic |
| `"*"` | Multiply by statistic |
| `"/"` | Divide by statistic (proportion / rate) |
| `"%"` | Percentage of statistic |
| `"%%"` | Modulo |
| `"-%%"` | Subtract modulo |

**Example:**
```r
# Demean by group (single C pass; no copy)
fmean(mtcars$mpg, mtcars$cyl, TRA = "-")

# Equivalent to:
mtcars$mpg - fmean(mtcars$mpg, mtcars$cyl)

# Group-share of total
fsum(exports$v, exports$country, TRA = "/")
```

## Reference-Semantic Operators

In-place arithmetic from `setop()` avoids intermediate copies:

```r
x %+=% 1                      # x <- x + 1, no copy
data %/=% fsum(data, group)   # divide rows by group sum, in place
```

## Documentation

**Package site:** https://fastverse.org/collapse/
**CRAN:** https://cran.r-project.org/web/packages/collapse/
**Reference index:** https://fastverse.org/collapse/reference/index.html
**NEWS:** https://fastverse.org/collapse/news/index.html
