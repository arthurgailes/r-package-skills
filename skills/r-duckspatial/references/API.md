# duckspatial API Reference

Complete function reference for the duckspatial package (v1.0.0).

## Common Arguments

Most functions that produce a duckspatial_df share the following standardized arguments (added in v1.0.0):

- `conn`: DuckDB connection. If `NULL` (default), a temporary in-memory database is used.
- `name`: Optional table name to persist the result inside DuckDB. If `NULL`, results are returned without naming.
- `mode`: Output type. Either `"duckspatial"` (lazy, default) or `"sf"` (eagerly collect to sf).
- `overwrite`: If `TRUE`, replaces existing tables of the same `name`.
- `quiet`: If `TRUE`, suppresses informational messages.

These are omitted from individual signatures below where not load-bearing for the function's purpose.

## Setup and Connection

**v1.0.0 workflow shift:** Most users never need to manage a connection explicitly. Functions accept `conn = NULL` (default) and use a temporary in-memory DuckDB. Pass an explicit `conn` only when you want to persist results across sessions or share state across pipelines.

### ddbs_create_conn(dbdir = ":memory:", ...)
Creates a DuckDB connection with the spatial extension loaded.

**Parameters:**
- `dbdir`: Path to a `.duckdb` file for persistent storage, or `":memory:"` (default).

**Returns:** DuckDB connection object

**Usage:**
```r
# Ephemeral - default behavior, often unnecessary
conn <- ddbs_create_conn()

# Persistent
conn <- ddbs_create_conn(dbdir = "project.duckdb")
```

### ddbs_stop_conn(conn)
Closes an active DuckDB connection.

**Parameters:**
- `conn`: DuckDB connection object

### ddbs_install()
Verifies and installs the DuckDB Spatial extension if not present.

### ddbs_load(conn)
Activates the Spatial extension on a connection.

**Parameters:**
- `conn`: DuckDB connection object

### ddbs_options()
Retrieves or modifies global duckspatial settings.

**Returns:** List of current options

### ddbs_set_resources(conn, ...) / ddbs_get_resources(conn)
Manages connection resource allocation (memory, threads).

**Parameters:**
- `conn`: DuckDB connection object
- `...`: Resource parameters (memory_limit, threads, etc.)

### ddbs_sitrep()
Displays duckspatial configuration information and diagnostics.

## Data Import/Export

### ddbs_open_dataset(path, ...)
Loads spatial dataset as lazy table without materializing into memory.

**Parameters:**
- `path`: File path to spatial data (GeoPackage, Shapefile, GeoJSON, etc.)
- `...`: Additional GDAL options

**Returns:** duckspatial_df object (lazy)

**Usage:**
```r
data <- ddbs_open_dataset("path/to/file.geojson")
```

### ddbs_read_table(conn, table_name)
Retrieves vectorial table from DuckDB into R environment as sf object.

**Parameters:**
- `conn`: DuckDB connection
- `table_name`: Name of table to read

**Returns:** sf object

### ddbs_write_dataset(x, path, ...)
Exports spatial dataset to disk storage.

**Parameters:**
- `x`: duckspatial_df or sf object
- `path`: Output file path
- `...`: GDAL driver options

### ddbs_write_table(x, conn, table_name, ...)
Persists sf object to DuckDB database table.

**Parameters:**
- `x`: sf object
- `conn`: DuckDB connection
- `table_name`: Name for new table
- `...`: Additional options

### ddbs_register_table(x, conn, table_name)
Registers sf object as Arrow Table in DuckDB for zero-copy access.

**Parameters:**
- `x`: sf object
- `conn`: DuckDB connection
- `table_name`: Name for registered table

## Lazy Spatial Data Frames

### as_duckspatial_df(x, ...)
Converts sf object to lazy duckspatial_df format.

**Parameters:**
- `x`: sf object
- `...`: Additional parameters

**Returns:** duckspatial_df object

