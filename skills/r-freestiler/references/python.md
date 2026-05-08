# Cross-Language Tips: Python Clients and Tooling

freestiler has both an R package and a Python package; the R skill is what this repository targets, but PMTiles archives are interoperable, so cross-language workflows come up often.

## Format Choice for Python Clients

When tiles produced in R will be consumed by Python tooling (Folium, leafmap, MapLibre via Streamlit/Jupyter), prefer MVT:

```r
# R producer
freestile(data, "out.pmtiles", layer_name = "features")  # MVT default
```

The MLT decoder ecosystem is still catching up; MVT works with every PMTiles-aware Python client. If you must use MLT for size reasons, verify the destination viewer supports MLT before shipping.

## Serving PMTiles for Python Clients

PMTiles relies on HTTP byte-range requests. Python's built-in `http.server` does **not** support range requests, so it cannot serve PMTiles. Use one of:

- `serve_tiles()` from R (works for files up to ~1 GB)
- `npx http-server <dir> --cors -c-1` (Node, handles larger files better)
- A dedicated static server (Caddy, nginx) for production

```r
# R-side: start a local server, then point Python at it
serve_tiles("tiles/")  # serves http://localhost:8080/
```

Python consumer (MapLibre GL JS 5.17+ inside a Jupyter widget):

```python
# pseudo-code
m = maplibre.Map(...)
m.add_source("data", {"type": "vector", "url": "pmtiles://http://localhost:8080/out.pmtiles"})
```

## Producing Tiles in Python

The Python package mirrors the R API. The main differences:

- Multi-layer input is a Python `dict` instead of an R named list
- Pre-built wheels include GeoPandas, DuckDB, and SQL helpers
- Supports Python 3.9-3.14

```python
# Python equivalent of freestile()
import freestiler
freestiler.freestile(gdf, "out.pmtiles", layer_name="features")
```

For large datasets, the `freestile_file()` and `freestile_query()` paths skip the GeoDataFrame round-trip; reprojection to WGS84 plus geometry serialization is usually the most expensive step.

## Mixed-Language Workflow

A common pattern: tile the data in Python (where the source is, e.g. a Parquet lake), then visualise in R via mapgl. Positron IDE can run both side by side.

```r
# R consumes Python-built tiles
serve_tiles("./from_python/")
maplibre() |>
  add_pmtiles_source(id = "src", url = "http://localhost:8080/data.pmtiles") |>
  add_fill_layer(source = "src", source_layer = "features")
```
