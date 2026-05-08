# mapgl API Reference

Complete function reference for mapgl. Grouped by section. Unless otherwise noted, every layer/source function accepts a piped `map` object created with `maplibre()` or `mapboxgl()` (or a proxy from `maplibre_proxy()`/`mapboxgl_proxy()`).

## Creating Maps

### `mapboxgl(style=NULL, center=c(0,0), zoom=0, bearing=0, pitch=0, projection="globe", parallels=NULL, access_token=NULL, bounds=NULL, width="100%", height=NULL, ...)`

Initialize a Mapbox GL v3 map. Requires `MAPBOX_PUBLIC_TOKEN` unless you pass `access_token` directly. Arguments:

- `style`: Mapbox style URL (`mapbox_style("standard"|"streets"|"light"|"dark"|"satellite"|"satellite-streets"|"navigation-day-v1"|"navigation-night-v1"|"outdoors"|"standard-satellite")`).
- `center`: `c(lng, lat)`.
- `zoom`, `bearing`, `pitch`: camera initial state.
- `projection`: `"mercator"`, `"globe"`, `"winkelTripel"`, `"albers"`, `"equalEarth"`, `"equirectangular"`, `"lambertConformalConic"`, `"naturalEarth"`. Use `"mercator"` when you need fill-extrusion.
- `parallels`: `c(lat1, lat2)` for albers / lambertConformalConic.
- `bounds`: sf, `st_bbox()` output, or `c(xmin, ymin, xmax, ymax)`.
- `width`, `height`: widget dims.
- `...`: extra Mapbox GL JS Map options, e.g., `scrollZoom=FALSE`, `maxZoom`, `minZoom`, `maxBounds`, `dragRotate`, `touchZoomRotate`.

### `maplibre(style=carto_style("voyager"), center=c(0,0), zoom=0, bearing=0, pitch=0, projection="globe", bounds=NULL, width="100%", height=NULL, ...)`

Initialize a MapLibre GL map. No token needed by default (CARTO tiles). Same argument semantics as `mapboxgl()` minus `access_token`/`parallels`.

### `mapboxgl_view(data, ...)` / `maplibre_view(data, column=NULL, n=5, palette=NULL, ...)`

One-call visualization from sf or terra. Auto-detects geometry type, classifies `column` if supplied, adds a legend.

### `add_view(map, data, column=NULL, ...)`

Add a `*_view`-style auto-styled layer onto an existing map.

## Layers

All layers share these arguments (listed once here, not repeated per function):

- `map`, `id` (unique string), `source` (source id string, sf object, sfc geometry vector, or a `list(type=..., data=...)`). As of 0.4.6, raw `sfc` vectors work in any layer/quickview/`bounds` slot -- wrapping with `st_sf()` is no longer needed.
- `source_layer`: required for vector tile / PMTiles sources.
- `popup`, `tooltip`: column name to render on click / hover.
- `hover_options`: named list of paint-property overrides when feature is hovered.
- `filter`: expression list filtering features.
- `before_id`: insert below a named layer.
- `visibility`: `"visible"` or `"none"`.
- `slot`: for Mapbox Standard style slots.
- `min_zoom`, `max_zoom`: layer zoom range.

### `add_fill_layer()`

Paint args: `fill_color`, `fill_opacity`, `fill_outline_color`, `fill_antialias=TRUE`, `fill_emissive_strength`, `fill_pattern`, `fill_pattern_cross_fade`, `fill_sort_key`, `fill_translate`, `fill_translate_anchor="map"`, `fill_z_offset`.

### `add_line_layer()`

`line_color`, `line_width`, `line_opacity`, `line_blur`, `line_cap` (`"butt"|"round"|"square"`), `line_dasharray`, `line_gap_width`, `line_gradient`, `line_join` (`"bevel"|"round"|"miter"`), `line_miter_limit`, `line_offset`, `line_pattern`, `line_sort_key`, `line_translate`, `line_translate_anchor="map"`, `line_trim_color`, `line_trim_offset`, `line_trim_fade_range`, `line_elevation_ground_scale`, `line_elevation_reference` (`"none"|"sea"|"ground"|"hd-road-markup"`), `line_emissive_strength`, `line_occlusion_opacity`, `line_pattern_cross_fade`, `line_round_limit`, `line_z_offset`.

