# flextable API Reference

Complete function reference for the flextable package (240+ functions).

## Create a Flextable

| Function | Purpose |
|----------|---------|
| `flextable()` | Main constructor to create flextable objects from data frames |
| `qflextable()` | Quick shortcut for creating flextables |
| `as_flextable()` | Generic method to transform objects into flextables |

**Example:**
```r
library(flextable)

# Basic table
ft <- flextable(mtcars)

# Quick table with defaults
qflextable(iris)
```

## Convert Objects to Flextable

| Function | Purpose |
|----------|---------|
| `tabulator()` | Create pivot-style summary tables |
| `summary(<tabulator>)` | Summary method for tabulator objects |
| `tabulator_colnames()` | Get column keys of tabulator objects |
| `as_flextable(<tabulator>)` | Transform tabulator object into flextable |
| `summarizor()` | Prepare descriptive statistics for display |
| `as_flextable(<summarizor>)` | Transform summarizor object into flextable |
| `as_flextable(<compact_summary>)` | Transform compact summary objects |
| `as_flextable(<data.frame>)` | Transform and summarize data frames |
| `as_flextable(<gam>)` | Convert generalized additive models |
| `as_flextable(<glm>)` | Convert generalized linear models |
| `as_flextable(<grouped_data>)` | Transform grouped data objects |
| `as_flextable(<htest>)` | Convert hypothesis test results |
| `as_flextable(<kmeans>)` | Transform kmeans clustering results |
| `as_flextable(<lm>)` | Convert linear models |
| `as_flextable(<merMod>)` | Convert mixed-effects models (merMod, lme, gls, nlme, brmsfit, glmmTMB, glmmadmb) |
| `as_flextable(<pam>)` | Transform partitioning around medoids results |
| `as_flextable(<table>)` | Transform table objects |
| `as_flextable(<tabular>)` | Transform tables::tabular object |
| `as_flextable(<xtable>)` | Transform xtable object into flextable |
| `continuous_summary()` | Summarize continuous variables |
| `compact_summary()` | Create compact dataset summaries |
| `as_grouped_data()` | Insert group-label rows into data frames |
| `proc_freq()` | Generate frequency tables (gained `count_format_fun` arg in 0.9.9) |
| `shift_table()` | Create shift tables |

**Example:**
```r
# Convert model to table
model <- lm(mpg ~ wt + cyl, data = mtcars)
as_flextable(model)

# Create summary statistics
summarizor(iris, by = "Species") |>
  as_flextable()
```

## Headers, Footers & Structural Changes

| Function | Purpose |
|----------|---------|
| `add_header_row()` | Add header row with spanning labels |
| `add_header_lines()` | Add full-width header rows |
| `add_header()` | Add header rows with one value per column |
| `add_footer_row()` | Add footer row with spanning labels |
| `add_footer_lines()` | Add full-width footer rows |
| `add_footer()` | Add footer rows with one value per column |
| `add_body()` | Add body rows with one value per column |
| `add_body_row()` | Add body row with spanning labels |
| `separate_header()` | Split column names into multiple rows |
| `set_header_labels()` | Rename column labels in header |
| `set_header_df()` | Replace entire header from data frame |
| `set_footer_df()` | Replace entire footer from data frame |
| `delete_part()` | Delete table parts (header, body, footer) |
| `delete_rows()` | Remove rows from table |
| `delete_columns()` | Remove columns from table |

**Example:**
```r
ft |>
  add_header_row(values = c("Group A", "Group B"), colwidths = c(2, 3)) |>
  add_footer_lines("Source: Internal data")
```

## Merge Cells

| Function | Purpose |
|----------|---------|
| `merge_at()` | Merge cells into a single cell |
| `merge_h()` | Merge cells horizontally |
| `merge_h_range()` | Merge column range rowwise |
| `merge_none()` | Remove all merging information |
| `merge_v()` | Merge cells vertically |

