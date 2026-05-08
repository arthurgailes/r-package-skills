# Getting Started with freestiler

## Overview

freestiler is a vector tile engine for R and Python that transforms spatial data into PMTiles archives. As described in the documentation, it "creates PMTiles archives from spatial data" with a Rust-based engine that requires no additional installation.

The tool supports two tile formats:
- **Mapbox Vector Tiles (MVT)** - the default; widely-supported protobuf format that works with MapLibre GL JS, Mapbox GL JS (3.21+), deck.gl, and most other clients
- **MapLibre Tiles (MLT)** - opt-in columnar format with smaller files for polygon/line data; requires MapLibre GL JS 5.21+ or mapgl

## Installation

For R users, install from r-universe:

```r
install.packages(
  "freestiler",
  repos = c("https://walkerke.r-universe.dev", "https://cloud.r-project.org")
)
```

The r-universe build includes the Rust DuckDB backend on macOS and Linux. GitHub installation is also available via devtools.

## Creating Your First Tileset

The primary function is `freestile()`. Here's a basic example using the North Carolina counties dataset:

```r
library(sf)
library(freestiler)

nc <- st_read(system.file("shape/nc.shp", package = "sf"))

freestile(nc, "nc_counties.pmtiles", layer_name = "counties")
```

## Scaling to Larger Datasets

For substantial datasets like US block groups (242,000 features), the process handles thousands of spatial units efficiently:

```r
library(tigris)
options(tigris_use_cache = TRUE)

bgs <- block_groups(cb = TRUE)

freestile(
  bgs,
  "us_bgs.pmtiles",
  layer_name = "bgs",
  min_zoom = 4,
  max_zoom = 12
)
```

## Viewing Tilesets

The `view_tiles()` function launches an interactive map with automatic layer detection:

```r
view_tiles("us_bgs.pmtiles")
```

For advanced customization, use `serve_tiles()` alongside the mapgl library for manual styling control.

## Tile Format Selection

Choose between formats based on your needs:

```r
# MVT is the default - no tile_format argument needed for broad compatibility
freestile(nc, "nc_mvt.pmtiles", layer_name = "counties")

# Switch to MLT for smaller polygon/line files (mapgl + MapLibre GL JS 5.21+)
freestile(nc, "nc_mlt.pmtiles", layer_name = "counties", tile_format = "mlt")
```

MLT typically produces smaller files for polygon-heavy datasets (about 15-20% savings on real-world Census data); point-only datasets see minimal benefit. See `maplibre-tiles.md` for ecosystem details.

## Zoom Level Control

Specify the zoom range for your tileset:

```r
freestile(nc, "nc_z4_10.pmtiles",
  layer_name = "counties",
  min_zoom = 4,
  max_zoom = 10
)
```

## Feature Thinning for Large Datasets

The `drop_rate` parameter exponentially reduces features at lower zoom levels while maintaining coverage:

```r
freestile(nc, "nc_dropping.pmtiles",
  layer_name = "counties",
  drop_rate = 2.5,
  base_zoom = 10
)
```

## File-Based Input

Process spatial files directly without loading into memory:

```r
# GeoParquet files
freestile_file("census_blocks.parquet", "blocks.pmtiles")

# Other formats via DuckDB
freestile_file("counties.gpkg", "counties.pmtiles", engine = "duckdb")
```

## DuckDB Queries

Execute SQL transformations before tiling:

```r
freestile_query(
  "SELECT * FROM ST_Read('counties.shp') WHERE pop > 50000",
  "large_counties.pmtiles"
)
```

For massive point datasets, enable streaming mode:

```r
freestile_query(
  query = "SELECT naics, state, ST_Point(lon, lat) AS geometry FROM jobs_dots",
  output = "us_jobs_dots.pmtiles",
  db_path = db_path,
  layer_name = "jobs",
  tile_format = "mvt",
  min_zoom = 4,
  max_zoom = 14,
  base_zoom = 14,
  drop_rate = 2.5,
  source_crs = "EPSG:4326",
  streaming = "always",
  overwrite = TRUE
)
```

This streaming approach successfully processed 146 million job points into a 2.3 GB archive in approximately 12 minutes.

## Multi-Layer Tilesets

Combine multiple layers with independent zoom controls:

```r
pts <- st_centroid(nc)

freestile(
  list(
    counties = freestile_layer(nc, min_zoom = 0, max_zoom = 10),
    centroids = freestile_layer(pts, min_zoom = 6, max_zoom = 14)
  ),
  "nc_layers.pmtiles"
)
```

## Point Clustering

Merge proximate points into clusters with aggregate counts:

```r
freestile(pts, "nc_clustered.pmtiles",
  layer_name = "centroids",
  cluster_distance = 50,
  cluster_maxzoom = 8
)
```

## Feature Coalescing

Merge features sharing identical attributes within tiles:

```r
freestile(nc, "nc_coalesced.pmtiles",
  layer_name = "counties",
  coalesce = TRUE
)
```

This consolidation joins connected lines and groups polygons into MultiPolygon collections.