### `add_circle_layer()`

`circle_color`, `circle_radius`, `circle_opacity`, `circle_blur`, `circle_emissive_strength` (3D lighting), `circle_pitch_alignment`, `circle_pitch_scale`, `circle_sort_key`, `circle_stroke_color`, `circle_stroke_opacity`, `circle_stroke_width`, `circle_translate`, `circle_translate_anchor="map"`, `cluster_options` (from `cluster_options()`).

### `add_heatmap_layer()`

`heatmap_weight`, `heatmap_intensity`, `heatmap_color`, `heatmap_radius`, `heatmap_opacity`.

### `add_symbol_layer()`

Icon: `icon_image`, `icon_size`, `icon_color`, `icon_halo_blur`, `icon_halo_color`, `icon_halo_width`, `icon_opacity`, `icon_offset`, `icon_rotate`, `icon_anchor`, `icon_allow_overlap`, `icon_ignore_placement`, `icon_optional`, `icon_padding`, `icon_keep_upright`, `icon_pitch_alignment`, `icon_rotation_alignment`, `icon_text_fit`, `icon_text_fit_padding`, `icon_translate`, `icon_translate_anchor`, `icon_image_cross_fade`, `icon_emissive_strength`, `icon_occlusion_opacity`, `icon_color_brightness_max/min/contrast/saturation`. Text: `text_field`, `text_color`, `text_size`, `text_font`, `text_anchor`, `text_halo_blur`, `text_halo_color`, `text_halo_width`, `text_letter_spacing`, `text_line_height`, `text_max_angle`, `text_max_width`, `text_offset`, `text_opacity`, `text_optional`, `text_padding`, `text_pitch_alignment`, `text_radial_offset`, `text_rotate`, `text_rotation_alignment`, `text_transform`, `text_translate`, `text_translate_anchor`, `text_variable_anchor`, `text_writing_mode`, `text_justify`, `text_keep_upright`, `text_allow_overlap`, `text_ignore_placement`, `text_emissive_strength`, `text_occlusion_opacity`. Symbol layout: `symbol_placement` (`"point"|"line"|"line-center"`), `symbol_spacing`, `symbol_sort_key`, `symbol_avoid_edges`, `symbol_z_elevate`, `symbol_z_order`, `symbol_z_offset`. Plus `cluster_options`.

### `add_fill_extrusion_layer()`

`fill_extrusion_color`, `fill_extrusion_height`, `fill_extrusion_base`, `fill_extrusion_opacity`, `fill_extrusion_pattern`, `fill_extrusion_vertical_gradient=TRUE`, `fill_extrusion_cast_shadows=TRUE`, `fill_extrusion_ambient_occlusion_intensity`, `fill_extrusion_ambient_occlusion_radius`, `fill_extrusion_emissive_strength`, `fill_extrusion_cutoff_fade_range`, `fill_extrusion_translate`, `fill_extrusion_translate_anchor="map"`. **Set `projection="mercator"` on the map.**

### `add_raster_layer()`

`raster_opacity`, `raster_color`, `raster_color_mix`, `raster_color_range`, `raster_contrast`, `raster_saturation`, `raster_hue_rotate`, `raster_brightness_max`, `raster_brightness_min`, `raster_fade_duration`, `raster_resampling` (`"linear"|"nearest"`), `raster_emissive_strength`.

### `add_layer(map, layer)`

Low-level: attach a raw style-spec layer list.

## Sources

### `add_source(map, id, data, ...)`

`data`: sf object or URL to remote GeoJSON.

### `add_vector_source(map, id, url=NULL, tiles=NULL, promote_id=NULL, ...)`

