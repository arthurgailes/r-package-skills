---
name: r-duckspatial
description: Use when code loads or uses duckspatial (library(duckspatial), duckspatial::), performing spatial joins or areal interpolation on large vector datasets in R, or needing faster spatial operations than sf
---

# duckspatial: Memory-Efficient Spatial Analysis

## Overview

**duckspatial** bridges R's sf ecosystem with DuckDB's spatial extension for memory-efficient analysis of large spatial datasets. The primary class is `duckspatial_df`: a lazy, table-like object that materializes into R memory on demand.

**Core principle:** Operations execute in DuckDB and stay outside R memory until you call `ddbs_collect()`. As of v1.0.0, you do NOT need to manage a DuckDB connection explicitly for the typical workflow - duckspatial uses a temporary database by default.

## References

Read `references/API.md` before writing code.

- `references/API.md` - Complete function reference and spatial operations

## When to Use

- Dataset too large for sf to load into memory
- Spatial joins / filters with millions of features
- Areal-weighted interpolation across large geographies
- Repeated spatial operations benefit from DuckDB's query optimization
- Working with spatial files larger than 1GB

## When NOT to Use

- Small datasets (<100K features) - sf is simpler
- Complex spatial topology operations not in DuckDB spatial extension
- Need GEOS-specific functionality (sf provides more geometry operations)
- Interactive spatial visualization - materialize with `ddbs_collect()` first

## Quick Reference

| Operation                 | Function                                           | Key Parameters                      |
| ------------------------- | -------------------------------------------------- | ----------------------------------- |
| **Load file**             | `ddbs_open_dataset(path)`                          | Returns lazy duckspatial_df         |
| **Convert from sf**       | `as_duckspatial_df(sf_obj)`                        | Registers sf as lazy table          |
| **Materialize**           | `ddbs_collect(x, as = "sf")`                       | as: "sf" (default), "tibble", "raw", "geoarrow" |
| **Drop geometry**         | `ddbs_drop_geometry(x)`                            | Returns lazy tibble                 |
| **Spatial join**          | `ddbs_join(x, y, join = "intersects")`             | `join` is the SPATIAL predicate     |
| **Attribute join**        | `left_join(x, y, by = ...)`                        | Use dplyr verbs for non-spatial joins |
| **Spatial filter**        | `ddbs_filter(x, y, predicate = "intersects")`      | Subset by spatial relation          |
| **Crop to extent**        | `ddbs_crop(x, y)`                                  | Clip geometries to y bbox           |
| **Areal interpolation**   | `ddbs_interpolate_aw(target, source, tid, sid, ...)` | extensive / intensive variables   |
| **Buffer**                | `ddbs_buffer(x, distance)`                         | + cap_style, join_style, etc.       |
| **Union (per-row)**       | `ddbs_union(x, y)`                                 | Pairwise union                      |
| **Dissolve (aggregate)**  | `ddbs_union_agg(x)`                                | Single multipolygon                 |
| **Simplify**              | `ddbs_simplify(x, tolerance = 0)`                  | + preserve_topology = FALSE         |
| **Transform CRS**         | `ddbs_transform(x, crs)`                           | EPSG code                           |
| **Set CRS (no reproject)**| `ddbs_set_crs(x, crs)`                             | Tag without transforming            |
| **Validate**              | `ddbs_make_valid(x)`                               | Fix invalid geometries              |

**See `references/API.md` for complete function reference.**

## Core Pattern

```r
library(duckspatial)
library(dplyr)
library(sf)

# Load large dataset as lazy table (not in R memory)
buildings <- ddbs_open_dataset("buildings.gpkg")

# Or convert existing sf object
neighborhoods <- as_duckspatial_df(sf_neighborhoods)

# Chain operations - all execute lazily in DuckDB
result <- buildings |>
  ddbs_make_valid() |>
  ddbs_transform(crs = 32633) |>
  ddbs_join(neighborhoods, join = "intersects") |>
  ddbs_collect()                              # NOW pull into R as sf

# Result is sf, ready for plotting / further analysis
plot(result["neighborhood_name"])
```

## Common Workflows

### Spatial Join (Point-in-Polygon)

`ddbs_join()`'s `join` argument is the **spatial predicate**, not a SQL join type. For each point, attributes from any polygon satisfying the predicate are appended.

```r
points <- ddbs_open_dataset("locations.geojson")
zones  <- as_duckspatial_df(zone_boundaries)

joined <- ddbs_join(points, zones, join = "within") |>
  ddbs_collect()
```

For non-spatial joins on attribute columns, use dplyr verbs (`left_join()`, `inner_join()`, etc.) directly on the lazy duckspatial_df.

### Areal-Weighted Interpolation

The signature uses `target` and `source` (target receives the values), with `tid` / `sid` as ID columns and explicit `extensive` (counts that sum, e.g. population) vs `intensive` (rates that average, e.g. density) arguments.

```r
census <- ddbs_open_dataset("census_tracts.gpkg")
neighborhoods <- as_duckspatial_df(neighborhoods_sf)

interpolated <- ddbs_interpolate_aw(
  target    = neighborhoods,
  source    = census,
  tid       = "neighborhood_id",
  sid       = "tract_id",
  extensive = "population",       # sums proportionally to overlap
  intensive = "median_income"     # averages weighted by area
) |>
  ddbs_collect()
```

### Large File Filtering