**Usage:**
```r
lazy_data <- as_duckspatial_df(sf_object)
```

### is_duckspatial_df(x)
Tests if object is a duckspatial_df.

**Parameters:**
- `x`: Object to test

**Returns:** Logical

### ddbs_collect(x, ..., as = c("sf", "tibble", "raw", "geoarrow")) / collect(x)
Materializes a lazy duckspatial_df into R memory.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional arguments passed to `dplyr::collect`
- `as`: Output format:
  - `"sf"` (default) - sf object with `sfc` geometry
  - `"tibble"` - tibble with geometry column dropped (fastest)
  - `"raw"` - tibble with geometry as raw WKB bytes
  - `"geoarrow"` - tibble with geometry as `geoarrow_vctr` (zero-copy with Arrow tooling)

**Returns:** Object of the requested type.

**Usage:**
```r
# Default: sf
result <- lazy_data |> ddbs_make_valid() |> ddbs_collect()

# Attributes only, fastest
attrs <- lazy_data |> ddbs_collect(as = "tibble")
```

### ddbs_compute(x, ...)
Forces lazy evaluation and caches result in DuckDB.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** duckspatial_df object (computed)

### ddbs_geom_col(x)
Identifies the geometry column name.

**Parameters:**
- `x`: duckspatial_df object

**Returns:** Character string

### ddbs_drop_geometry(x)
Removes the geometry column, returning a lazy tibble (dbplyr) without spatial data.

**Parameters:**
- `x`: duckspatial_df, sf, tbl_lazy, or character table name

**Returns:** Lazy tibble (no geometry).

**Usage:**
```r
attrs <- ddbs_open_dataset("countries.geojson") |>
  ddbs_drop_geometry() |>
  collect()
```

## Spatial Predicates

All spatial predicate functions test relationships between geometries.

### ddbs_predicate(x, y, predicate = "intersects", conn = NULL, conn_x = NULL, conn_y = NULL, name = NULL, id_x = NULL, id_y = NULL, sparse = TRUE, distance = NULL, mode = NULL, overwrite = FALSE, quiet = TRUE)
Generic spatial relationship testing. Underlies all the predicate shortcuts (`ddbs_intersects()`, `ddbs_within()`, etc.).

**Parameters:**
- `x`, `y`: duckspatial_df, sf objects, or table names
- `predicate`: Relationship type. One of `"intersects"`, `"covers"`, `"covered_by"`, `"touches"`, `"disjoint"`, `"within"`, `"contains"`, `"overlaps"`, `"crosses"`, `"equals"`, `"intersects_extent"`, `"contains_properly"`, `"within_properly"`, `"dwithin"`.
- `id_x`, `id_y`: Optional ID column names for cross-referencing.
- `sparse`: If `TRUE` (default), returns a sparse representation (only matching pairs).
- `distance`: Numeric distance threshold (used with `predicate = "dwithin"`).
- Standard `conn`, `name`, `mode`, `overwrite`, `quiet` arguments apply.

**Returns:** Lazy duckspatial_df by default. With `mode = "sf"`, returns an eagerly collected list. When `name` is supplied, writes to DuckDB and returns `TRUE` invisibly.

### Predicate Functions
All follow pattern: `ddbs_<predicate>(x, y, ...)`

- `ddbs_intersects()` - Geometries share any space
- `ddbs_covers()` - x completely covers y (boundary can touch)
- `ddbs_covered_by()` - x is covered by y (boundary can touch)
- `ddbs_touches()` - Geometries share boundary but not interior
- `ddbs_within()` - x is completely inside y
- `ddbs_within_properly()` - x is properly within y (interior only)
- `ddbs_contains()` - x completely contains y
- `ddbs_contains_properly()` - x properly contains y (interior only)
- `ddbs_overlaps()` - Geometries overlap but neither contains the other
- `ddbs_crosses()` - Geometries cross (for lines)
- `ddbs_equals()` - Geometries are spatially equal
- `ddbs_disjoint()` - Geometries do not interact