Pass either `url` (TileJSON) or `tiles` (vector of tile URL templates). `promote_id` is required for hover or `feature_click` on vector tiles.

### `add_raster_source(map, id, url=NULL, tiles=NULL, tileSize=256, maxzoom=22, ...)`

Remote raster tile source.

### `add_raster_dem_source(map, id, url, tileSize=512, maxzoom=NULL, ...)`

DEM source for `set_terrain()`.

### `add_image_source(map, id, url=NULL, data=NULL, coordinates=NULL, colors=NULL)`

Single image or terra `SpatRaster`/`RasterLayer`. `coordinates`: list of four `c(lng, lat)` in clockwise order starting top-left; auto-extracted for raster data. `colors`: vector of colors used as a color table for categorical rasters (added in 0.4.6).

### `add_video_source(map, id, urls, coordinates)`

Looping video at fixed coordinates.

### `add_pmtiles_source(map, id, url, source_type="vector", maxzoom=22, tilesize=256, promote_id=NULL, ...)`

PMTiles archive (protomaps). `source_type="raster"` for raster PMTiles (MapLibre only). MapLibre Tiles (MLT) format is also supported inside PMTiles. As of mapgl 0.4.6 / Mapbox GL JS v3.21, PMTiles works natively on Mapbox without a JS shim.

### `add_h3j_source(map, id, url)`

H3 hexagon source.

## Map controls

### `add_navigation_control(map, show_compass=TRUE, show_zoom=TRUE, visualize_pitch=FALSE, position="top-right", orientation="vertical")`

### `add_fullscreen_control(map, position="top-right")`

### `add_scale_control(map, position="bottom-left", unit="metric"|"imperial"|"nautical", max_width=100)`

### `add_geolocate_control(map, position="top-right", ...)`

Geolocation (fit to user).

### `add_layers_control(map, position="top-left", layers=NULL, collapsible=TRUE, use_icon=TRUE, background_color=NULL, active_color=NULL, hover_color=NULL, active_text_color=NULL, inactive_text_color=NULL, margin_top=NULL, margin_right=NULL, margin_bottom=NULL, margin_left=NULL)`

`layers` may be a character vector of IDs, a named list `list("Label" = "id")`, or a nested list for grouping.

### `add_draw_control(map, position="top-left", freehand=FALSE, simplify_freehand=FALSE, rectangle=FALSE, radius=FALSE, bezier=FALSE, bezier_polygon=FALSE, orientation="vertical", source=NULL, attributes=NULL, point_color="#3bb2d0", line_color="#3bb2d0", fill_color="#3bb2d0", fill_opacity=0.1, active_color="#fbb03b", vertex_radius=5, line_width=2, download_button=FALSE, download_filename="drawn-features", show_measurements=FALSE, measurement_units="both", ...)`

Drawing modes: free-form (default), `freehand`, `rectangle`, `radius` (circle), `bezier` (curved line), `bezier_polygon` (curved polygon). Drawn features land in `input$<mapId>_drawn_features`.

`attributes`: optional named list of `draw_attribute()` field definitions to attach editable properties to drawn features. Keys become property names, values are `draw_attribute()` configs.

### `draw_attribute(type=NULL, label=NULL, choices=NULL, default=NULL, required=FALSE, placeholder=NULL, min=NULL, max=NULL, step=NULL)`

Helper that builds one field definition for `add_draw_control(attributes=...)`. `type` accepts `"text"`, `"textarea"`, `"select"`, `"number"`, `"checkbox"` (aliases `"numeric"`, `"logical"`); inferred from defaults if `NULL`. `choices` is a vector or named list (names display as labels) for `"select"` fields. `min`/`max`/`step` only apply to numeric fields.

### `add_geocoder_control(map, position="top-right", placeholder="Search", collapsed=FALSE, provider="osm"|"maptiler", maptiler_api_key=NULL, ...)`

