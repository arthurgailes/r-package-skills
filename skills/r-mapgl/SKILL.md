---
name: r-mapgl
description: Use when code loads or uses mapgl (library(mapgl), mapgl::), calls maplibre()/mapboxgl()/add_*_layer()/maplibre_proxy(), builds interactive WebGL maps in R, displays PMTiles on a map, or creates story maps / scrollytelling in R Shiny
---

# mapgl: Interactive WebGL Maps in R

## Overview

**mapgl wraps Mapbox GL JS and MapLibre GL JS for interactive WebGL maps in R.** Data goes on a map as a **source** and is drawn by a **layer**. Typical pipe: `maplibre()` or `mapboxgl()` -> `add_*_layer()`; style with `interpolate()` / `match_expr()` / `step_expr()`; update reactively in Shiny with `maplibre_proxy()`.

## References

Read `references/API.md` before writing code.

- `references/API.md` -- Complete function reference (every `add_*`, `set_*`, `turf_*`, expression helper, legend, Shiny output).
- `references/getting-started.md` -- Vignette: Mapbox vs MapLibre, styles, tokens, `*_view()`.
- `references/layers-overview.md` -- Vignette: every layer type with full worked examples.
- `references/shiny.md` -- Vignette: proxies, reactive map inputs, compare widgets.
- `references/story-maps.md` -- Vignette: scrollytelling with `story_map()` / `on_section()`.
- `references/turf.md` -- Vignette: client-side spatial ops (buffer, filter, intersect).

## When NOT to Use

- Simple marker maps with <1k points -- `leaflet` is lighter.
- Static print maps -- `tmap` or `ggplot2 + sf`.
- Non-spatial plots -- `ggplot2`.
- Datasets >100k features -- tile first with `freestiler`, then `add_pmtiles_source()`.

## API Keys

- `maplibre()` + `carto_style()` / `openfreemap_style()` -- **no key**. Default style is `carto_style("voyager")`.
- `mapboxgl()` / `mapbox_style()` -- set `MAPBOX_PUBLIC_TOKEN` in `.Renviron`.
- `maptiler_style()` -- set `MAPTILER_API_KEY`.

## Quick Reference

### Create a map

| Function | Purpose | Key parameters |
|---|---|---|
| `maplibre()` | MapLibre map | `style` (default `carto_style("voyager")`), `center`, `zoom`, `bearing`, `pitch`, `bounds`, `projection`, `...` (e.g. `scrollZoom=FALSE`, `maxZoom`) |
| `mapboxgl()` | Mapbox map (needs token) | same args + `access_token` (opt), `parallels` (adv) |
| `maplibre_view()` / `mapboxgl_view()` | Auto-styled view | `data` (sf/terra), `column` (opt), `n` (opt) |

### Layers

All take `map, id, source, source_layer` (opt, for vector/PMTiles), `popup` (opt), `tooltip` (opt), `hover_options` (opt), `filter` (opt), `before_id` (opt), `min_zoom`/`max_zoom` (opt).

| Function | Most-used style params |
|---|---|
| `add_fill_layer()` | `fill_color`, `fill_opacity`, `fill_outline_color` |
| `add_line_layer()` | `line_color`, `line_width`, `line_opacity`, `line_dasharray` (opt) |
| `add_circle_layer()` | `circle_color`, `circle_radius`, `circle_stroke_color`, `circle_stroke_width`, `cluster_options` (opt) |
| `add_heatmap_layer()` | `heatmap_weight`, `heatmap_intensity`, `heatmap_color`, `heatmap_radius` |
| `add_symbol_layer()` | `icon_image`, `icon_size`, `text_field`, `text_size`, `text_color` |
| `add_fill_extrusion_layer()` | `fill_extrusion_color`, `fill_extrusion_height`, `fill_extrusion_base`. **Use `projection="mercator"` on the map** -- globe has artifacts. |
| `add_raster_layer()` | `raster_opacity`, `raster_color` (opt) |

### Sources

| Function | Use |
|---|---|
| `add_source(id, data)` | sf object or GeoJSON URL |
| `add_vector_source(id, url|tiles, promote_id=NULL)` | Remote vector tiles |
| `add_pmtiles_source(id, url, source_type="vector", maxzoom=22, promote_id=NULL)` | PMTiles archive |
| `add_raster_source(id, url|tiles, tileSize=256, maxzoom=22)` | Remote raster tiles |
| `add_raster_dem_source(id, url, tileSize=512)` | DEM for `set_terrain()` |
| `add_image_source(id, url|data, coordinates)` | Single image or terra raster |

`promote_id` is **required** on vector / PMTiles sources that need hover or `feature_click`.

### Styling expressions (data-driven styling)

| Function | Use |
|---|---|
| `interpolate(column, values, stops, type="linear", na_color=NULL)` | Continuous color/size |
| `match_expr(column, values, stops, default="#cccccc")` | Categorical |
| `step_expr(column, base, values, stops)` | Threshold classes |
| `step_equal_interval()` / `step_quantile()` / `step_jenks()` | Auto classes + legend helpers (`get_legend_labels`, `get_legend_colors`, `get_breaks`) |
| `interpolate_palette()` | Auto palette + breaks |
| `get_column("name")` | Data-column reference inside any expression |

### Camera, controls, legends