**Parameters:**
- `x`: duckspatial_df object
- `y`: duckspatial_df or sf object
- `...`: Additional parameters

**Returns:** Logical vector or filtered duckspatial_df

### ddbs_is_within_distance(x, y, distance, ...)
Tests if geometries are within specified distance.

**Parameters:**
- `x`: duckspatial_df object
- `y`: duckspatial_df or sf object
- `distance`: Numeric distance threshold
- `...`: Additional parameters

**Returns:** Logical vector or filtered duckspatial_df

### ddbs_intersects_extent(x, extent, ...)
Tests if geometries intersect with bounding box.

**Parameters:**
- `x`: duckspatial_df object
- `extent`: Bounding box (xmin, ymin, xmax, ymax)
- `...`: Additional parameters

**Returns:** Filtered duckspatial_df

## Spatial Joins and Filters

### ddbs_join(x, y, join = "intersects", conn = NULL, conn_x = NULL, conn_y = NULL, name = NULL, distance = NULL, mode = NULL, overwrite = FALSE, quiet = FALSE)
Combines geometries based on a spatial relationship. **The `join` argument is the spatial predicate, not a SQL join type.**

**Parameters:**
- `x`, `y`: duckspatial_df, sf objects, or table names
- `join`: Spatial predicate. One of `"intersects"` (default), `"within"`, `"contains"`, `"covers"`, `"covered_by"`, `"touches"`, `"overlaps"`, `"crosses"`, `"equals"`, `"disjoint"`, `"contains_properly"`, `"within_properly"`, `"intersects_extent"`, or `"dwithin"` (use with `distance`).
- `conn_x`, `conn_y`: Per-side connections when `x` and `y` live in different DuckDB databases.
- `distance`: Numeric distance (meters) when `join = "dwithin"`.
- `name`, `mode`, `overwrite`, `quiet`: See Common Arguments.

**Returns:** duckspatial_df (or sf when `mode = "sf"`) with combined attributes from features satisfying the predicate.

**Usage:**
```r
# Each point gets attributes from any zone it falls within
result <- ddbs_join(points, zones, join = "within")

# Within 500m
near <- ddbs_join(stops, schools, join = "dwithin", distance = 500)
```

For non-spatial joins on attribute columns, use dplyr verbs (`left_join()`, `inner_join()`, etc.) directly on the lazy duckspatial_df.

### ddbs_filter(x, y, predicate = "intersects", ...)
Subsets x to features that satisfy the predicate against y.

