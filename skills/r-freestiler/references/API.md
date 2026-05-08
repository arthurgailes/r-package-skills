# freestiler Function Reference

Complete API documentation for freestiler package functions.

---

## freestile()

Create PMTiles archives from spatial data. Handles both Mapbox Vector Tile (MVT) and MapLibre Tile (MLT) formats with support for multi-layer output, feature thinning, point clustering, and feature merging.

### Signature

```r
freestile(
  input,
  output,
  layer_name = NULL,
  tile_format = "mvt",
  min_zoom = 0L,
  max_zoom = 14L,
  base_zoom = NULL,
  drop_rate = NULL,
  cluster_distance = NULL,
  cluster_maxzoom = NULL,
  coalesce = FALSE,
  simplification = TRUE,
  generate_ids = TRUE,
  overwrite = TRUE,
  quiet = FALSE
)
```

### Parameters

**input**

- An sf data frame OR
- A named list of sf/freestile_layer objects for multi-layer configurations

**output**

- File path for the resulting `.pmtiles` archive

**layer_name**

- Optional layer identifier
- Automatically derived from output filename if not specified

**tile_format**

- Encoding choice: `"mvt"` (default, broadest compatibility) or `"mlt"` (smaller files for polygon/line data; requires MapLibre GL JS 5.21+)

**min_zoom** / **max_zoom**

- Zoom level boundaries (default 0-14)

**base_zoom**

- Reference zoom where all features appear
- Controls feature thinning curves below this level
- Defaults to `max_zoom` if not specified

**drop_rate**

- Exponential thinning parameter for progressive feature reduction at lower zoom levels
- `NULL` disables thinning

**cluster_distance**

- Pixel radius for merging nearby points into clusters
- `NULL` disables clustering

**cluster_maxzoom**

- Zoom threshold above which individual points display instead of clusters
- Defaults to `max_zoom - 1`

**coalesce**

- Logical: enables merging identical features within each tile (default `FALSE`)

**simplification**

- Logical: applies zoom-adaptive geometry snapping to tile grids (default `TRUE`)

**generate_ids**

- Logical: assigns sequential identifiers to features (default `TRUE`)

**overwrite**

- Logical: controls whether existing files are replaced (default `TRUE`)

**quiet**

- Logical: suppresses progress notifications (default `FALSE`)

### Return Value

Invisibly returns the output file path.

### Examples

```r
# Basic usage
freestile(
  input = nc,
  output = "nc.pmtiles",
  layer_name = "counties",
  min_zoom = 4,
  max_zoom = 12
)

# Multi-layer
freestile(
  input = list(
    states = freestile_layer(states, min_zoom = 0, max_zoom = 6),
    counties = freestile_layer(counties, min_zoom = 4, max_zoom = 10)
  ),
  output = "multilayer.pmtiles"
)
```

---

## freestile_file()

Create vector tiles from a spatial file without loading into R. Supports GeoParquet, GeoPackage, Shapefile, and other formats.

### Signature

```r
freestile_file(
  input,
  output,
  layer_name = NULL,
  tile_format = "mvt",
  min_zoom = 0L,
  max_zoom = 14L,
  base_zoom = NULL,
  drop_rate = NULL,
  cluster_distance = NULL,
  cluster_maxzoom = NULL,
  coalesce = FALSE,
  simplification = TRUE,
  overwrite = TRUE,
  quiet = FALSE,
  engine = "geoparquet"
)
```

### Parameters

**input**

- File path pointing to a spatial dataset
- Supports: GeoParquet, GeoPackage, Shapefile, other formats

**output**

- Destination path for the resulting `.pmtiles` file

**layer_name**

- Optional identifier for the tile layer
- Defaults to filename-derived value

**tile_format**

- Output format: `"mvt"` (default) or `"mlt"`

**min_zoom** / **max_zoom**

- Zoom level boundaries (0-14 by default)

**base_zoom**

- Threshold above which all features display completely
- Uses `max_zoom` if unspecified

**drop_rate**

