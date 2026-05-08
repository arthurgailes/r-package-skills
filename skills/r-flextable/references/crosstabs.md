# flextable Crosstabs Reference

flextable does not aggregate data, but presents pre-aggregated data beautifully. Three approaches:

## 1. proc_freq() - SAS-Style Contingency Tables

Quick crosstabs mimicking SAS PROC FREQ:

### Two-Way Table
```r
proc_freq(mtcars, row = "gear", col = "vs")
```

Output shows:
- Cell counts
- Row percentages
- Column percentages
- Total percentages

### One-Way Table
```r
proc_freq(mtcars, row = "gear")
```

Output shows frequency and percentage for each level.

### Customization
```r
proc_freq(mtcars, "gear", "vs") |>
  theme_vanilla() |>
  autofit()
```

### Custom Count Formatting (>= 0.9.9)

`proc_freq()` accepts `count_format_fun` to control how integer counts render
(e.g. thousands separator):

```r
proc_freq(
  large_df,
  row = "region",
  col = "status",
  count_format_fun = function(x) format(x, big.mark = ",")
)
```

## 2. tables Package Integration

Use the `tables` package for complex tabulations, then convert to flextable.

### Basic Usage
```r
library(tables)

tab <- tabular(
  (Species + 1) ~ (n = 1) + Format(digits = 2) *
    (Sepal.Length + Sepal.Width) * (mean + sd),
  data = iris
)

as_flextable(tab)
```

**Formula syntax:**
- `+` separates rows/columns
- `*` creates interactions
- `1` adds total row
- `Format()` controls numeric display
- `(n = 1)` adds count column

### Row Groups
```r
tab <- tabular(
  (cut * color + 1) ~ (n = 1) + Format(digits = 2) *
    (price + x) * (mean + sd),
  data = ggplot2::diamonds
)

as_flextable(tab, spread_first_col = TRUE)
```

`spread_first_col = TRUE` displays first grouping variable as row titles.

### Custom Row Titles
```r
as_flextable(tab,
  spread_first_col = TRUE,
  row_title = as_paragraph(
    colorize(as_b(.row_title), color = "#336699")
  )
)
```

## 3. tabulator() - Native flextable Crosstabs

Most flexible approach for custom aggregations.

### Basic Syntax
```r
# First, aggregate your data
library(dplyr)

dat <- ggplot2::diamonds |>
  group_by(cut, color, clarity) |>
  summarise(
    y_mean = mean(y),
    y_sd = sd(y),
    .groups = "drop"
  )

# Then create tabulator
tab <- tabulator(
  x = dat,
  rows = c("cut", "color"),      # Row dimensions
  columns = "clarity",            # Column dimension
  `Y Stats` = as_paragraph(       # Named value column
    fmt_avg_dev(y_mean, y_sd)
  )
)

as_flextable(tab)
```

### Multiple Value Columns
```r
dat <- diamonds |>
  group_by(cut, color, clarity) |>
  summarise(
    y_mean = mean(y), y_sd = sd(y),
    z_mean = mean(z), z_sd = sd(z),
    .groups = "drop"
  )

tab <- tabulator(
  x = dat,
  rows = c("cut", "color"),
  columns = "clarity",
  `Y` = as_paragraph(fmt_avg_dev(y_mean, y_sd)),
  `Z` = as_paragraph(fmt_avg_dev(z_mean, z_sd))
)
```

### Formatting Functions

Built-in formatters for common patterns:

**fmt_avg_dev()** - Mean with standard deviation
```r
fmt_avg_dev(mean_col, sd_col, digit1 = 1, digit2 = 1)
# Output: "12.3 (2.1)"
```

**fmt_n_percent()** - Count with percentage
```r
fmt_n_percent(n_col, pct_col, digit = 1)
# Output: "45 (32.1%)"
```

**fmt_2stats()** - Two statistics
```r
fmt_2stats(stat1, stat2, digit1 = 1, digit2 = 1)
# Output: "12.3 / 5.6"
```

**fmt_header_n()** - Group N in header
```r
fmt_header_n(n_col, newline = TRUE)
# Output: "\n(N=156)"
```

### Hidden Data for Row Labels

Add statistics to row labels without showing in data:
```r
# Separate N calculation
nstat <- dat |>
  group_by(cut) |>
  summarise(n = n())

tab <- tabulator(
  x = dat,
  rows = c("cut", "color"),
  columns = "clarity",
  hidden_data = nstat,
  row_compose = list(
    cut = as_paragraph(
      as_chunk(cut),
      fmt_header_n(n)
    )
  ),
  `Stats` = as_paragraph(fmt_avg_dev(y_mean, y_sd))
)
```

### Supplementary Data Columns

