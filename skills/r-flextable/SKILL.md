---
name: r-flextable
description: Use when code loads or uses flextable (library(flextable), flextable::), creating formatted tables for Word (.docx), PowerPoint (.pptx), HTML, or PDF in R, or needing merged cells and conditional formatting
---

# R flextable

## Overview

**flextable creates publication-quality tables that render consistently across Word, PowerPoint, HTML, and PDF.**

Core principle: Separate data from display. Build from a data.frame, then layer formatting declaratively using selectors.

## References

Read `references/API.md` before writing code.

- `references/API.md` - Complete function reference
- `references/overview.md` - Core concepts and workflow
- `references/formatting.md` - Styling and appearance
- `references/layout.md` - Structure and merging
- `references/selectors.md` - Targeting cells for formatting
- `references/output.md` - Rendering to different formats

## When to Use

**Use flextable when:**
- Output to Word/PowerPoint required
- Need cell merging, conditional formatting, mini charts
- Cross-references and auto-numbered captions
- Same table across multiple formats

## When NOT to Use

- HTML-only output: prefer `gt` or `reactable`
- LaTeX-heavy / academic PDF tables: prefer `kableExtra`
- Interactive dashboards (sortable, filterable in browser): prefer `DT` or `reactable`
- Aggregation or reshaping: flextable does not aggregate -- pre-aggregate
  with `dplyr` / `collapse` / `tabulator()` / `proc_freq()` first

## Quick Reference

| Task | Function |
|------|----------|
| Create table | `flextable(df)` |
| Format numbers | `colformat_double()`, `colformat_int()` |
| Bold/italic/color | `bold()`, `italic()`, `color()` |
| Strikethrough chunk | `as_strike()` |
| Background color | `bg()` |
| Merge cells | `merge_v()`, `merge_h()`, `merge_at()` |
| Add header | `add_header_row()` |
| Auto-size | `autofit()` |
| Apply theme | `theme_vanilla()`, `theme_zebra()`, `theme_borderless()` |
| Quarto-rich cell content | `as_qmd()` + `use_flextable_qmd()` |
| Combine with ggplot2 | `wrap_flextable()` (patchwork) |
| Save to Word | `save_as_docx()` |
| Save to PowerPoint | `save_as_pptx()` |

## Core Pattern

```r
library(flextable)

# Create -> format -> output
ft <- flextable(head(mtcars, 5)) |>
  colformat_double(digits = 1) |>
  # Conditional formatting with formula selectors
  color(i = ~ mpg > 20, j = "mpg", color = "darkgreen") |>
  bold(i = ~ mpg > 20, j = "mpg") |>
  # Spanning header
  add_header_row(
    values = c("Performance", "Engine", "Transmission"),
    colwidths = c(2, 4, 5)
  ) |>
  theme_vanilla() |>
  autofit()

save_as_docx(ft, path = "table.docx")
```

## Selectors (Key Concept)

Target cells for formatting:

```r
# Row selector (formula)
bold(ft, i = ~ cyl == 6)

# Column selector
bg(ft, j = ~ mpg + hp, bg = "lightgray")

# Part selector
bold(ft, part = "header")
```

See `references/selectors.md` for details on multi-content cells, mini charts, output-specific notes.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Theme doesn't format new header | Apply theme AFTER `add_header_row()` |
| `autofit()` overflows margins | Use `set_table_properties(layout = "autofit")` |
| `height()` has no effect | Set `hrule(rule = "exact")` first |
| Formula selectors fail in header | Use integer indices for header rows |
| Images missing in Word | Use `officedown::rdocx_document()` |
| PDF ignores padding | Use `ft.tabcolsep` chunk option |
| `add_header_row` values wrong | One value per span; colwidths sum to ncol |
| Header repeats on every Word page | `set_table_properties(opts_word = list(repeat_headers = FALSE))` (>= 0.9.10) |
| Empty footnote symbol `""` errors | Use `add_footer_lines()` instead (defunct since 0.9.9) |
| `lollipop()`, `as_raster()`, `set_formatter_type()` error | Defunct since 0.9.9 -- use `linerange()`/`minibar()`, `gen_grob()`, `colformat_*()` |

## Advanced

See `references/` for:
- **API.md**: Complete function reference (240+ functions)
- **selectors.md**: Multi-content cells, mini charts, output-specific notes
- **formatting.md**: All formatting functions and options
- **layout.md**: Merging, sizing, headers/footers
- **output.md**: Rendering to all formats