- Optional numeric parameter controlling exponential feature reduction

**cluster_distance**

- Pixel-based threshold for feature grouping

**cluster_maxzoom**

- Upper zoom cutoff for clustering operations

**coalesce**

- Boolean toggle for merging identically-attributed features

**simplification**

- Geometry snapping to tile grid (enabled by default)

**overwrite**

- Permission to replace existing output files

**quiet**

- Suppresses status messaging during processing

**engine**

- Processing backend: `"geoparquet"` or `"duckdb"` depending on input format

### Return Value

The output file path, returned invisibly.

### Examples

```r
# GeoParquet file
freestile_file("data.parquet", "output.pmtiles")

# GeoPackage with DuckDB engine
freestile_file("data.gpkg", "output.pmtiles", engine = "duckdb")
```

---

## freestile_query()

Create vector tiles from a DuckDB SQL query. Enables filtering, joining, and aggregation before tiling. Supports streaming mode for massive datasets.

### Signature

```r
freestile_query(
  query,
  output,
  db_path = NULL,
  layer_name = NULL,
  tile_format = "mvt",
  min_zoom = 0L,
  max_zoom = 14L,
  base_zoom = NULL,
  drop_rate = NULL,
  cluster_distance = NULL,
  cluster_maxzoom = NULL,
  coalesce = FALSE,
  simplification = TRUE,
  overwrite = TRUE,
  quiet = FALSE,
  source_crs = NULL,
  streaming = "auto"
)
```

### Parameters

**query**

- A SQL query that returns a geometry column
- Supports DuckDB spatial functions

**output**

- Path for the output `.pmtiles` file

**db_path**

- Optional path to DuckDB database file
- `NULL` uses in-memory database

**layer_name**

- Tile layer name
- Auto-derived from output filename if `NULL`

**tile_format**

- Output format: `"mvt"` (default) or `"mlt"`

**min_zoom** / **max_zoom**

- Zoom level boundaries (default 0-14)

**base_zoom**

- Zoom level ensuring all features present
- Defaults to `max_zoom`

**drop_rate**

- Exponential drop rate
- `NULL` disables

**cluster_distance**

- Pixel distance for clustering
- `NULL` disables

**cluster_maxzoom**

- Maximum zoom for clustering (default `max_zoom - 1`)

**coalesce**

- Merge identical features (default `FALSE`)

**simplification**

- Snap geometries to tile grid (default `TRUE`)

**overwrite**

- Overwrite existing output (default `TRUE`)

**quiet**

- Suppress progress messages (default `FALSE`)

**source_crs**

- CRS specification (e.g., `"EPSG:4326"`) for R fallback only
- **Important:** When using R DuckDB fallback, explicitly provide `source_crs` for proper interpretation or reprojection to WGS84

**streaming**

- Query execution mode: `"auto"`, `"always"`, or `"never"`
- Use `"always"` for datasets >10M features

### Return Value

The output file path (invisibly).

### Details

Use `freestile_file()` with `engine = "duckdb"` for file inputs with embedded CRS information.

### Examples

```r
# Filter GeoParquet
freestile_query(
  query = "SELECT * FROM read_parquet('data.parquet') WHERE value > 0",
  output = "filtered.pmtiles",
  layer_name = "features"
)

# Aggregate with spatial join
freestile_query(
  query = "
    SELECT
      g.geom,
      COUNT(p.id) as point_count
    FROM read_parquet('grid.parquet') g
    LEFT JOIN read_parquet('points.parquet') p
      ON ST_Within(p.geom, g.geom)
    GROUP BY g.geom
  ",
  output = "aggregated.pmtiles",
  layer_name = "grid"
)

# Massive dataset with streaming
freestile_query(
  query = "SELECT * FROM read_parquet('huge.parquet')",
  output = "massive.pmtiles",
  layer_name = "points",
  streaming = "always",  # Critical for 10M+ points
  min_zoom = 0,
  max_zoom = 14
)
```

---

## freestile_layer()

Wrap an sf object with optional per-layer zoom range overrides for use in multi-layer tile generation.