### `add_coordinates_control(map, position="bottom-right", format=c("decimal","dms"), precision=NULL, label=NULL, empty_text="Move cursor over map", wrap=TRUE)`

Compact readout of cursor position (lng/lat in WGS84). `precision` defaults to 5 for decimal, 1 for DMS seconds.

### `add_reset_control()`, `add_globe_control()`, `add_screenshot_control()`, `add_control(map, html, position)`

### `clear_controls(map_proxy)`

Remove all controls (Shiny proxy).

## Legends

### `add_legend(map, legend_title, values=NULL, colors=NULL, type=c("continuous","categorical"), patch_shape="square", position="top-left", sizes=NULL, add=FALSE, unique_id=NULL, width=NULL, layer_id=NULL, margin_top=NULL, margin_right=NULL, margin_bottom=NULL, margin_left=NULL, style=NULL, target=NULL, interactive=FALSE, filter_column=NULL, filter_values=NULL, classification=NULL, breaks=NULL, draggable=FALSE, circular_patches=FALSE)`

`patch_shape`: `"square"`, `"circle"`, `"line"`, `"hexagon"`, raw SVG, or sf object. `interactive=TRUE` + `filter_column` makes legend items filter their layer on click. `classification`/`breaks` auto-extract from `step_equal_interval()` etc.

### `add_categorical_legend(...)` / `add_continuous_legend(...)`

Thin wrappers around `add_legend()` with `type` preset.

### `legend_style(background_color=NULL, text_color=NULL, title_color=NULL, font_family=NULL, font_weight=NULL, ...)`

Styling payload for `add_legend(style=...)`.

### `clear_legend(map_proxy, legend_ids=NULL)`

## Markers

### `add_markers(map, data, color="red", rotation=0, popup=NULL, marker_id=NULL, draggable=FALSE, ...)`

`data`: `c(lng,lat)`, list of length-2 vectors, or sf POINT object.

### `clear_markers(map_proxy)`

## Static export

### `save_map(map, filename="map.png", width=900, height=500, include_legend=TRUE, hide_controls=TRUE, include_scale_bar=TRUE, basemap_color=NULL, image_scale=1, background="white", delay=NULL)`

Render widget to PNG via headless Chrome (`chromote`). Captures legends, attribution, optional scale bar. `delay` (seconds) lets tiles load before snapshotting.

### `print_map(map, ...)`

In-session preview wrapper around `save_map()`; opens PNG in the RStudio viewer. Same render args.

## Style helpers

### `mapbox_style(style_name)`

`"standard"`, `"standard-satellite"`, `"streets"`, `"outdoors"`, `"light"`, `"dark"`, `"satellite"`, `"satellite-streets"`, `"navigation-day-v1"`, `"navigation-night-v1"`.

### `maptiler_style(style_name)`

Needs `MAPTILER_API_KEY`. `"basic"`, `"bright"`, `"dataviz"`, `"backdrop"`, `"basic-v2"`, `"streets"`, `"outdoor"`, `"satellite"`, `"topo"`, `"toner"`, `"hybrid"`.

### `carto_style(style_name)`

`"voyager"`, `"positron"`, `"dark-matter"`.

### `openfreemap_style(style_name)`

`"bright"`, `"positron"`, `"liberty"`, `"dark"`, `"fiord"`.

### `esri_style(style_name, api_key=NULL)` / `esri_open_style(style_name)`

Esri basemaps; `esri_style()` needs a key.

### `basemap_style()`

Blank style for building from scratch.

## Data-driven styling expressions

### `interpolate(column=NULL, property=NULL, type="linear", values, stops, na_color=NULL)`

`type` can be `"linear"`, `list("exponential", base)`, or `list("cubic-bezier", x1, y1, x2, y2)`. `values` are numeric breakpoints; `stops` are output values (colors, sizes). `property="zoom"` to vary by zoom.

### `match_expr(column=NULL, property=NULL, values, stops, default="#cccccc")`

Categorical.

### `step_expr(column=NULL, property=NULL, base, values, stops, na_color=NULL)`

