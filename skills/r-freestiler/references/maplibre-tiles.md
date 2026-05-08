# MapLibre Tiles (MLT) Format

freestiler can encode tiles in two formats inside the same PMTiles archive: MVT (Mapbox Vector Tiles, the default) or MLT (MapLibre Tiles). Pick MLT only when both ends of the pipeline support it.

## When to Use Each

| Format | Default? | Best for | Compatible with |
| --- | --- | --- | --- |
| MVT | yes | Cross-tool compatibility, Python clients, deck.gl, Folium | MapLibre GL JS, Mapbox GL JS (3.21+), deck.gl, MapLibre Native, MapTiler/Tippecanoe pipelines |
| MLT | opt-in | Smaller polygon/line tiles when you control the viewer | mapgl (R), MapLibre GL JS 5.21+ (experimental), mlt-core |

Use MVT (the default) unless you have a specific reason to switch. Use MLT when:

- You are visualising in mapgl or MapLibre GL JS 5.21+ AND
- The dataset is polygon- or line-heavy AND
- File size matters (large national/state datasets, static hosting bandwidth)

Avoid MLT for:

- Point-only datasets (compression gain is small because coordinates dominate)
- Pipelines that involve Mapbox GL JS < 3.21, deck.gl, Folium, or other clients without MLT decoders
- Anything you want to be portable across viewer ecosystems

## Switching Format

```r
# Default - emits MVT, no argument required
freestile(nc, "nc.pmtiles", layer_name = "counties")

# Opt in to MLT
freestile(nc, "nc_mlt.pmtiles", layer_name = "counties", tile_format = "mlt")
```

The same flag is available on `freestile_file()` and `freestile_query()`.

## How MLT Compresses

MLT replaces MVT's row-by-row protobuf layout with a columnar layout, then applies:

- delta encoding for coordinates
- run-length encoding for repeated attribute values
- dictionary encoding for low-cardinality string columns

In practice this yields roughly:

- ~17% size reduction on NC counties (polygon)
- ~16% size reduction on US block groups (polygon)
- minimal reduction on point-only datasets

The freestiler MLT encoder is spec-compliant against `mlt-core` 0.1.2.

## Ecosystem Status (current as of 0.1.6)

- Encoders: freestiler, mlt-core
- Decoders: mapgl (via MapLibre GL JS 5.21+, experimental), mlt-core
- Mapbox GL JS, deck.gl, Folium, leaflet plugins: not yet