### Signature

```r
freestile_layer(input, min_zoom = NULL, max_zoom = NULL)
```

### Parameters

**input**

- An sf data frame to be wrapped as a layer

**min_zoom**

- Integer specifying the minimum zoom level for this layer
- When `NULL`, inherits the global minimum from `freestile()`

**max_zoom**

- Integer specifying the maximum zoom level for this layer
- When `NULL`, inherits the global maximum from `freestile()`

### Return Value

Returns a freestile_layer object structured as a list with a class attribute.

### Examples

```r
# Multi-layer with different zoom ranges
freestile(
  input = list(
    counties = freestile_layer(nc, min_zoom = 0, max_zoom = 10),
    roads = freestile_layer(roads, min_zoom = 8, max_zoom = 14)
  ),
  output = "layers.pmtiles"
)
```

This allows fine-grained control over when each geographic layer appears during map zooming operations.

---

## view_tiles()

Quickly visualize a PMTiles file on an interactive map. Automatically starts a local server and opens the map in a viewer. Returns a mapgl map object that can be further customized.

### Signature

```r
view_tiles(
  input,
  layer = NULL,
  layer_type = NULL,
  color = NULL,
  opacity = 0.5,
  port = 8080,
  promote_id = NULL
)
```

### Parameters

**input**

- Path to a `.pmtiles` file

**layer**

- Source layer name within the tileset
- Defaults to the first layer if not specified

**layer_type**

- Type of map layer: `"fill"`, `"line"`, or `"circle"`
- Auto-detected from geometry type if omitted

**color**

- Layer color
- Defaults: `"navy"` (fill/line) or `"steelblue"` (circles)

**opacity**

- Layer transparency (0-1)
- Default: 0.5

**port**

- Local server port number
- Default: 8080

**promote_id**

- Feature property name to enable interactive hover behavior
- Optional

### Return Value

Returns a mapgl map object that can be piped to additional mapgl functions.

### Examples

```r
# Basic usage
view_tiles("nc.pmtiles")

# Customize appearance
view_tiles("roads.pmtiles", layer_type = "line", color = "red")

view_tiles("airports.pmtiles", layer_type = "circle", color = "orange", opacity = 0.7)

# Chain with mapgl operations
view_tiles("data.pmtiles") |>
  add_navigation_control()
```

---

## serve_tiles()

Start a local HTTP server to serve PMTiles files with CORS headers and HTTP range request support. Enables consumption by mapgl, MapLibre GL JS, and other tile clients.

### Signature

```r
serve_tiles(path, port = 8080)
```

### Parameters

**path**

- Directory containing PMTiles files OR
- Path to a single PMTiles file (its parent directory will be served)

**port**

- HTTP server port number
- Default: 8080

### Return Value

Returns invisibly a list containing:
- `url`: Server URL
- `port`: Port number
- `dir`: Directory being served

The server handle is stored internally for management via `stop_server()`.

### Details

- Runs as a background process
- If a server is already running on the specified port, it stops automatically before starting
- Suitable for files up to approximately 1 GB
- Larger files benefit from external servers (e.g., Caddy, nginx)

### Examples

```r
# Serve a directory of tiles
serve_tiles("W:/project/tiles/")

# Serve a single file
serve_tiles("us_counties.pmtiles")

# Use custom port
serve_tiles("data.pmtiles", port = 3000)

# Use in mapgl
serve_tiles("nc.pmtiles")
maplibre() |>
  add_pmtiles_source(id = "src", url = "http://localhost:8080/nc.pmtiles") |>
  add_fill_layer(source = "src", source_layer = "counties")

# Stop when finished
stop_server()
```

---

## stop_server()

Stop the local tile server started by `serve_tiles()` or `view_tiles()`.

### Signature

```r
stop_server()
```

### Parameters

None.

### Return Value

Invisibly returns `NULL`.

### Examples

```r
# Start and stop server
serve_tiles("data.pmtiles")
# ... use tiles ...
stop_server()
```
