# Zoom Level Strategy

## General Guidance

- `min_zoom`: When features first become visible (lower = visible sooner = larger file)
- `max_zoom`: Maximum detail level (higher = more detail = larger file)
- `base_zoom`: Optimization hint (typically max_zoom - 1)

## Examples by Geometry Type

```r
# Large polygons (states, countries)
freestile(input = states, output = "states.pmtiles", layer_name = "states",
          min_zoom = 0, max_zoom = 6)

# Medium polygons (counties)
freestile(input = counties, output = "counties.pmtiles", layer_name = "counties",
          min_zoom = 4, max_zoom = 10)

# Small features (buildings, blocks)
freestile(input = blocks, output = "blocks.pmtiles", layer_name = "blocks",
          min_zoom = 8, max_zoom = 14)

# Points (variable density)
freestile(input = points, output = "points.pmtiles", layer_name = "points",
          min_zoom = 0, max_zoom = 14)
```

## Tile Format: MVT vs MLT

**Default: Mapbox Vector Tiles (MVT)**
- Row-based protobuf encoding
- Broadest viewer compatibility (MapLibre GL JS, Mapbox GL JS 3.21+, deck.gl, Folium, etc.)
- Use unless you have a specific reason to opt out

**Alternative: MapLibre Tiles (MLT)**
```r
freestile(input = data, output = "output.pmtiles", layer_name = "layer",
          tile_format = "mlt")
```
- Columnar encoding with delta/RLE/dictionary compression
- Smaller files for polygon/line data (about 15-20% savings on Census data)
- Minimal benefit for point-only datasets
- Requires MapLibre GL JS 5.21+ (mapgl supports it natively)
