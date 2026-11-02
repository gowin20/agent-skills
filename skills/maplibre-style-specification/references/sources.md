# Sources

`sources` is an object that maps source IDs to source definitions. A source supplies data; it renders nothing until a [layer](layers.md) references it. One source can feed many layers, which is how a single road dataset becomes casing, fill, and label layers.

```json
"sources": {
  "maplibre-demotiles": { "type": "vector", "url": "https://demotiles.maplibre.org/tiles/tiles.json" },
  "streets": {
    "type": "vector",
    "tiles": ["https://a.example.com/tiles/{z}/{x}/{y}.pbf", "https://b.example.com/tiles/{z}/{x}/{y}.pbf"],
    "maxzoom": 14
  },
  "wms-imagery": {
    "type": "raster",
    "tiles": ["https://example.com/wms?bbox={bbox-epsg-3857}&format=image/png&service=WMS&version=1.1.1&request=GetMap&srs=EPSG:3857&width=256&height=256&layers=example"],
    "tileSize": 256
  }
}
```

## Contents

- [Tiled sources: shared properties](#tiled-sources-shared-properties)
- [vector](#vector)
- [raster](#raster)
- [raster-dem](#raster-dem)
- [geojson](#geojson)
- [image](#image)
- [video](#video)
- [TileJSON](#tilejson)
- [Feature IDs and promoteId](#feature-ids-and-promoteid)
- [Which layer types can use which source](#which-layer-types-can-use-which-source)

## Tiled sources: shared properties

`vector`, `raster`, and `raster-dem` sources share these properties. Give **either** `url` (a TileJSON document) **or** `tiles` (URL templates). Values set on the source override the same key in the TileJSON.

| Property | Type | Default | Notes |
|---|---|---|---|
| `url` | string | | URL of a TileJSON resource (`http:` or `https:`; renderers may register other protocols such as `pmtiles://`). |
| `tiles` | array of strings | | One or more tile URL templates. |
| `bounds` | `[w, s, e, n]` | `[-180, -85.051129, 180, 85.051129]` | No tiles are requested outside these bounds. |
| `minzoom` | number | `0` | Lowest zoom with tiles. |
| `maxzoom` | number | `22` | Highest zoom with tiles. At higher map zooms the renderer **overzooms** tiles from this level. Set it to the real maximum of the tileset; setting it higher causes 404s, setting it lower wastes detail. |
| `scheme` | `xyz` \| `tms` | `xyz` | `tms` flips the y axis (row counts from the bottom). Not on `raster-dem`. |
| `attribution` | string | | Attribution HTML shown by the map's attribution control. |
| `volatile` | boolean | `false` | Native only: do not cache tiles locally. |
| `emptyResponse` | `blank` \| `missing` | `blank` | **Experimental.** What an empty tile (HTTP 204, or nothing in an archive like PMTiles) means: `blank` = really empty here; `missing` = no tile at this zoom, so a tile from another zoom may be shown instead. |

### Tile URL tokens

| Token | Replaced with |
|---|---|
| `{z}`, `{x}`, `{y}` | Tile zoom, column, row. With `scheme: "tms"`, the row counts from the bottom. |
| `{quadkey}` | Bing Maps quadkey. |
| `{bbox-epsg-3857}` | Tile bounds in EPSG:3857 meters, `minX,minY,maxX,maxY`, for WMS. |
| `{bbox}` | Experimental (GL JS 6.15.0): bounds in the map projection's units. |
| `{prefix}` | Two hex digits from column and row, to spread requests across hosts. |
| `{ratio}` | `@2x` when the device pixel ratio is above 1, otherwise empty. |

## vector

Vector tiles in [Mapbox Vector Tile](https://github.com/mapbox/vector-tile-spec) format (default) or [MapLibre Tile](https://github.com/maplibre/maplibre-tile-spec) format.

| Property | Type | Default | Notes |
|---|---|---|---|
| `type` | `"vector"` | | Required. |
| (shared tiled properties) | | | See above. |
| `encoding` | `mvt` \| `mlt` | `mvt` | `mlt` needs GL JS 5.12.0, Android 12.1.0, iOS 6.20.0. |
| `promoteId` | string or object | | Property to use as feature ID for feature state. See [below](#feature-ids-and-promoteid). |

Every layer that uses a vector source **must** set `source-layer` to the name of a layer inside the tiles. Find the names in the TileJSON's `vector_layers` array (each entry has an `id` and its `fields`), or inspect a tile. Common schemas: OpenMapTiles (`water`, `landcover`, `landuse`, `transportation`, `building`, `place`, `poi`, `boundary`...), Shortbread, and Protomaps basemaps.

Coordinates in a tile must lie between `-1 * extent` and `(extent * 2) - 1`. Features render only inside their own tile (plus its buffer), so very wide lines, large circles, or big offsets can be clipped at tile edges; reduce the value or generate tiles with a larger buffer.

```json
"openmaptiles": { "type": "vector", "url": "https://example.com/tiles/v3.json" }
```

## raster

Raster image tiles (PNG, JPEG, WebP) or WMS.

| Property | Type | Default | Notes |
|---|---|---|---|
| `type` | `"raster"` | | Required. |
| (shared tiled properties) | | | See above. |
| `tileSize` | number | `512` | Pixels. The minimum visual size of a tile. **Most XYZ raster services (OpenStreetMap, many WMTS) serve 256 px tiles; set `"tileSize": 256`** or imagery and its labels look too small. |

```json
"osm": {
  "type": "raster",
  "tiles": ["https://tile.openstreetmap.org/{z}/{x}/{y}.png"],
  "tileSize": 256,
  "maxzoom": 19,
  "attribution": "&copy; OpenStreetMap contributors"
}
```

## raster-dem

Elevation encoded in RGB tiles, for `hillshade` and `color-relief` layers and for `terrain`.

| Property | Type | Default | Notes |
|---|---|---|---|
| `type` | `"raster-dem"` | | Required. |
| (shared tiled properties) | | | No `scheme`. |
| `tileSize` | number | `512` | Pixels. |
| `encoding` | `mapbox` \| `terrarium` \| `custom` | `mapbox` | `mapbox`: Terrain-RGB, height = -10000 + (R·256² + G·256 + B) × 0.1. `terrarium`: height = (R·256 + G + B/256) - 32768. `custom`: uses the factors below. |
| `redFactor` | number | `1.0` | Multiplies the red channel. Custom encoding only (GL JS 3.4.0+; not in Native). |
| `greenFactor` | number | `1.0` | Multiplies the green channel. Custom encoding only. |
| `blueFactor` | number | `1.0` | Multiplies the blue channel. Custom encoding only. |
| `baseShift` | number | `0.0` | Offset combined with the weighted channel sum. Custom encoding only. |

The validator rejects `redFactor`, `greenFactor`, `blueFactor`, and `baseShift` unless `encoding` is `custom`. The spec text says `baseShift` is "added to the encoding mix", but the built-in encodings correspond to a shift that is *subtracted*: `mapbox` matches factors 6553.6 / 25.6 / 0.1 with shift 10000, and `terrarium` matches 256 / 1 / 1/256 with shift 32768. Check a custom encoding against a known elevation before relying on it.

```json
"dem": {
  "type": "raster-dem",
  "tiles": ["https://s3.amazonaws.com/elevation-tiles-prod/terrarium/{z}/{x}/{y}.png"],
  "encoding": "terrarium",
  "tileSize": 256,
  "maxzoom": 15
}
```

## geojson

GeoJSON data, from a URL or inline. The renderer tiles it on the fly (with geojson-vt or supercluster).

| Property | Type | Default | Notes |
|---|---|---|---|
| `type` | `"geojson"` | | Required. |
| `data` | URL or GeoJSON | | **Required.** A URL (same origin or served with CORS headers) or an inline `Feature`, `FeatureCollection`, or geometry. Coordinates are `[lng, lat]` in WGS84. |
| `maxzoom` | number | `18` | Highest zoom at which tiles are generated. Raise it for very detailed data. |
| `buffer` | 0-512 | `128` | Tile buffer on each side. Larger = fewer edge artifacts, slower. |
| `tolerance` | number | `0.375` | Douglas-Peucker simplification. Higher = simpler, faster. |
| `filter` | expression | | Filter applied before tiling. |
| `cluster` | boolean | `false` | Cluster point features. |
| `clusterRadius` | number ≥ 0 | `50` | Cluster radius; 512 = one tile width. |
| `clusterMaxZoom` | number | `maxzoom - 1` | Last zoom with clusters. Clusters re-evaluate at integer zooms. |
| `clusterMinPoints` | number | `2` | Minimum points to form a cluster. |
| `clusterProperties` | object | | Aggregate properties on clusters (see below). |
| `lineMetrics` | boolean | `false` | Compute line distances. **Required for `line-gradient`.** |
| `generateId` | boolean | `false` | Set each feature's `id` to its index in `features`, overwriting existing IDs. |
| `promoteId` | string | | Property to use as feature ID. |
| `attribution` | string | | |

GeoJSON layers must **not** set `source-layer`.

### Clustering

With `cluster: true`, clusters become `Point` features with these properties: `cluster` (`true`), `cluster_id`, `point_count`, and `point_count_abbreviated` (e.g. `"1.2k"`). Unclustered points keep their own properties.

```json
"sources": {
  "earthquakes": {
    "type": "geojson",
    "data": "https://example.com/earthquakes.geojson",
    "cluster": true,
    "clusterRadius": 50,
    "clusterMaxZoom": 14,
    "clusterProperties": {
      "maxMag": ["max", ["get", "mag"]],
      "sumMag": ["+", ["get", "mag"]]
    }
  }
},
"layers": [
  {
    "id": "clusters", "type": "circle", "source": "earthquakes",
    "filter": ["has", "point_count"],
    "paint": {
      "circle-color": ["step", ["get", "point_count"], "#51bbd6", 100, "#f1f075", 750, "#f28cb1"],
      "circle-radius": ["step", ["get", "point_count"], 20, 100, 30, 750, 40]
    }
  },
  {
    "id": "cluster-count", "type": "symbol", "source": "earthquakes",
    "filter": ["has", "point_count"],
    "layout": { "text-field": ["get", "point_count_abbreviated"], "text-font": ["Open Sans Semibold"], "text-size": 12 }
  },
  {
    "id": "unclustered", "type": "circle", "source": "earthquakes",
    "filter": ["!", ["has", "point_count"]],
    "paint": { "circle-color": "#11b4da", "circle-radius": 4 }
  }
]
```

`clusterProperties` entries have the form `{"name": [operator, map_expression]}`. `operator` is any expression taking at least two operands (`"+"`, `"max"`, `"min"`, `"all"`...) and accumulates values; `map_expression` produces the value for a single point. For a custom reducer, use `["accumulated"]`:

```json
"clusterProperties": { "sum": [["+", ["accumulated"], ["get", "sum"]], ["get", "scalerank"]] }
```

## image

One image placed on the map.

| Property | Type | Notes |
|---|---|---|
| `type` | `"image"` | Required. |
| `url` | string | Image URL. Optional: without it the source is empty until an image is set at runtime. |
| `coordinates` | 4 × `[lng, lat]` | **Required.** Corners in clockwise order: top left, top right, bottom right, bottom left. |

```json
"radar": {
  "type": "image",
  "url": "https://example.com/radar.gif",
  "coordinates": [[-80.425, 46.437], [-71.516, 46.437], [-71.516, 37.936], [-80.425, 37.936]]
}
```

Draw it with a `raster` layer.

## video

| Property | Type | Notes |
|---|---|---|
| `type` | `"video"` | Required. |
| `urls` | array of strings | **Required.** One `<source>` element per URL; list several formats for browser coverage. |
| `coordinates` | 4 × `[lng, lat]` | **Required.** Same corner order as `image`. |

Draw it with a `raster` layer. The layer's `raster-fade-duration` makes the video fade in on play, pause, resume, and coordinate changes; set it to `0` to avoid that.

## TileJSON

A `vector`, `raster`, or `raster-dem` source can point `url` at a [TileJSON 3.0.0](https://github.com/mapbox/tilejson-spec/tree/master/3.0.0) document. MapLibre reads `tiles`, `minzoom`, `maxzoom`, `bounds`, `scheme`, `attribution`, and `vector_layers`, and extends TileJSON with these keys:

| Key | Sources | Meaning |
|---|---|---|
| `tileSize` | raster, raster-dem | Tile pixel size, as on the source. |
| `encoding` | vector (`mvt`/`mlt`), raster-dem (`mapbox`/`terrarium`/`custom`) | Tile encoding, as on the source. |
| `emptyResponse` | vector, raster, raster-dem | Experimental; as on the source. |

A value set on the source in the style takes precedence over the same key in the TileJSON.

## Feature IDs and promoteId

[`feature-state`](expressions.md#feature-data) and runtime calls like GL JS `map.setFeatureState()` identify features by ID.

- Without `promoteId`, a feature's `id` must be an integer, or a string that can be cast to one.
- `promoteId` uses a property as the ID; the value may be any primitive. For a vector source, a string applies to every source layer; an object sets it per source layer:

```json
"promoteId": { "buildings": "osm_id", "pois": "poi_id" }
```

- For GeoJSON without IDs, set `"generateId": true`.

## Which layer types can use which source

| Source type | Layer types |
|---|---|
| `vector`, `geojson` | `fill`, `line`, `symbol`, `circle`, `heatmap`, `fill-extrusion` |
| `raster`, `image`, `video` | `raster` |
| `raster-dem` | `hillshade`, `color-relief` (and root `terrain`) |
| none | `background` |