**Example:**
```r
ft |>
  merge_v(j = "Species") |>
  merge_h(i = 1, part = "header")
```

## Format Cell Values

| Function | Purpose |
|----------|---------|
| `colformat_char()` | Format character columns |
| `colformat_date()` | Format date columns |
| `colformat_datetime()` | Format datetime columns |
| `colformat_double()` | Format double columns |
| `colformat_image()` | Display images in cells |
| `colformat_int()` | Format integer columns |
| `colformat_lgl()` | Format logical columns |
| `colformat_num()` | Format numeric columns with format() |
| `set_formatter()` | Apply custom formatter functions |
| `labelizor()` | Replace displayed text with labels |

**Example:**
```r
ft |>
  colformat_double(j = "mpg", digits = 1) |>
  colformat_date(j = "date", fmt_date = "%Y-%m-%d")
```

## Compose Rich Content

| Function | Purpose |
|----------|---------|
| `compose()` | Set cell content from paragraph chunks |
| `mk_par()` | Alternative to compose() |
| `as_paragraph()` | Build paragraphs from chunks |
| `append_chunks()` | Add chunks to existing content |
| `prepend_chunks()` | Add chunks before existing content |
| `footnote()` | Add footnotes to table (gained `symbol_sep` arg in 0.9.11 for separating multiple footnote symbols in a cell) |

**Example:**
```r
ft |>
  compose(
    j = "text",
    value = as_paragraph(
      "Mean: ", as_chunk(mean_val, formatter = fmt_dbl),
      " (SD: ", as_chunk(sd_val, formatter = fmt_dbl), ")"
    )
  )
```

## Content Chunks

| Function | Purpose |
|----------|---------|
| `as_chunk()` | Create text chunk |
| `as_b()` | Create bold text chunk |
| `as_i()` | Create italic text chunk |
| `as_strike()` | Create strikethrough text chunk |
| `as_sub()` | Create subscript text chunk |
| `as_sup()` | Create superscript text chunk |
| `as_highlight()` | Create highlighted text chunk |
| `colorize()` | Create colored text chunk |
| `as_qmd()` | Create Quarto markdown chunk |
| `as_bracket()` | Create bracketed text chunk |
| `hyperlink_text()` | Create hyperlink chunk |
| `as_equation()` | Create equation chunk |
| `as_word_field()` | Create Word dynamic field chunk |
| `as_image()` | Create image chunk |
| `gg_chunk()` | Create ggplot chunk |
| `plot_chunk()` | Create mini plot chunk |
| `grid_chunk()` | Create grid graphics chunk |
| `minibar()` | Create mini barplot chunk |
| `linerange()` | Create mini linerange chunk |

**Example:**
```r
# Bold and italic
as_paragraph(as_b("Bold"), " and ", as_i("italic"))

# Mini chart
compose(
  j = "trend",
  value = as_paragraph(minibar(values))
)

# Image
compose(
  j = "logo",
  value = as_paragraph(as_image(src = "path/to/image.png"))
)
```

## Apply Themes

| Function | Purpose |
|----------|---------|
| `theme_alafoli()` | Apply alafoli visual theme |
| `theme_apa()` | Apply APA style theme |
| `theme_booktabs()` | Apply booktabs theme |
| `theme_borderless()` | Apply borderless theme |
| `theme_box()` | Apply box theme |
| `theme_tron()` | Apply tron theme |
| `theme_tron_legacy()` | Apply tron legacy theme |
| `theme_vader()` | Apply Darth Vader (Sith) theme |
| `theme_vanilla()` | Apply vanilla theme |
| `theme_zebra()` | Apply zebra striping theme |

**Example:**
```r
ft |>
  theme_booktabs() |>
  autofit()
```

## Style Text and Cells

