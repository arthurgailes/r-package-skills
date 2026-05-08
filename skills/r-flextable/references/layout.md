# flextable Layout Reference

## Cell Merging

### merge_v() - Vertical Merge
Merge adjacent cells with same value vertically:
```r
dat <- data.frame(
  group = c("A", "A", "B", "B"),
  item = c("x", "y", "x", "y"),
  value = 1:4
)

flextable(dat) |>
  merge_v(j = "group") |>      # Merge duplicate "A" and "B"
  valign(valign = "top")       # Align merged content to top
```

**Gotcha:** After merging, call `fix_border_issues()` to correct borders.

### merge_h() - Horizontal Merge
Merge adjacent cells with same value horizontally:
```r
flextable(dat) |>
  merge_h()  # Merges across all columns where adjacent values match
```

### merge_h_range() - Merge Column Range
Merge specific column range regardless of values:
```r
flextable(dat) |>
  merge_h_range(i = 1, j1 = "col1", j2 = "col3")  # Merge cols 1-3 in row 1
```

### merge_at() - Merge Rectangle
Merge a specific rectangular region:
```r
flextable(dat) |>
  merge_at(i = 1:2, j = 1:2)  # Merge 2x2 block at top-left
```

Content from top-left cell is kept.

### merge_none() - Unmerge All
Remove all merge information:
```r
flextable(dat) |>
  merge_none()
```

### fix_border_issues()
After merging, borders may not render correctly:
```r
flextable(dat) |>
  merge_v(j = "group") |>
  theme_box() |>
  fix_border_issues()
```

## Table Sizing

### set_table_properties()
Control overall table layout:
```r
ft |> set_table_properties(
  layout = "autofit",   # "fixed" (default) or "autofit"
  width = 1,            # 0-1 proportion of available width (1 = 100%)
  align = "center"      # Table alignment: "left", "center", "right"
)
```

**Layout options:**
- `"fixed"`: Manual column widths via `width()`
- `"autofit"`: Word/HTML compute widths from content

**Warning:** `autofit()` function is different from `layout = "autofit"`. The function computes minimal widths ignoring page margins. The layout property uses Word's algorithm respecting margins.

### width()
Set individual column widths (in inches):
```r
ft |> width(j = 1, width = 2)           # Column 1 = 2 inches
ft |> width(j = "name", width = 1.5)
ft |> width(width = c(1, 2, 1.5, 1))    # All columns
```

### height()
Set row heights (in inches):
```r
ft |>
  height(i = 1, height = 0.5) |>
  hrule(rule = "exact")          # Required for height to work
```

**Rules:**
- `"auto"` (default): Height from content
- `"atleast"`: Minimum height
- `"exact"`: Exact height (required for `height()` to take effect)

### autofit()
Compute minimal widths and heights:
```r
ft |> autofit()
ft |> autofit(add_w = 0.1, add_h = 0.05)  # Add padding
```

**Warning:** May overflow page margins. Use `set_table_properties(layout = "autofit")` instead for Word.

### fit_to_width()
Scale table to fit specific width:
```r
ft |> fit_to_width(max_width = 6)  # Max 6 inches
```

### dim_pretty()
Preview optimal dimensions without applying:
```r
dim_pretty(ft)
# $widths: [1] 1.2 0.8 1.5 ...
# $heights: [1] 0.3 0.3 0.3 ...
```

### dim()
Get current dimensions:
```r
dim(ft)
```

## Headers and Footers

### Visible Columns (col_keys)
Control which columns display and their order:
```r
flextable(mtcars, col_keys = c("mpg", "cyl", "hp"))  # Subset
flextable(mtcars, col_keys = c("mpg", "", "hp"))    # With blank separator
```

### set_header_labels()
Replace header labels:
```r
ft |> set_header_labels(
  mpg = "Miles per Gallon",
  cyl = "Cylinders",
  hp = "Horsepower"
)
```

### labelizor()
Replace values throughout table:
```r
# Named vector replacement
ft |> labelizor(
  part = "header",
  labels = c("mpg" = "MPG", "cyl" = "Cylinders")
)

# Function replacement
ft |> labelizor(
  part = "header",
  labels = tools::toTitleCase
)
```

### separate_header()
Split concatenated column names into multi-row header:
```r
# Data with names like "measure_mean", "measure_sd"
dat <- data.frame(
  group = c("A", "B"),
  value_mean = c(1.2, 3.4),
  value_sd = c(0.1, 0.2)
)

flextable(dat) |>
  separate_header()  # Splits on "_" and "."
```

**Options:**
```r
separate_header(
  opts = c(
    "span-top",       # Span empty cells vertically
    "center-hspan",   # Center horizontally spanned cells
    "bottom-vspan",   # Bottom-align vertically spanned cells
    "default-theme"   # Apply theme to new header rows
  )
)
```

**Custom separator:**
```r
separate_header(split = "/", fixed = TRUE)
```

### add_header_row()
Add row with merged cells:
```r
# For a 5-column table: first value spans 3 cols, second spans 2
ft |> add_header_row(
  values = c("Group A", "Group B"),
  colwidths = c(3, 2),        # Must sum to total columns (3+2=5)
  top = TRUE                   # Add at top (default) or bottom
)
```

**Critical:** `values` has one entry per span, NOT one per column. `colwidths` says how many columns each value spans. Sum of colwidths must equal total column count.

