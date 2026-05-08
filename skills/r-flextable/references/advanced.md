# flextable Advanced Reference

## Multi-Content Cells (compose/mk_par)

### Basic Structure

`mk_par()` (alias: `compose()`) replaces cell content with rich formatted paragraphs:

```r
ft |> mk_par(
  i = row_selector,           # Which rows
  j = column_selector,        # Which columns
  value = as_paragraph(...),  # Content
  part = "body"               # Which part
)
```

### Chunk Functions

**Text chunks:**
| Function | Effect |
|----------|--------|
| `as_chunk(x)` | Plain text |
| `as_b(x)` | Bold |
| `as_i(x)` | Italic |
| `as_u(x)` | Underline |
| `as_sub(x)` | Subscript |
| `as_sup(x)` | Superscript |
| `as_highlight(x, color)` | Highlighted |
| `colorize(x, color)` | Colored text |
| `hyperlink_text(x, url)` | Hyperlink |

**With formatting properties:**
```r
as_chunk(
  value,
  props = fp_text_default(
    bold = TRUE,
    color = "red",
    font.size = 12,
    font.family = "Arial"
  )
)
```

### Combining Chunks
```r
ft |> mk_par(
  j = "species",
  value = as_paragraph(
    as_b(common_name),
    " (",
    as_i(latin_name),
    ")"
  )
)
```

### Conditional Content
```r
ft |> mk_par(
  j = "status",
  value = as_paragraph(
    as_chunk(
      ifelse(value > 0, "\u2191", "\u2193"),  # Up/down arrows
      props = fp_text_default(
        color = ifelse(value > 0, "green", "red")
      )
    ),
    " ",
    as_chunk(abs(value))
  )
)
```

### Bracket Notation
```r
# Add confidence intervals
ft |> mk_par(
  j = "estimate",
  value = as_paragraph(
    as_chunk(estimate),
    " ",
    as_bracket(as_chunk(ci_low), as_chunk(ci_high))  # [low, high]
  )
)
```

### Soft Returns and Tabs
```r
# Line break within cell
ft |> mk_par(
  j = "address",
  value = as_paragraph(
    as_chunk(street),
    as_chunk("\n"),
    as_chunk(city), ", ", as_chunk(state)
  )
)

# Tab stops (Word/RTF only)
ft |>
  prepend_chunks(i = 2:5, j = "col", as_chunk("\t")) |>
  tab_settings(j = "col", value = fp_tabs(fp_tab(pos = 0.5)))
```

## Images in Cells

### External Images
```r
# From file paths
ft |> mk_par(
  j = "photo",
  value = as_paragraph(
    as_image(src = photo_path, width = 0.5, height = 0.5)
  )
)
```

### From URLs
```r
ft |> mk_par(
  j = "flag",
  value = as_paragraph(
    as_image(src = flag_url, width = 0.3, height = 0.2)
  )
)
```

### Using colformat_image()
```r
# When column contains file paths
ft |> colformat_image(
  j = "image_path",
  width = 0.5,
  height = 0.5
)
```

**Note:** Images don't work in PowerPoint table cells. Export table as image instead.

## Mini Charts

### minibar() - Inline Bar Charts
```r
dat <- data.frame(
  category = c("A", "B", "C"),
  value = c(30, 70, 50),
  max_val = 100
)

flextable(dat) |>
  mk_par(
    j = "value",
    value = as_paragraph(
      minibar(value, max = max_val, barcol = "steelblue", height = 0.2)
    )
  ) |>
  width(j = "value", width = 1.5)
```

### linerange() - Confidence Intervals
```r
dat <- data.frame(
  group = c("A", "B"),
  estimate = c(2.5, 3.1),
  ci_low = c(1.8, 2.4),
  ci_high = c(3.2, 3.8)
)

flextable(dat) |>
  mk_par(
    j = "estimate",
    value = as_paragraph(
      linerange(
        value = estimate,
        min = ci_low,
        max = ci_high,
        rangecol = "gray",
        stickcol = "black",
        bg = "transparent",
        width = 1.5,
        height = 0.2
      )
    )
  )
```

