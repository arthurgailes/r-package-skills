---
name: r-freestiler
description: Use when code loads or uses freestiler, working with .pmtiles files, creating or serving PMTiles vector tilesets in R, or preparing spatial data for mapgl/MapLibre visualization
---

# freestiler: Vector Tile Generator

## Overview

**freestiler converts spatial data into PMTiles vector tilesets.** Rust-powered tool for creating `.pmtiles` from sf objects, files, or DuckDB SQL.

## References

Read `references/API.md` before writing code.

- `references/API.md` - Complete function reference with all parameters
- `references/getting-started.md` - Usage patterns and parameter examples
- `references/zoom-strategy.md` - Zoom level guidance by geometry type
- `references/workflows.md` - Complete workflows and integration patterns
- `references/mapping.md` - Visualization and server configuration
- `references/maplibre-tiles.md` - MLT vs MVT tradeoffs (when to switch formats)
- `references/python.md` - Cross-language tips (PMTiles serving, format choice)

## When to Use

**Use when:**

- Dataset >10k features (too large for direct mapgl)
- Need static hosting
- Dataset exceeds memory

**Don't use:**

- <10k features - pass sf to mapgl directly
- Quick exploration - use `maplibre_view()`
- Existing tile pipeline works

## Quick Reference

| Function            | Use Case                  | Key Params                                                                                                         |
| ------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `freestile()`       | Tile sf objects           | `input`, `output`, `layer_name`, `min_zoom`, `max_zoom`, `base_zoom` (opt), `drop_rate` (opt), `tile_format` (opt) |
| `freestile_query()` | Tile from SQL (streaming) | `query`, `output`, `layer_name`, `min_zoom`, `max_zoom`, `streaming = "always"`                                    |
| `freestile_file()`  | Tile without loading      | `input`, `output`, `layer_name`, `min_zoom`, `max_zoom`, `engine`                                                  |
| `view_tiles()`      | Quick visualization       | `input`, `layer_type`, `color`                                                                                     |
| `serve_tiles()`     | Start local server        | `path`, `port`                                                                                                     |

## Quick Start

```r
library(freestiler)
library(sf)

# Basic workflow - counties use max_zoom = 10
nc <- st_read(system.file("shape/nc.shp", package = "sf"))
freestile(input = nc, output = "nc.pmtiles", layer_name = "counties", max_zoom = 10)
view_tiles("nc.pmtiles")  # Auto-starts server + opens map

# Large datasets: streaming
freestile_query(query = "SELECT * FROM read_parquet('huge.parquet')",
                output = "points.pmtiles", layer_name = "locations",
                streaming = "always")  # Critical for 10M+ points

# Direct file (no loading to memory)
freestile_file(input = "data.gpkg", output = "output.pmtiles",
               layer_name = "features")
```

## Visualization

**view_tiles() (Easiest)**: `view_tiles("nc.pmtiles")`

**Manual Server + mapgl**:

```r
serve_tiles("tiles/")
maplibre() |>
  add_pmtiles_source(id = "src", url = "http://localhost:8080/data.pmtiles") |>
  add_fill_layer(source = "src", source_layer = "layer1")
```

**Direct file://**: Requires absolute path in `add_pmtiles_source(url = "file://W:/...")`

## Common Mistakes

1. **Parameter names:** Use `input`, `output`, `layer_name` (NOT `data`, `tileset`, `layer`)
2. **Source layer must match:** `layer_name` in `freestile()` must match `source_layer` in mapgl
3. **CRS must be WGS84:** Use `st_transform(4326)` before tiling
4. **Large sf objects:** Use `freestile_file()` or `freestile_query()` to avoid loading into memory
5. **Massive points (10M+):** Add `streaming = "always"` to `freestile_query()`
6. **Tile format:** MVT is the default (broadest compatibility). Pass `tile_format = "mlt"` for smaller polygon/line files when targeting MapLibre GL JS 5.21+ or mapgl. Do not assume MLT is the default.
7. **Large tilesets (>500MB):** `serve_tiles()` struggles with byte-range requests; launch `npx http-server` from R via `processx::process$new()` instead. See `references/mapping.md` "Serving Large Tilesets" for a ready-made helper function.

## When NOT to Use

- Small datasets (<10k features) - pass sf directly to mapgl
- Quick exploration - use `maplibre_view()` from mapgl
- Existing tile pipeline already works

## Advanced

See `references/` for:

- **API.md**: Complete function reference
- **getting-started.md**, **mapping.md**: CRAN vignettes
- Workflows, zoom strategy, SQL patterns, performance tips

**Test with validator:** Use `lib/r-validators/plot-validator.R` to verify map output in tests

**Resources:** [Docs](https://walker-data.com/freestiler/) | [GitHub](https://github.com/walkerke/freestiler/) | See `r-mapgl` skill