```r
# Filter in DuckDB before pulling anything into R
parcels <- ddbs_open_dataset("parcels_nationwide.shp")

city_parcels <- parcels |>
  ddbs_filter(study_area, predicate = "intersects") |>
  ddbs_make_valid() |>
  ddbs_simplify(tolerance = 0.5) |>
  ddbs_collect()
```

### Mixing dplyr verbs and spatial ops

duckspatial v1.0.0 implements DuckDB macros so duckspatial functions can be used inside dplyr verbs. Operations stay lazy until `ddbs_collect()`.

```r
filtered <- buildings |>
  filter(height > 50) |>                       # dplyr (attribute)
  mutate(area_m2 = ddbs_area(geom)) |>         # ddbs_* inside mutate
  ddbs_filter(downtown, predicate = "within")  # spatial filter

# sf accessors also work on lazy duckspatial_df
crs_info <- st_crs(buildings)
bbox     <- st_bbox(buildings)
```

## Output Formats from `ddbs_collect()`

```r
ddbs_collect(x, as = "sf")        # default: sf object
ddbs_collect(x, as = "tibble")    # fastest: tibble, geometry dropped
ddbs_collect(x, as = "raw")       # tibble with WKB raw bytes
ddbs_collect(x, as = "geoarrow")  # tibble with geoarrow_vctr column
```

Use `"tibble"` when you only need attributes (faster than dropping geometry afterwards). Use `"geoarrow"` for zero-copy handoff to Arrow-based tooling.

## Common Mistakes

| Mistake                                              | Fix                                                          |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| Using `join = "left"` in `ddbs_join()`               | `join` is the **spatial predicate** ("intersects", "within"). For SQL joins use dplyr `left_join()` |
| Passing `predicate=` to `ddbs_join()`                | `ddbs_join()` has no `predicate` arg; its `join` IS the predicate. `predicate` exists on `ddbs_filter()` and `ddbs_predicate()` |
| Calling `ddbs_interpolate_aw(x, y, values = ...)`    | Use `target`, `source`, `tid`, `sid`, `extensive`, `intensive` |
| Forgetting `ddbs_collect()`                          | Data stays lazy in DuckDB - call `ddbs_collect()` to get sf  |
| Using `ddbs_union()` to dissolve                     | `ddbs_union()` is pairwise; use `ddbs_union_agg()` for dissolve |
| Not validating before union/dissolve                 | Always `ddbs_make_valid()` first                             |
| Wrong CRS for measurements                           | Transform to projected CRS before distance/area              |
| Calling `ddbs_simplify(x)` without tolerance         | Default `tolerance = 0` is a no-op; pass a real tolerance    |
| Manually creating a connection for simple workflows  | v1.0.0+ uses a temp DB by default; skip `ddbs_create_conn()` unless persisting |

## Performance Characteristics

**When duckspatial is faster:**
- Spatial joins with >100K features per layer
- Operations on files >500MB
- Repeated queries (DuckDB caches and optimizes)
- Chained spatial operations (single optimized query plan)

**When sf is faster:**
- Small datasets (<100K features)
- Complex GEOS operations not in DuckDB spatial extension
- One-shot operations where lazy setup overhead dominates

**Memory pattern:**
```r
# BAD: Loads entire file into R memory
data <- st_read("huge_file.gpkg") |>
  st_filter(bbox)

# GOOD: Filters in DuckDB, only loads result
data <- ddbs_open_dataset("huge_file.gpkg") |>
  ddbs_intersects_extent(bbox) |>
  ddbs_collect()
```

## Gotchas

1. **First call overhead:** Initial `ddbs_open_dataset()` takes seconds for connection setup
2. **Lazy evaluation:** Operations don't execute until `ddbs_collect()` - errors appear late
3. **Geometry validity:** Invalid geometries cause cryptic DuckDB errors - validate early with `ddbs_make_valid()`
4. **CRS handling:** DuckDB spatial extension uses EPSG codes; proj4 strings may not work
5. **Non-persistent:** The default temp DuckDB discards data after session - pass `conn` (from `ddbs_create_conn(dbdir = "...")`) to persist
6. **`ddbs_join` predicate naming:** Argument is `join`, not `predicate` - this is the most common bug

## Spatial Predicates Reference

| Predicate            | Meaning                                    | Use Case                    |
| -------------------- | ------------------------------------------ | --------------------------- |
| `intersects`         | Geometries share any space                 | Most permissive, default    |
| `within`             | x completely inside y                      | Points in polygons          |
| `contains`           | x completely contains y                    | Find polygons containing pt |
| `contains_properly`  | x contains y in interior only              | Strict containment          |
| `within_properly`    | x within y in interior only                | Strict within               |
| `covers`             | x covers y (boundary can touch)            | Relaxed containment         |
| `covered_by`         | x is covered by y                          | Reverse of covers           |
| `touches`            | Share boundary but not interior            | Adjacent polygons           |
| `overlaps`           | Share space, neither contains other        | Partial overlap             |
| `crosses`            | Geometries cross (lines)                   | Line intersections          |
| `equals`             | Spatially identical                        | Exact match                 |
| `disjoint`           | No spatial interaction                     | Exclusion filtering         |
| `intersects_extent`  | Bounding boxes intersect                   | Fast prefilter              |

## Real-World Impact

**Before (sf):** 45 minutes to spatial join 2M buildings to 500 neighborhoods, 32GB RAM
**After (duckspatial):** 3 minutes, 4GB RAM

Operations scale sub-linearly with data size due to DuckDB's columnar storage and parallel execution.