### plot_chunk() - Base R Plots
```r
# Create plot functions
dat$sparkfun <- lapply(dat$values, function(x) {
 function() {
    par(mar = c(0, 0, 0, 0))
    plot(x, type = "l", axes = FALSE)
  }
})

flextable(dat) |>
  mk_par(
    j = "sparkfun",
    value = as_paragraph(
      plot_chunk(sparkfun, width = 1, height = 0.3)
    )
  )
```

### gg_chunk() - ggplot2 Objects
```r
library(ggplot2)

# Add ggplot objects as list column
dat$ggplot <- lapply(dat$data_list, function(d) {
  ggplot(d, aes(x, y)) +
    geom_line(color = "steelblue") +
    theme_void()
})

flextable(dat) |>
  mk_par(
    j = "ggplot",
    value = as_paragraph(
      gg_chunk(ggplot, width = 1.5, height = 0.4)
    )
  )
```

### Looping with use_dot
```r
# Apply to multiple columns at once
ft |> mk_par(
  j = ~ sparkline_a + sparkline_b + sparkline_c,
  value = as_paragraph(
    gg_chunk(., width = 1, height = 0.3)
  ),
  use_dot = TRUE  # '.' represents each column's values
)
```

## Patchwork Integration

Combine flextables with ggplot2 visualizations:

### Basic Combination
```r
library(patchwork)

p <- ggplot(mtcars, aes(mpg, hp)) + geom_point()
ft <- flextable(head(mtcars[, 1:4]))

# Side by side
wrap_flextable(ft) | p

# Stacked
wrap_flextable(ft) / p

# Complex layouts
(wrap_flextable(ft) | p) / p2
```

### Row Alignment (flex_body)
```r
# Table body rows stretch to match plot height
wrap_flextable(ft, flex_body = TRUE) + p

# Useful for matching categorical data to plot categories
```

### Column Alignment (flex_cols)
```r
# Data columns expand to match plot width
wrap_flextable(ft, flex_cols = TRUE, n_row_headers = 1) / p

# n_row_headers excludes leading columns from stretching
```

### Positioning (just)
```r
wrap_flextable(ft, just = "left")    # Default
wrap_flextable(ft, just = "center")
wrap_flextable(ft, just = "right")
```

### Panel Alignment
```r
wrap_flextable(ft, panel = "body")  # Default: header/footer outside
wrap_flextable(ft, panel = "full")  # Entire table inside panel
```

### Layout Control
```r
wrap_flextable(ft) | p +
  plot_layout(widths = c(1, 2))  # Table narrower than plot

wrap_flextable(ft) / p +
  plot_layout(heights = c(1, 3))  # Table shorter than plot
```

## Programming Utilities

### Part Dimensions
```r
nrow_part(ft, part = "body")     # Number of body rows
nrow_part(ft, part = "header")   # Number of header rows
ncol_keys(ft)                    # Number of columns
```

### Dynamic Last Row
```r
ft |> bold(i = nrow_part(ft, part = "body"), part = "body")
```

### before() Function
Select rows before a condition:
```r
ft |> hline(
  i = ~ before(category, "Total"),
  border = fp_border(width = 2)
)
```

### Programmatic Column Iteration
```r
# Apply to multiple columns dynamically
cols_to_format <- c("col1", "col2", "col3")

for (col in cols_to_format) {
  ft <- ft |> mk_par(
    j = col,
    value = as_paragraph(gg_chunk(get(col), width = 1, height = 0.3))
  )
}

# Or with purrr
ft <- reduce(cols_to_format, function(ft, col) {
  mk_par(ft, j = col, value = as_paragraph(...))
}, .init = ft)
```

## Model Output Tables