```r
# WRONG - don't repeat values for each column
add_header_row(values = c("A", "A", "A", "B", "B"), colwidths = c(1,1,1,1,1))

# RIGHT - one value per span
add_header_row(values = c("A", "B"), colwidths = c(3, 2))
```

### add_header_lines()
Add full-width text rows (all columns merged):
```r
ft |> add_header_lines(
  values = c("Table Title", "Subtitle")
)
```

### add_header() / add_footer()
Column-oriented addition:
```r
ft |> add_header(
  mpg = "Performance",
  cyl = "Engine",
  hp = "Engine"
)
```

### set_header_df()
Build complex headers from a data.frame:
```r
header_map <- data.frame(
  col_keys = c("mpg", "cyl", "hp", "wt"),
  row1 = c("Performance", "Performance", "Engine", "Weight"),
  row2 = c("MPG", "Cylinders", "HP", "Tons"),
  stringsAsFactors = FALSE
)

ft |>
  set_header_df(mapping = header_map, key = "col_keys") |>
  merge_h(part = "header") |>
  merge_v(part = "header") |>
  theme_vanilla()
```

### add_footer_lines()
```r
ft |> add_footer_lines(
  values = c(
    "Source: Motor Trend, 1974",
    "Note: Values are estimates"
  )
)
```

### add_footer_row()
Same as `add_header_row()` but for footer:
```r
ft |> add_footer_row(
  values = c("Total", "", "100"),
  colwidths = c(1, 1, 1)
)
```

## Footnotes

### footnote()
Add footnotes with reference marks:
```r
ft |> footnote(
  i = 1, j = 2,                          # Cell location
  value = as_paragraph("p < 0.05"),
  ref_symbols = "*",
  part = "body"
)

# Multiple footnotes
ft |> footnote(
  i = c(1, 2, 3),
  j = c(2, 2, 3),
  value = as_paragraph(c("Note A", "Note B", "Note C")),
  ref_symbols = c("a", "b", "c")
)

# Several symbols on one cell (flextable >= 0.9.11): symbol_sep
# controls how multiple ref symbols are joined when they collide
ft |> footnote(
  i = c(1, 1), j = c(2, 2),
  value = as_paragraph(c("Note A", "Note B")),
  ref_symbols = c("a", "b"),
  symbol_sep = ","
)
```

Notes:
- An empty `ref_symbols` value (`""`) is not allowed (defunct since 0.9.9).
  Use `add_footer_lines()` for unmarked notes.

### Superscript in Headers
```r
ft |> mk_par(
  j = "pvalue",
  value = as_paragraph("P", as_sup("*")),
  part = "header"
)
```

## Captions

### set_caption()
```r
ft |> set_caption(
  caption = "Table 1: Motor Trend Car Data",
  autonum = run_autonum(seq_id = "tab", bkm = "mtcars_table")
)
```

**With formatting:**
```r
ft |> set_caption(
  caption = as_paragraph(
    "Table 1: ",
    as_b("Motor Trend"),
    " Car Data"
  )
)
```

**Paragraph properties:**
```r
ft |> set_caption(
  caption = "Table 1",
  fp_p = fp_par(text.align = "left", padding.bottom = 10)
)
```

### Knitr Chunk Options
```r
#| tab.id: mtcars-table
#| tab.cap: Motor Trend car road tests
#| tab.cap.style: Table Caption
#| tab.topcaption: true

flextable(head(mtcars))
```

### Cross-References

**Bookdown:**
```markdown
See Table \@ref(tab:mtcars-table) for details.
```

**Quarto:**
```markdown
See @tbl-mtcars for details.
```

Requires `flextable-qmd` Lua filter for Quarto.

## Pagination (Word/RTF)

### paginate()
Control page breaks:
```r
# Keep small table together
ft |> paginate(init = TRUE, hdr_ftr = TRUE)

# Keep header with first body row (large tables)
ft |> paginate(init = FALSE, hdr_ftr = TRUE)

# Keep groups together
ft |> paginate(
  group = "category",
  group_def = "rle"      # "rle" or "nonempty"
)
```

### keep_with_next()
Keep specific rows with next row:
```r
ft |> keep_with_next(i = c(1, 3, 5))
```

## Practical Examples

### Grouped Table with Merged Cells
```r
library(dplyr)

mtcars |>
  arrange(cyl, mpg) |>
  select(cyl, mpg, hp, wt) |>
  flextable() |>
  merge_v(j = "cyl") |>
  valign(j = "cyl", valign = "top") |>
  hline(i = ~ cyl != lead(cyl), border = fp_border(width = 2)) |>
  fix_border_issues() |>
  theme_vanilla() |>
  autofit()
```

### Multi-Level Header
```r
ft <- flextable(head(mtcars, 5),
                col_keys = c("mpg", "cyl", "disp", "hp", "drat", "wt"))

ft |>
  add_header_row(
    values = c("Performance", "Engine", "Weight"),
    colwidths = c(2, 3, 1)
  ) |>
  set_header_labels(
    mpg = "MPG", cyl = "Cyl", disp = "Disp",
    hp = "HP", drat = "Drat", wt = "Wt"
  ) |>
  align(align = "center", part = "header") |>
  merge_v(part = "header") |>
  theme_vanilla()
```

### Summary Row
```r
dat <- mtcars |>
  summarise(
    n = n(),
    mpg_mean = mean(mpg),
    hp_mean = mean(hp)
  ) |>
  mutate(label = "Total") |>
  select(label, everything())

flextable(dat) |>
  bold(i = 1) |>
  hline_top(border = fp_border(width = 2))
```