**Parameters:**
- `x`: duckspatial_df object
- `y`: duckspatial_df or sf object
- `predicate`: Spatial relationship (same set as `ddbs_join`'s `join`). Default `"intersects"`.
- Standard `conn`, `name`, `mode`, `overwrite`, `quiet` arguments apply.

**Returns:** Filtered duckspatial_df

### ddbs_intersection(x, y, ...)
Computes geometric intersection of x and y.

**Parameters:**
- `x`: duckspatial_df object
- `y`: duckspatial_df or sf object
- `...`: Additional parameters

**Returns:** duckspatial_df with intersection geometries

### ddbs_difference(x, y, ...)
Computes geometric difference (x minus y).

**Parameters:**
- `x`: duckspatial_df object
- `y`: duckspatial_df or sf object
- `...`: Additional parameters

**Returns:** duckspatial_df with difference geometries

### ddbs_sym_difference(x, y, ...)
Computes symmetric difference (parts unique to each).

**Parameters:**
- `x`: duckspatial_df object
- `y`: duckspatial_df or sf object
- `...`: Additional parameters

**Returns:** duckspatial_df with symmetric difference geometries

### ddbs_crop(x, y, ...)
Crops x geometries to the extent of y.

**Parameters:**
- `x`: duckspatial_df object to crop
- `y`: duckspatial_df or bounding box to crop to
- `...`: Additional parameters

**Returns:** duckspatial_df cropped to y extent

### ddbs_interpolate_aw(target, source, tid, sid, extensive = NULL, intensive = NULL, weight = "sum", mode = NULL, keep_NA = TRUE, na.rm = FALSE, join_crs = NULL, conn = NULL, name = NULL, overwrite = FALSE, quiet = FALSE)
Performs areal-weighted interpolation from `source` polygons onto `target` polygons.

**Parameters:**
- `target`: duckspatial_df target polygons (these receive the interpolated values).
- `source`: duckspatial_df source polygons (carry the original values).
- `tid`: Character. Column name in `target` that uniquely identifies each target feature.
- `sid`: Character. Column name in `source` that uniquely identifies each source feature.
- `extensive`: Character vector of column names whose values are counts/totals (e.g. population). Distributed proportionally to overlapping area.
- `intensive`: Character vector of column names whose values are rates/densities (e.g. median income). Averaged using area weights.
- `weight`: For extensive variables. `"sum"` (default) divides by sum of overlapping area; `"total"` divides by full source geometry area (strict mass preservation).
- `keep_NA`: If `TRUE` (default), retains target features with no overlap as NA values.
- `na.rm`: If `TRUE`, drops missing values from interpolation.
- `join_crs`: Optional CRS (EPSG or WKT) used during area calculations. Useful when inputs are in geographic coordinates.
- Standard `conn`, `name`, `mode`, `overwrite`, `quiet` arguments apply.

**Returns:** duckspatial_df keyed by `tid` with interpolated columns.

**Usage:**
```r
# Population (extensive) and median income (intensive) from census tracts to neighborhoods
result <- ddbs_interpolate_aw(
  target    = neighborhoods,
  source    = census,
  tid       = "neighborhood_id",
  sid       = "tract_id",
  extensive = "population",
  intensive = "median_income"
) |>
  ddbs_collect()
```

## Data Utilities

### ddbs_read_meta(path, ...)
Reads spatial metadata from a file without loading geometry data.

**Parameters:**
- `path`: Path to spatial file
- `...`: Additional GDAL options

**Returns:** Data frame with layer name, geometry type, CRS, feature count, and extent

## Geometry Construction

### ddbs_generate_points(x, n, ...)
Produces n random points within each feature's bounding box.

**Parameters:**
- `x`: duckspatial_df object
- `n`: Number of points per feature
- `...`: Additional parameters

**Returns:** duckspatial_df with point geometries

### ddbs_as_points(x, ...)
Converts non-point geometries to representative point geometries.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** duckspatial_df with point geometries

### ddbs_quadkey(x, level, ...)
Converts point geometries to QuadKey tile identifiers (Bing Maps tiles).

**Parameters:**
- `x`: duckspatial_df with point geometries
- `level`: Zoom level (0-23)
- `...`: Additional parameters

**Returns:** duckspatial_df with quadkey column

## Line Operations

### ddbs_line_startpoint(x, ...)
Gets the first point of linestring geometries.

**Parameters:**
- `x`: duckspatial_df with linestring geometries
- `...`: Additional parameters

**Returns:** duckspatial_df with start point geometries

### ddbs_line_endpoint(x, ...)
Gets the last point of linestring geometries.

**Parameters:**
- `x`: duckspatial_df with linestring geometries
- `...`: Additional parameters

**Returns:** duckspatial_df with end point geometries

### ddbs_line_interpolate(x, distance, ...)
Interpolates a point at a given distance along each linestring.

**Parameters:**
- `x`: duckspatial_df with linestring geometries
- `distance`: Distance along line (absolute or fraction 0-1)
- `...`: Additional parameters

**Returns:** duckspatial_df with interpolated point geometries

### ddbs_line_merge(x, ...)
Merges connected linestring segments into longer linestrings.

**Parameters:**
- `x`: duckspatial_df with linestring geometries
- `...`: Additional parameters

**Returns:** duckspatial_df with merged linestrings

### ddbs_line_substring(x, start, end, ...)
Extracts a substring of linestring between two distances.

**Parameters:**
- `x`: duckspatial_df with linestring geometries
- `start`: Start distance (absolute or fraction 0-1)
- `end`: End distance (absolute or fraction 0-1)
- `...`: Additional parameters

**Returns:** duckspatial_df with substring geometries

## Geometry Processing

### ddbs_boundary(x, ...)
Extracts geometry boundary (perimeter).

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** duckspatial_df with boundary geometries

### ddbs_buffer(x, distance, num_triangles = 8L, cap_style = "CAP_ROUND", join_style = "JOIN_ROUND", mitre_limit = 1, ...)
Creates buffer zones around geometries.

**Parameters:**
- `x`: duckspatial_df object
- `distance`: Buffer distance in CRS units
- `num_triangles`: Triangles per quarter circle (higher = smoother). Default `8`.
- `cap_style`: Line ending style. `"CAP_ROUND"` (default), `"CAP_FLAT"`, or `"CAP_SQUARE"`.
- `join_style`: Corner join style. `"JOIN_ROUND"` (default), `"JOIN_MITRE"`, or `"JOIN_BEVEL"`.
- `mitre_limit`: Mitre length ratio (only used with `JOIN_MITRE`). Default `1`.
- Standard `conn`, `name`, `mode`, `overwrite`, `quiet` arguments apply.

**Returns:** duckspatial_df with buffered geometries

**Usage:**
```r
buffered <- points |> ddbs_buffer(distance = 1000)  # 1km buffer

# Square caps for road buffers
roads_buf <- roads |> ddbs_buffer(distance = 5, cap_style = "CAP_SQUARE")
```

### ddbs_build_area(x, ...)
Assembles polygons from multiple linestrings.

**Parameters:**
- `x`: duckspatial_df with linestring geometries
- `...`: Additional parameters

**Returns:** duckspatial_df with polygon geometries

### ddbs_centroid(x, ...)
Calculates geometric center points.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** duckspatial_df with point geometries at centroids

### ddbs_concave_hull(x, ...) / ddbs_convex_hull(x, ...)
Computes minimum enclosing shapes.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters (concavity for concave_hull)

**Returns:** duckspatial_df with hull geometries

### ddbs_exterior_ring(x, ...)
Extracts outer boundary of polygon geometries.

**Parameters:**
- `x`: duckspatial_df with polygon geometries
- `...`: Additional parameters

**Returns:** duckspatial_df with linestring geometries

### ddbs_make_polygon(x, ...)
Creates polygon from closed linestring.

**Parameters:**
- `x`: duckspatial_df with closed linestring geometries
- `...`: Additional parameters

**Returns:** duckspatial_df with polygon geometries

### ddbs_polygonize(x, ...)
Assembles polygons from linestring networks.

**Parameters:**
- `x`: duckspatial_df with linestring geometries
- `...`: Additional parameters

**Returns:** duckspatial_df with polygon geometries

### ddbs_union(x, y, ...)
Pairwise (per-row) union of two geometry sets. Combines `x` and `y` geometries row-by-row into single geometries.

**Parameters:**
- `x`, `y`: duckspatial_df objects
- Standard `conn`, `name`, `mode`, `overwrite`, `quiet` arguments apply.

### ddbs_union_agg(x, ...)
Aggregate union (dissolve). Collapses all features in `x` into a single geometry.

**Parameters:**
- `x`: duckspatial_df object
- Standard `conn`, `name`, `mode`, `overwrite`, `quiet` arguments apply.

**Returns:** duckspatial_df with one row containing the dissolved geometry.

**Usage:**
```r
# Dissolve all polygons into a single multipolygon
dissolved <- polygons |>
  ddbs_make_valid() |>
  ddbs_union_agg()

# Dissolve by group (e.g., by region) using dplyr
by_region <- polygons |>
  group_by(region) |>
  summarise(geom = ddbs_union_agg(geom))
```

### ddbs_combine(x, ...)
Combines geometries without dissolving boundaries.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** duckspatial_df with combined geometries

### ddbs_dump(x, ...)
Explodes multi-geometry features into individual single-part geometries.

**Parameters:**
- `x`: duckspatial_df with multi-geometry types
- `...`: Additional parameters

**Returns:** duckspatial_df with single-part geometries (more rows)

### ddbs_multi(x, ...)
Wraps single-part geometries into their multi-type equivalents.

**Parameters:**
- `x`: duckspatial_df with single-part geometries
- `...`: Additional parameters

**Returns:** duckspatial_df with multi-type geometries

### ddbs_maximum_inscribed_circle(x, ...)
Computes the largest circle that fits within each polygon.

**Parameters:**
- `x`: duckspatial_df with polygon geometries
- `...`: Additional parameters

**Returns:** duckspatial_df with circle center points and radius

### ddbs_minimum_rotated_rectangle(x, ...)
Computes the minimum area bounding rectangle (may be rotated).

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** duckspatial_df with rectangle polygon geometries

### ddbs_voronoi(x, ...)
Generates Voronoi diagram from point geometries.

**Parameters:**
- `x`: duckspatial_df with point geometries
- `...`: Additional parameters

**Returns:** duckspatial_df with Voronoi polygon geometries

## Coordinate Accessors

### Coordinate Extraction Functions
Extract coordinate values from geometries:

- `ddbs_x(x, ...)` - Extract X (longitude) coordinate
- `ddbs_y(x, ...)` - Extract Y (latitude) coordinate
- `ddbs_z(x, ...)` - Extract Z (elevation) coordinate
- `ddbs_m(x, ...)` - Extract M (measure) coordinate

### ddbs_locate_along(x, measure, ...)
Returns point geometry at the given measure value along a linestring.

**Parameters:**
- `x`: duckspatial_df with linestring geometries
- `measure`: M-value to locate
- `...`: Additional parameters

**Returns:** duckspatial_df with point geometries

### ddbs_locate_between(x, start, end, ...)
Returns geometry segments between two measure values.

**Parameters:**
- `x`: duckspatial_df with M-enabled linestring geometries
- `start`: Start M-value
- `end`: End M-value
- `...`: Additional parameters

**Returns:** duckspatial_df with linestring segments

## Coordinate Operations

### ddbs_transform(x, crs, ...)
Converts between coordinate reference systems.

**Parameters:**
- `x`: duckspatial_df object
- `crs`: Target CRS (EPSG code or proj4string)
- `...`: Additional parameters

**Returns:** duckspatial_df in new CRS

**Usage:**
```r
# Transform to UTM Zone 33N
utm <- data |> ddbs_transform(crs = 32633)
```

### ddbs_set_crs(x, crs, ...)
Assigns a CRS to a geometry without reprojecting coordinates (use when CRS is missing or wrong).

**Parameters:**
- `x`: duckspatial_df object
- `crs`: CRS to assign (EPSG code or proj4string)
- `...`: Additional parameters

**Returns:** duckspatial_df with new CRS metadata (coordinates unchanged)

### ddbs_flip_coordinates(x, ...)
Swaps X and Y coordinate axes.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** duckspatial_df with flipped coordinates

## Geometry Validation

### ddbs_geometry_type(x, ...)
Identifies geometry type for each feature.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** Character vector of geometry types

### ddbs_make_valid(x, ...)
Repairs invalid geometries using ST_MakeValid.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** duckspatial_df with valid geometries

**Usage:**
```r
# Always validate before union operations
valid_data <- data |> ddbs_make_valid()
```

### ddbs_remove_repeated_points(x, ...)
Removes consecutive duplicate vertices from geometries.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** duckspatial_df with deduplicated vertices

### ddbs_simplify(x, tolerance = 0, preserve_topology = FALSE, ...)
Reduces geometry complexity.

**Parameters:**
- `x`: duckspatial_df object
- `tolerance`: Simplification tolerance in CRS units. Default `0` (no-op).
- `preserve_topology`: If `FALSE` (default), uses Douglas-Peucker (faster, may produce invalid geometries). If `TRUE`, uses topology-preserving simplification (slower but output remains valid).
- Standard `conn`, `name`, `mode`, `overwrite`, `quiet` arguments apply.

**Returns:** duckspatial_df with simplified geometries

**Usage:**
```r
simplified <- polygons |> ddbs_simplify(tolerance = 100)

# Safe simplification for self-touching geometries
simplified <- polygons |> ddbs_simplify(tolerance = 100, preserve_topology = TRUE)
```

### Validation Check Functions

All return logical vectors:

- `ddbs_is_simple(x, ...)` - Geometry has no self-intersections
- `ddbs_is_valid(x, ...)` - Geometry is topologically valid
- `ddbs_is_closed(x, ...)` - Linestring start equals end
- `ddbs_is_empty(x, ...)` - Geometry is empty
- `ddbs_is_ring(x, ...)` - Linestring is simple and closed

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** Logical vector

### Dimension Check Functions

- `ddbs_has_z(x, ...)` - Geometry has Z coordinate
- `ddbs_has_m(x, ...)` - Geometry has M coordinate

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** Logical vector

### Dimension Forcing Functions

- `ddbs_force_2d(x, ...)` - Removes Z and M coordinates
- `ddbs_force_3d(x, ...)` - Ensures Z coordinate (adds 0 if missing)
- `ddbs_force_4d(x, ...)` - Ensures Z and M coordinates

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** duckspatial_df with specified dimensions

## Format Conversion

### ddbs_as_text(x, ...)
Exports geometries to WKT (Well-Known Text) format.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** Character vector of WKT strings

### ddbs_as_wkb(x, ...)
Exports geometries to WKB (Well-Known Binary) format.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** List of raw vectors

### ddbs_as_hexwkb(x, ...)
Exports geometries to hexadecimal WKB format.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** Character vector of hex strings

### ddbs_as_geojson(x, ...)
Exports geometries to GeoJSON format.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** Character vector of GeoJSON strings

## Spatial Extent and Boundaries

### ddbs_bbox(x, ...)
Retrieves minimum bounding rectangle for each feature.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** Data frame with xmin, ymin, xmax, ymax columns

### ddbs_make_envelope(xmin, ymin, xmax, ymax, ...)
Creates a rectangular envelope polygon from bounding coordinates.

**Parameters:**
- `xmin`, `ymin`, `xmax`, `ymax`: Bounding coordinates
- `...`: Additional parameters (crs)

**Returns:** duckspatial_df with rectangle polygon geometry

### ddbs_envelope(x, ...)
Extracts axis-aligned bounding box as geometry.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** duckspatial_df with polygon bounding boxes

## Affine Transformations

### ddbs_flip(x, axis, ...)
Mirrors geometries along specified axis.

**Parameters:**
- `x`: duckspatial_df object
- `axis`: "x" or "y"
- `...`: Additional parameters

**Returns:** duckspatial_df with flipped geometries

### ddbs_rotate(x, angle, ...)
Rotates geometries around centroid.

**Parameters:**
- `x`: duckspatial_df object
- `angle`: Rotation angle in degrees
- `...`: Additional parameters

**Returns:** duckspatial_df with rotated geometries

### ddbs_rotate_3d(x, angle, axis, ...)
Rotates 3D geometries along specified axis.

**Parameters:**
- `x`: duckspatial_df object
- `angle`: Rotation angle in degrees
- `axis`: "x", "y", or "z"
- `...`: Additional parameters

**Returns:** duckspatial_df with rotated 3D geometries

### ddbs_scale(x, factor, ...)
Resizes geometries by scale factor(s).

**Parameters:**
- `x`: duckspatial_df object
- `factor`: Numeric scale factor (single value or c(x_scale, y_scale))
- `...`: Additional parameters

**Returns:** duckspatial_df with scaled geometries

### ddbs_shear(x, factor, ...)
Applies shearing transformation.

**Parameters:**
- `x`: duckspatial_df object
- `factor`: Numeric shear factor
- `...`: Additional parameters

**Returns:** duckspatial_df with sheared geometries

### ddbs_shift(x, offset, ...)
Translates geometries by offset.

**Parameters:**
- `x`: duckspatial_df object
- `offset`: Numeric vector c(x_offset, y_offset)
- `...`: Additional parameters

**Returns:** duckspatial_df with shifted geometries

**Usage:**
```r
shifted <- data |> ddbs_shift(offset = c(1000, 500))
```

## Measurements

### ddbs_area(x, ...)
Computes area of polygon geometries.

**Parameters:**
- `x`: duckspatial_df with polygon geometries
- `...`: Additional parameters

**Returns:** Numeric vector of areas in CRS units squared

### ddbs_length(x, ...)
Computes length of linestring geometries.

**Parameters:**
- `x`: duckspatial_df with linestring geometries
- `...`: Additional parameters

**Returns:** Numeric vector of lengths in CRS units

### ddbs_perimeter(x, ...)
Computes perimeter of polygon geometries.

**Parameters:**
- `x`: duckspatial_df with polygon geometries
- `...`: Additional parameters

**Returns:** Numeric vector of perimeters in CRS units

### ddbs_distance(x, y, ...)
Computes minimum distance between geometries.

**Parameters:**
- `x`: duckspatial_df object
- `y`: duckspatial_df or sf object
- `...`: Additional parameters

**Returns:** Numeric vector or matrix of distances

**Usage:**
```r
# Distance from each point to nearest polygon
distances <- ddbs_distance(points, polygons)
```

## Database Utilities

### ddbs_create_schema(conn, schema_name, ...)
Establishes new database schema.

**Parameters:**
- `conn`: DuckDB connection
- `schema_name`: Name for new schema
- `...`: Additional parameters

**Returns:** NULL (modifies database)

### ddbs_crs(x)
Retrieves coordinate reference system metadata.

**Parameters:**
- `x`: duckspatial_df object

**Returns:** CRS object

### ddbs_drivers()
Lists GDAL-supported file formats available.

**Returns:** Data frame with driver information

### ddbs_glimpse(x, ...)
Shows preview of first rows.

**Parameters:**
- `x`: duckspatial_df object
- `...`: Additional parameters

**Returns:** Printed summary

### ddbs_list_tables(conn, schema = NULL, ...)
Enumerates database tables.

**Parameters:**
- `conn`: DuckDB connection
- `schema`: Optional schema name filter
- `...`: Additional parameters

**Returns:** Character vector of table names

## sf Integration Methods

duckspatial_df objects support standard sf methods:

- `st_crs(x)` - Get/set CRS
- `st_geometry(x)` - Extract geometry column
- `st_bbox(x)` - Get bounding box
- `st_as_sf(x)` - Convert to sf object (same as ddbs_collect)
- `print(x)` - Print object summary

## dplyr Integration Methods

duckspatial_df objects support dplyr verbs:

- `left_join()`, `inner_join()`, `right_join()`, `full_join()` - Non-spatial joins
- `head(x, n)` - Return first n rows
- `count(x, ...)` - Count by group
- `glimpse(x)` - Compact summary
- `compute(x)` - Force computation
- `distinct(x, ...)` - Select unique rows
- `filter()`, `select()`, `mutate()`, `summarise()`, `group_by()` - Standard dplyr operations

**Note:** All dplyr operations preserve the lazy duckspatial_df structure until `collect()` is called.