Stepwise classes. `base` is the value before the first threshold.

### `step_equal_interval(column, n=5, palette=viridisLite::viridis, na_color=NULL)` / `step_quantile(...)` / `step_jenks(...)`

Auto-classified step expressions. Pair with:

- `get_legend_labels(classification, digits=2, format="both"|"left"|"right")`
- `get_legend_colors(classification)`
- `get_breaks(classification)`

to feed `add_legend()`.

### `interpolate_palette(column, method="equal_interval"|"quantile"|"jenks", n=5, palette=viridisLite::viridis, na_color=NULL)`

Auto-built `interpolate()` with breaks + palette.

### `get_column(column)`

Reference a source column inside any expression. Required when mixing columns with literals in nested expressions.

### `concat(..., separator="")` / `number_format(x, locale=NULL, style=NULL, currency=NULL, min_fraction_digits=NULL, max_fraction_digits=NULL)`

Compose popup/tooltip strings.

### `cluster_options(cluster_max_zoom=14, cluster_radius=50, ...)`

Config payload for `add_circle_layer(cluster_options=...)`.

### `palette_to_lut(palette, n=256)`

Convert an R palette to a mapgl LUT (for `raster_color` etc.).

## Camera & view

### `fit_bounds(map, bbox, animate=FALSE, ...)`

`bbox`: sf object, `st_bbox()`, or `c(xmin, ymin, xmax, ymax)`. Extra `...` are Mapbox/MapLibre camera options (e.g. `duration`, `padding`, `pitch`, `bearing`, `maxZoom`).

### `fly_to(map, center, zoom=NULL, ...)` / `ease_to(map, center, zoom=NULL, ...)` / `jump_to(map, center, zoom=NULL, ...)`

`...` passes through: `bearing`, `pitch`, `speed`, `curve`, `duration`, etc.

### `set_view(map, center, zoom)`

Set center/zoom without animation.

## Map configuration

### `set_style(map, style, config=NULL, diff=TRUE, preserve_layers=TRUE)`

Switch basemap; `preserve_layers=TRUE` keeps user layers/sources.

### `set_projection(map_proxy, projection)` / `set_terrain(map, source, exaggeration=1)` / `set_fog(map, ...)` / `set_rain(map, ...)` / `set_snow(map, ...)` / `set_config_property(map, import_id, property, value)`

## Layer management (proxy + pipeline)

### `set_filter(map, layer_id, filter)`

`filter`: Mapbox expression list, e.g. `list(">=", get_column("pop"), 1000)` or `list("==", "cat", "A")`. Use `NULL` to remove.

### `set_paint_property(map, layer_id, name, value)`

e.g. `set_paint_property("ct", "fill-color", "#ff0000")`.

### `set_layout_property(map, layer_id, name, value)`

e.g. `set_layout_property("ct", "visibility", "none")`.

### `set_tooltip(map, layer_id, tooltip)` / `set_popup(map, layer_id, popup)`

Swap tooltip/popup column without re-creating the layer.

### `set_source(map, layer_id, source)`

Swap a layer's source.

### `clear_layer(map_proxy, layer_id)`

Remove a layer.

### `move_layer(map_proxy, layer_id, before_id=NULL)`

Change z-order.

## Shiny integration

### `mapboxglOutput(outputId, width="100%", height="400px")` / `maplibreOutput(...)`

Output placeholder.

### `renderMapboxgl(expr, env=parent.frame(), quoted=FALSE)` / `renderMaplibre(...)`

Reactive render.

### `mapboxgl_proxy(mapId, session=shiny::getDefaultReactiveDomain())` / `maplibre_proxy(...)`

Proxy for mutation. Use inside `observeEvent()` / `observe()`. Most `set_*`, `add_*_layer`, `fly_to`, `fit_bounds`, `clear_*`, `add_markers` work on proxies.

### Compare widget