- Camera: `fit_bounds(map, bbox, animate=FALSE)`, `fly_to(map, center, zoom)`, `ease_to()`, `jump_to()`, `set_view()`.
- Controls: `add_navigation_control()`, `add_fullscreen_control()`, `add_scale_control()`, `add_geolocate_control()`, `add_globe_control()`, `add_layers_control(layers=NULL, collapsible=TRUE)`, `add_draw_control(rectangle=FALSE, radius=FALSE, bezier=FALSE, attributes=NULL, show_measurements=FALSE)`, `add_geocoder_control(provider="osm"|"maptiler")`, `add_coordinates_control(format="decimal"|"dms")`, `add_screenshot_control()`, `add_reset_control()`.
- Legends: `add_legend(legend_title, values, colors, type=c("continuous","categorical"), patch_shape="square", position="top-left", add=FALSE, layer_id=NULL, interactive=FALSE, draggable=FALSE)`. Also `add_categorical_legend()`, `add_continuous_legend()`, `clear_legend()`. Interactive legends filter their layer on click (categorical) or via drag handles (continuous).

### Shiny

- Output / render: `maplibreOutput("map", height="600px")` + `renderMaplibre({...})`. Same for `mapboxgl`.
- Proxy (mutate without re-render): `maplibre_proxy("map")` / `mapboxgl_proxy("map")` inside `observeEvent()`.
- Proxy-compatible: `set_filter()`, `set_paint_property()`, `set_layout_property()`, `set_style()`, `clear_layer()`, `add_*_layer()`, `fly_to()`, `fit_bounds()`, `add_markers()`.
- Auto inputs: `input$<mapId>_click`, `..._feature_click`, `..._zoom`, `..._center`, `..._bbox`, `..._drawn_features`.
- Compare widget: `compare(m1, m2, mode="swipe"|"sync")`; Shiny: `maplibreCompareOutput()` + `maplibre_compare_proxy(map_side="before"|"after")`.

### Story maps & turf.js

- Story: `story_map()` / `story_maplibre()` / `story_leaflet()` UI + `story_section(title, content, position="left")` + server `on_section(map_id, section_id, expr)`.
- Client-side spatial: `turf_buffer()`, `turf_filter(predicate="intersects"|"within"|"contains"|"crosses"|"disjoint")`, `turf_intersect()`, `turf_union()`, `turf_difference()`, `turf_convex_hull()`, `turf_concave_hull()`, `turf_voronoi()`, `turf_centroid()`.

### Static export

- `save_map(map, filename, width=900, height=500, include_legend=TRUE, hide_controls=TRUE, include_scale_bar=TRUE, image_scale=1, delay=NULL)` renders a PNG via headless Chrome (needs `chromote`). Use `delay` (seconds) to let tiles finish loading. `print_map()` previews in the RStudio viewer.

## Quick Start

```r
library(mapgl); library(sf)
nc <- st_read(system.file("shape/nc.shp", package = "sf"))

maplibre(style = carto_style("positron"), bounds = nc) |>
  add_fill_layer(
    id = "counties",
    source = nc,
    fill_color = interpolate(
      column = "BIR74",
      values = c(500, 20000),
      stops = c("#eff3ff", "#08519c"),
      na_color = "lightgrey"
    ),
    fill_opacity = 0.7,
    popup = "NAME",
    hover_options = list(fill_color = "yellow")
  ) |>
  add_legend("Births (1974)",
             values = c(500, 20000),
             colors = c("#eff3ff", "#08519c"))
```

## Shiny pattern (proxy, not re-render)

```r
library(shiny); library(mapgl)
ui <- fluidPage(sliderInput("min", "Min BIR74", 0, 22000, 500),
                maplibreOutput("map", height = "600px"))
server <- function(input, output, session) {
  output$map <- renderMaplibre({
    maplibre(carto_style("positron"), bounds = nc) |>
      add_fill_layer(id = "ct", source = nc, fill_color = "steelblue")
  })
  observeEvent(input$min, {
    maplibre_proxy("map") |>
      set_filter("ct", list(">=", get_column("BIR74"), input$min))
  })
}
```

## PMTiles pattern (large data)

PMTiles is supported on both `maplibre()` and `mapboxgl()` (native in Mapbox GL JS v3.21+, no JS shim required).

```r
maplibre() |>
  add_pmtiles_source(id = "src", url = "https://example.com/data.pmtiles") |>
  add_fill_layer(id = "pm", source = "src",
                 source_layer = "features",  # must match freestiler layer_name
                 fill_color = "steelblue")
```

## Common Mistakes

| Mistake | Fix |
|---|---|
| Using `%>%` | Use native `\|>`. |
| Hard-coding colors for numeric data | `interpolate()` + matching `add_legend()`. |
| `source_layer` omitted for vector/PMTiles | Required. Must match the tileset layer name. |
| Re-rendering map on every input | Use `maplibre_proxy()` + `set_*` in `observeEvent()`. |
| Missing `promote_id` for hover/feature_click | Set `promote_id` on `add_vector_source()` / `add_pmtiles_source()`. |
| Fill-extrusion + globe projection | Set `projection = "mercator"`. |
| `mapboxgl()` with no token | Use `maplibre()` for token-free maps. |
| `circular_patches = TRUE` | Deprecated; use `patch_shape = "circle"`. |
| Wrapping `sfc` in `st_sf()` just to plot | As of 0.4.6, `add_*_layer()`, `*_view()`, and `bounds=` accept raw `sfc` vectors. |
| Forgetting `delay` in `save_map()` for remote tiles | Pass `delay = 2` (or higher) so tiles finish loading before the snapshot. |

## Resources

[Package site](https://walker-data.com/mapgl/) | [GitHub](https://github.com/walkerke/mapgl) | Sibling skills `r-freestiler` (tiles) and `r-mapping` (chooser).