Add columns before or after computed columns:
```r
# Extra column to add
extra <- data.frame(
  cut = unique(dat$cut),
  note = c("Good", "Better", "Best", "Premium", "Ideal")
)

tab <- tabulator(
  x = dat,
  rows = "cut",
  columns = "clarity",
  datasup_first = extra[, c("cut", "note")],  # Before
  # datasup_last = extra,                      # After
  `Stats` = as_paragraph(fmt_avg_dev(y_mean, y_sd))
)
```

### Getting Column Names
```r
# After creating tabulator
colnames <- tabulator_colnames(tab)
# Use for conditional formatting
```

### Post-Processing
```r
tab <- tabulator(...)

ft <- as_flextable(tab) |>
  # Standard flextable operations work
  color(i = ~ cut == "Ideal", color = "blue") |>
  bold(part = "header") |>
  add_header_lines("Diamond Statistics by Cut, Color, and Clarity") |>
  theme_vanilla()
```

## Grouped Data Display

### as_grouped_data()

Transform data for hierarchical display:
```r
library(data.table)

# Wide format with groups
dat <- dcast(
  as.data.table(CO2),
  Treatment + conc ~ Type,
  value.var = "uptake",
  fun.aggregate = mean
)

# Add group structure
dat_grouped <- as_grouped_data(dat, groups = "Treatment")

# Convert to flextable
ft <- as_flextable(dat_grouped) |>
  bold(i = ~ !is.na(Treatment))  # Bold group rows
```

### Styling Group Rows
```r
ft <- as_flextable(dat_grouped) |>
  # Group rows have NA in non-group columns
  bg(i = ~ !is.na(Treatment), bg = "#E8E8E8") |>
  bold(i = ~ !is.na(Treatment)) |>
  # Add mini charts in data rows
  mk_par(
    i = ~ is.na(Treatment),
    j = "sparkline",
    value = as_paragraph(minibar(value, max = max_val))
  )
```

## Summary Tables (summarizor)

Create clinical-style demographic tables:
```r
library(palmerpenguins)

z <- penguins |>
  select(species, island, bill_length_mm, body_mass_g) |>
  summarizor(
    by = "species",
    overall_label = "Overall"
  )

ft <- as_flextable(z, spread_first_col = TRUE)
```

Output includes:
- Categorical: n (%)
- Numeric: mean (sd), median [IQR], range, missing count

### Custom Statistics
```r
z <- penguins |>
  summarizor(
    by = "species",
    fun = list(
      numeric = summarize_numeric,    # Default
      factor = summarize_factor       # Default
    )
  )
```

## Practical Examples

### Basic Crosstab
```r
# Aggregate
counts <- mtcars |>
  count(cyl, gear) |>
  pivot_wider(names_from = gear, values_from = n, values_fill = 0)

# Display
flextable(counts) |>
  add_header_row(values = c("", "Gears"), colwidths = c(1, 3)) |>
  set_header_labels(cyl = "Cylinders") |>
  align(align = "center", part = "all") |>
  theme_vanilla()
```

### Statistical Summary Table
```r
library(dplyr)

summary_dat <- mtcars |>
  group_by(cyl) |>
  summarise(
    n = n(),
    mpg_mean = mean(mpg),
    mpg_sd = sd(mpg),
    hp_mean = mean(hp),
    hp_sd = sd(hp)
  )

tab <- tabulator(
  x = summary_dat,
  rows = "cyl",
  `N` = as_paragraph(as_chunk(n)),
  `MPG` = as_paragraph(fmt_avg_dev(mpg_mean, mpg_sd)),
  `HP` = as_paragraph(fmt_avg_dev(hp_mean, hp_sd))
)

as_flextable(tab) |>
  set_header_labels(cyl = "Cylinders") |>
  add_header_lines("Summary by Cylinder Count") |>
  theme_vanilla()
```

### Multi-Dimensional Table
```r
# Three-way aggregation
dat <- ggplot2::diamonds |>
  filter(cut %in% c("Good", "Very Good", "Ideal")) |>
  group_by(cut, color, clarity) |>
  summarise(
    n = n(),
    price_mean = mean(price),
    price_sd = sd(price),
    .groups = "drop"
  )

tab <- tabulator(
  x = dat,
  rows = c("cut", "color"),
  columns = "clarity",
  `N` = as_paragraph(as_chunk(n)),
  `Price` = as_paragraph(
    "$", fmt_avg_dev(price_mean, price_sd, digit1 = 0, digit2 = 0)
  )
)

as_flextable(tab) |>
  labelizor(part = "header", labels = function(x) gsub("_", " ", x)) |>
  theme_vanilla() |>
  autofit()
```