`mapboxglCompareOutput()`, `maplibreCompareOutput()`, `renderMapboxglCompare()`, `renderMaplibreCompare()`, `mapboxgl_compare_proxy(mapId, map_side="before"|"after")`, `maplibre_compare_proxy(...)`.

### `compare(map1, map2, width="100%", height=NULL, elementId=NULL, mousemove=FALSE, orientation="vertical", mode="swipe"|"sync", swiper_color=NULL)`

Non-Shiny compare widget.

### `enable_shiny_hover(map, coordinates=TRUE, features=TRUE, layer_id=NULL)`

Enable hover-related reactive inputs. `coordinates=TRUE` exposes `input$<mapId>_hover` (cursor lng/lat); `features=TRUE` exposes `input$<mapId>_feature_hover` (feature under cursor). Pass a layer id (or vector) to `layer_id` to restrict feature hover to specific layers.

### Auto-exposed Shiny inputs

From any `maplibreOutput()`/`mapboxglOutput()` with `outputId = "map"`:

- `input$map_center` -- list(lng, lat)
- `input$map_zoom` -- numeric
- `input$map_bbox` -- list(xmin, xmax, ymin, ymax)
- `input$map_click` -- list(lng, lat, time)
- `input$map_feature_click` -- list(id, properties, lng, lat)
- `input$map_feature_hover` -- same (requires `enable_shiny_hover()`)
- `input$map_drawn_features` -- GeoJSON FeatureCollection (with `add_draw_control()`)
- `input$map_view_state` -- camera snapshot

## Turf.js client-side spatial ops

All accept a `layer_id` (existing map layer), `data` (sf), or raw coords; write to `source_id`. In Shiny, `input_id` also publishes results to `input$<mapId>_<input_id>`.

- `turf_buffer(map, layer_id|data, radius, units="kilometers"|"miles"|"meters"|"degrees"|"radians", source_id, input_id=NULL)`
- `turf_filter(map, layer_id, filter_layer_id|filter_data, predicate="intersects"|"within"|"contains"|"crosses"|"disjoint", source_id, input_id=NULL)`
- `turf_union(map, layer_id_1, layer_id_2, source_id)`
- `turf_intersect(map, layer_id, layer_id_2, source_id)`
- `turf_difference(map, layer_id, layer_id_2, source_id)`
- `turf_convex_hull(map, layer_id|data, source_id)`
- `turf_concave_hull(map, layer_id|data, max_edge=Infinity, source_id)`
- `turf_voronoi(map, layer_id|data, bbox=NULL, source_id)`
- `turf_centroid(map, layer_id|data, source_id)`
- `turf_center_of_mass(map, layer_id|data, source_id)`
- `turf_distance(map, point1, point2, units="kilometers", input_id)` (Shiny only)
- `turf_area(map, layer_id|data, units="meters", input_id)` (Shiny only)

## Drawing & feature query

- `get_drawn_features(map)`, `add_features_to_draw(map, data)`, `clear_drawn_features(map_proxy)`.
- `query_rendered_features(map_proxy, geometry=NULL, layers=NULL, filter=NULL, input_id)`, `get_queried_features(map_proxy, input_id)` -- return features as sf via `input$<mapId>_<input_id>`.
- `add_image(map, id, url|data, sdf=FALSE, pixelRatio=1)` -- register a sprite for `icon_image`.
- `add_globe_minimap(map, position="bottom-right", ...)`.

## Story maps

### `story_map(map_id, sections, font_family=NULL, background_color=NULL, text_color=NULL, theme="light"|"dark", show_credits=TRUE, ...)` / `story_maplibre(...)` / `story_leaflet(...)`

UI builder. `sections` is a named list of `story_section()` objects; names are section IDs used by `on_section()`.

### `story_section(title, content, position="left"|"center"|"right", background_color=NULL, text_color=NULL, ...)`

One scrolling section. `content` is Shiny UI (tags, inputs, outputs). `title=NULL` or `""` renders without a title.

### `on_section(map_id, section_id, expr)`

Server-side trigger run whenever the named section becomes active.