| Function | Purpose |
|----------|---------|
| `style()` | Set formatting properties on selections |
| `font()` | Set font family |
| `fontsize()` | Set font size |
| `bold()` | Apply bold formatting |
| `italic()` | Apply italic formatting |
| `color()` | Set font color |
| `highlight()` | Set text highlight color |
| `bg()` | Set cell background color |
| `fp_text_default()` | Create text formatting with defaults |

**Example:**
```r
ft |>
  bold(i = 1, bold = TRUE, part = "header") |>
  bg(i = ~ mpg > 20, bg = "yellow") |>
  color(j = "cyl", color = "red")
```

## Alignment and Spacing

| Function | Purpose |
|----------|---------|
| `align()` | Set horizontal text alignment |
| `align_text_col()` | Align text columns |
| `align_nottext_col()` | Align non-text columns |
| `valign()` | Set vertical alignment |
| `padding()` | Set cell padding |
| `line_spacing()` | Set line spacing within cells |
| `rotate()` | Rotate cell text |
| `tab_settings()` | Configure tabulation marks |
| `empty_blanks()` | Make blank columns transparent |

**Example:**
```r
ft |>
  align(align = "center", part = "all") |>
  padding(padding = 10, part = "body")
```

## Borders

| Function | Purpose |
|----------|---------|
| `fp_border_default()` | Create border formatting with defaults |
| `hline()` | Set horizontal borders below rows |
| `hline_bottom()` | Set table part bottom border |
| `hline_top()` | Set table part top border |
| `vline()` | Set vertical borders to right of columns |
| `vline_left()` | Set left table border |
| `vline_right()` | Set right table border |
| `border_inner()` | Set all inner borders |
| `border_inner_h()` | Set inner horizontal borders |
| `border_inner_v()` | Set inner vertical borders |
| `border_outer()` | Set outer borders |
| `border_remove()` | Remove all borders |
| `surround()` | Surround cells with borders |

**Example:**
```r
ft |>
  border_remove() |>
  hline_top(border = fp_border(width = 2)) |>
  hline_bottom(border = fp_border(width = 2))
```

## Table and Column Dimensions

| Function | Purpose |
|----------|---------|
| `width()` | Set column widths manually |
| `autofit()` | Automatically adjust cell widths and heights |
| `fit_to_width()` | Fit table to maximum width |
| `set_table_properties()` | Set table layout and width properties; pass `opts_word = list(repeat_headers = FALSE)` (added in 0.9.10) to stop header repetition across Word pages |

**Example:**
```r
ft |>
  autofit() # Adjust to content

# Or manual widths
ft |>
  width(j = 1, width = 2) |>
  width(j = 2:4, width = 1)
```

## Row Height and Pagination

| Function | Purpose |
|----------|---------|
| `height()` | Set row heights |
| `height_all()` | Set heights for all rows |
| `hrule()` | Determine how row heights are calculated |
| `paginate()` | Prevent page breaks within table |
| `keep_with_next()` | Set Word "Keep with next" instructions |

**Example:**
```r
ft |>
  height(height = 0.5) |>
  hrule(rule = "exact")
```

## Get Dimensions

| Function | Purpose |
|----------|---------|
| `dim(<flextable>)` | Get column widths and row heights |
| `dim_pretty()` | Calculate optimal column widths and heights |
| `flextable_dim()` | Get overall table width and height |
| `nrow_part()` | Get number of rows in a table part |
| `ncol_keys()` | Get number of columns |

## Caption and Labels

| Function | Purpose |
|----------|---------|
| `set_caption()` | Set table caption for cross-referencing |

**Example:**
```r
ft |>
  set_caption("Table 1: Summary Statistics")
```

## Save to Files

| Function | Purpose |
|----------|---------|
| `save_as_docx()` | Export to Word (.docx) format |
| `save_as_pptx()` | Export to PowerPoint (.pptx) format |
| `save_as_html()` | Export to HTML format |
| `save_as_rtf()` | Export to RTF format |
| `save_as_image()` | Export to PNG or SVG image format |

