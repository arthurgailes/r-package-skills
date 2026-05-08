# SQL Preprocessing Patterns

Filter before tiling to reduce output size:

## Filter by Bounding Box

```r
freestile_query(
  query = "
    SELECT * FROM read_parquet('data.parquet')
    WHERE lon BETWEEN -100 AND -90
      AND lat BETWEEN 30 AND 40
  ",
  output = "filtered.pmtiles",
  layer_name = "subset"
)
```

## Join and Aggregate

```r
freestile_query(
  query = "
    SELECT
      g.geom,
      g.id,
      COUNT(p.id) as point_count
    FROM read_parquet('grid.parquet') g
    LEFT JOIN read_parquet('points.parquet') p
      ON ST_Within(p.geom, g.geom)
    GROUP BY g.geom, g.id
  ",
  output = "aggregated.pmtiles",
  layer_name = "grid"
)
```

## When to Use Each Function

```r
# Use freestile() when:
# - Data already in R as sf object
# - <1M features
# - No preprocessing needed
nc <- st_read("nc.shp")
freestile(input = nc, output = "nc.pmtiles", layer_name = "counties")

# Use freestile_file() when:
# - Large file you don't need to load
# - Simple file-to-tiles conversion
# - GeoParquet, GPKG, SHP input
freestile_file(input = "large.gpkg", output = "output.pmtiles", layer_name = "layer")

# Use freestile_query() when:
# - Need SQL preprocessing
# - Filtering/joining before tiling
# - Streaming massive point datasets
freestile_query(
  query = "SELECT * FROM read_parquet('huge.parquet') WHERE condition",
  output = "output.pmtiles",
  layer_name = "layer",
  streaming = "always"
)

# Use freestile() with freestile_layer() when:
# - Multiple datasets at different zoom levels
# - Hierarchical display (states -> counties -> blocks)
# Note: layer names come from named-list keys, not freestile_layer() args.
freestile(
  input = list(
    states   = freestile_layer(states,   min_zoom = 0, max_zoom = 6),
    counties = freestile_layer(counties, min_zoom = 4, max_zoom = 10)
  ),
  output = "multi.pmtiles"
)
```