### Linear Models (lm)
```r
model <- lm(mpg ~ cyl + hp + wt, data = mtcars)
as_flextable(model)
```

### Generalized Linear Models (glm)
```r
model <- glm(am ~ mpg + hp, data = mtcars, family = binomial)
as_flextable(model)
```

### GAM Models
```r
library(mgcv)
model <- gam(y ~ s(x1) + s(x2), data = dat)
as_flextable(model)
```

### Statistical Tests (htest)
```r
test <- t.test(mpg ~ am, data = mtcars)
as_flextable(test)

test <- chisq.test(table(mtcars$cyl, mtcars$gear))
as_flextable(test)
```

### Clustering Results
```r
km <- kmeans(mtcars[, c("mpg", "hp")], centers = 3)
as_flextable(km)
```

### xtable Objects
```r
library(xtable)
xt <- xtable(summary(model))
as_flextable(xt)
```

## Equations

### LaTeX Equations
```r
ft |> mk_par(
  j = "formula",
  value = as_paragraph(
    as_equation(formula_latex, width = 2, height = 0.3)
  )
)
```

**Note:** Requires `equatiomatic` or manual LaTeX strings.

## Quarto Markdown in Cells

With `flextable-qmd` filter:
```r
ft |> mk_par(
  j = "notes",
  value = as_paragraph(
    as_qmd("**Bold**, *italic*, and `code`")
  )
)
```

Supports: bold, italic, links, code, math, strikethrough.

## Advanced Formatting Patterns

### Heatmap Coloring
```r
library(scales)

ft |> bg(
  j = ~ col1 + col2 + col3,
  bg = col_numeric(
    palette = c("white", "steelblue"),
    domain = c(0, 100)
  )
)
```

### Traffic Light Indicators
```r
ft |> mk_par(
  j = "status",
  value = as_paragraph(
    as_chunk(
      "\u25CF",  # Filled circle
      props = fp_text_default(
        color = case_when(
          status == "good" ~ "green",
          status == "warning" ~ "orange",
          TRUE ~ "red"
        ),
        font.size = 16
      )
    )
  )
)
```

### Progress Bars
```r
ft |> mk_par(
  j = "progress",
  value = as_paragraph(
    minibar(
      value = progress,
      max = 100,
      barcol = ifelse(progress >= 80, "green",
                      ifelse(progress >= 50, "orange", "red")),
      height = 0.15
    )
  )
)
```

### Trend Arrows
```r
ft |> mk_par(
  j = "change",
  value = as_paragraph(
    as_chunk(
      case_when(
        change > 0 ~ "\u2191",   # Up arrow
        change < 0 ~ "\u2193",   # Down arrow
        TRUE ~ "\u2194"          # Left-right arrow
      ),
      props = fp_text_default(
        color = case_when(
          change > 0 ~ "green",
          change < 0 ~ "red",
          TRUE ~ "gray"
        )
      )
    ),
    " ",
    as_chunk(sprintf("%.1f%%", abs(change)))
  )
)
```

### Conditional Row Styling
```r
ft |>
  # Highlight outliers
  bg(i = ~ value > mean(value) + 2 * sd(value), bg = "#FFE0E0") |>
  # Bold significant rows
  bold(i = ~ pvalue < 0.05) |>
  # Gray out non-significant
  color(i = ~ pvalue >= 0.05, color = "gray")
```

### Alternating Row Colors (Manual)
```r
ft |> bg(
  i = seq(2, nrow_part(ft, "body"), by = 2),
  bg = "#F5F5F5"
)
```

## Performance Tips

1. **Build data first:** Complete all data transformations before creating flextable
2. **Batch formatting:** Use selectors to format multiple cells at once
3. **Avoid loops:** Use vectorized `mk_par()` with `use_dot = TRUE`
4. **Large tables:** Consider `autofit()` alternatives for 1000+ rows
5. **Mini charts:** Pre-compute ggplot objects, don't generate in `mk_par()`