**Example:**
```r
save_as_docx(ft, path = "table.docx")
save_as_image(ft, path = "table.png")
```

## Integrate with R Markdown and Quarto

| Function | Purpose |
|----------|---------|
| `use_flextable_qmd()` | Install flextable-qmd Quarto extension |
| `knit_print(<flextable>)` | Render flextable in knitr documents |
| `flextable_to_rmd()` | Print flextable in knitr loops |
| `df_printer()` | Enable automatic data.frame printing as flextable |
| `use_df_printer()` | Set data.frame automatic printing |
| `use_model_printer()` | Set automatic model printing as flextable |

**Example:**
```r
# In setup chunk
use_df_printer()

# Now all data.frames print as flextables
mtcars
```

## Integrate with Officer (Word/PowerPoint)

| Function | Purpose |
|----------|---------|
| `body_add_flextable()` | Insert flextable into Word document |
| `body_replace_flextable_at_bkm()` | Insert flextable at bookmark in Word |
| `ph_with(<flextable>)` | Add flextable to PowerPoint slide |
| `rtf_add(<flextable>)` | Insert flextable into RTF document |

**Example:**
```r
library(officer)

read_docx() |>
  body_add_flextable(ft) |>
  print(target = "output.docx")
```

## Other Output Formats

| Function | Purpose |
|----------|---------|
| `print(<flextable>)` | Display flextable |
| `to_html(<flextable>)` | Convert to HTML string |
| `htmltools_value()` | Convert to HTML object |
| `wrap_flextable()` | Wrap for use with patchwork |
| `plot(<flextable>)` | Plot flextable |
| `gen_grob()` | Render as graphic object |
| `plot(<flextableGrob>)` | Plot a flextable grob |
| `dim(<flextableGrob>)` | Get optimal width and height of grob |

**Example:**
```r
# As HTML
html_str <- to_html(ft)

# As plot
plot(ft)
```

## Global Defaults

| Function | Purpose |
|----------|---------|
| `set_flextable_defaults()` | Modify default formatting properties |
| `init_flextable_defaults()` | Initialize default formatting properties |
| `get_flextable_defaults()` | Retrieve current default properties |

**Example:**
```r
set_flextable_defaults(
  font.family = "Arial",
  font.size = 10,
  border.color = "gray"
)
```

## Utilities

| Function | Purpose |
|----------|---------|
| `fmt_2stats()` | Format summarizor statistics as text |
| `fmt_summarizor()` | Format summarizor statistics as text |
| `fmt_avg_dev()` | Format mean and standard deviation |
| `fmt_dbl()` | Format numbers as doubles |
| `fmt_header_n()` | Format count as '(N=XX)' for headers |
| `fmt_int()` | Format numbers as integers |
| `fmt_n_percent()` | Format count and percentage as text |
| `fmt_pct()` | Format numbers as percentages |
| `fmt_signif_after_zeros()` | Format with significant figures |
| `before()` | Detect rows before a given value |
| `void()` | Clear displayed content of columns |

**Example:**
```r
# Format as percentage
fmt_pct(0.156, digits = 1) # "15.6%"

# Mean and SD
fmt_avg_dev(5.2, 1.3, digits = 1) # "5.2 +/- 1.3"
```

## Defunct Functions

These previously deprecated functions were made defunct in flextable 0.9.9 -- calling them errors:

| Function | Replacement |
|----------|-------------|
| `as_raster()` | Use `gen_grob()` or `save_as_image()` |
| `lollipop()` | Use `linerange()` or `minibar()` |
| `set_formatter_type()` | Use `colformat_double()` / `colformat_int()` / `set_formatter()` |

Also: empty footnote ref symbol `''` is forbidden; use `add_footer_lines()` instead.

## Documentation

**Package book:** https://ardata-fr.github.io/flextable-book/
**Reference:** https://davidgohel.github.io/flextable/
**CRAN:** https://cran.r-project.org/web/packages/flextable/
