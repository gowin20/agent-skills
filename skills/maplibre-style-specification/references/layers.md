# Layers

`layers` is an ordered array. Each layer draws features from one source with one rendering type. This file covers how layers work; for the full list of properties per type, see [layer-properties.md](layer-properties.md).

## Contents

- [Layer object](#layer-object)
- [Draw order](#draw-order)
- [Layout vs paint](#layout-vs-paint)
- [Filters](#filters)
- [Layer types](#layer-types): background, fill, line, symbol, circle, heatmap, fill-extrusion, raster, hillshade, color-relief
- [Symbol placement and collision](#symbol-placement-and-collision)
- [Feature state and interactivity](#feature-state-and-interactivity)
- [Transitions](#transitions)

## Layer object

| Field | Type | Notes |
|---|---|---|
| `id` | string | **Required**, unique within the style. Runtime APIs address layers by ID. |
| `type` | enum | **Required**. One of the ten types below. |
| `source` | string | Source ID. Required for every type except `background`. |
| `source-layer` | string | Layer inside a vector tile. **Required** for `vector` sources; **forbidden** for every other source type, including `geojson`. |
| `minzoom` | 0-24 | Hidden at zooms **below** this. |
| `maxzoom` | 0-24 | Hidden at zooms **equal to or above** this. |
| `filter` | expression | Boolean; only matching features render. |
| `layout` | object | Layout properties, including `visibility`. |
| `paint` | object | Paint properties. |
| `metadata` | any | Free-form, e.g. `{"source:comment": "Hydrology FCCODE 460"}`. |

## Draw order

- Layers draw in array order: index 0 at the bottom, the last layer on top. There is no z-index; reorder the array (or use GL JS `map.moveLayer` / the `beforeId` argument of `map.addLayer`).
- Within one layer, features draw in source order unless a `*-sort-key` layout property is set (`fill-sort-key`, `line-sort-key`, `circle-sort-key`, `symbol-sort-key`). Higher keys draw on top for fills, lines, and circles.
- Symbol layers are drawn in array order too, but **placement priority runs from the top down**: the last symbol layer gets first claim on space, then the one below it, and so on. Put your most important labels last.
- A typical basemap order: `background` → landcover/landuse fills → water fills and lines → hillshade → building fills or extrusions → road casings → road fills → rail → boundaries → POI icons → road labels → place labels (largest places last).

Road casings: draw the same road features twice, a wider dark `line` layer first and a narrower light `line` layer above it. Alternatively, `line-gap-width` draws two lines either side of a gap, which suits single-layer casings.

## Layout vs paint

| | `layout` | `paint` |
|---|---|---|
| When evaluated | When a tile is laid out (on load, and at each integer zoom for zoom expressions) | Every frame |
| Zoom expressions | Re-evaluated only at integer zooms | Re-evaluated continuously (smooth) |
| `feature-state` | Not allowed | Allowed where data-driven styling is supported |
| Cost to change at runtime | Re-layout of tiles | Cheap |
| Examples | `visibility`, `line-cap`, `line-join`, `text-field`, `text-size`, `icon-image`, `symbol-placement` | colors, opacities, widths, offsets, translate, halos |

Putting a property in the wrong object is a validation error, reported as `unknown property "text-color"`. `text-size` and `icon-size` are layout properties; `text-color`, `text-halo-*`, `icon-color`, `icon-opacity` are paint properties.

## Filters

`filter` takes a boolean [expression](expressions.md). Only features for which it evaluates to `true` are rendered.

```json
"filter": ["all",
  ["==", ["geometry-type"], "LineString"],
  ["match", ["get", "class"], ["motorway", "trunk", "primary"], true, false],
  ["!=", ["get", "brunnel"], "tunnel"],
  [">=", ["zoom"], 6]
]
```

- Zoom in filters is evaluated only at integer zoom levels. Prefer layer `minzoom`/`maxzoom` where possible.
- `feature-state` is **not** supported in filters.
- `["geometry-type"]` returns `Point`, `LineString`, or `Polygon` (multi-geometries return the single form).
- Use `match` with array labels, or `["in", ["get", "x"], ["literal", [...]]]`, for set membership.
- A missing property yields `null` from `get`. `["==", ["get", "x"], null]` tests for missing values; `["has", "x"]` tests for presence.
- Legacy filter syntax (`["==", "class", "park"]`, `["in", "class", "a", "b"]`, `$type`) still works; see [deprecated-syntax.md](deprecated-syntax.md). Do not mix legacy and expression forms in one filter.

## Layer types

### background

Fills the whole map. No source. Properties: `background-color`, `background-pattern` (sprite image; overrides color), `background-opacity`. Put it first.

### fill

Polygons (and the polygon parts of mixed data).

- `fill-color`, `fill-opacity`, `fill-pattern` (sprite image; disables `fill-color` for drawing, except as the foreground color of SDF patterns where supported).
- `fill-outline-color` draws a 1 px antialiased edge; it needs `fill-antialias: true` (the default) and no `fill-pattern`. For real borders, add a `line` layer on the same source.
- `fill-translate`/`fill-translate-anchor` offset the geometry in pixels.
- `fill-layer-opacity` (GL JS 6.0.0) applies one opacity to the whole layer, so overlapping polygons do not darken each other; `fill-opacity` accumulates where features overlap. `fill-layer-blend` sets a compositing mode (`normal`, `multiply`, `screen`, `overlay`, ...); it is new and not yet implemented in released renderers.

Choropleth:

```json
{
  "id": "counties", "type": "fill", "source": "census", "source-layer": "counties",
  "paint": {
    "fill-color": ["step", ["to-number", ["get", "density"]],
      "#f7fbff", 10, "#c6dbef", 50, "#6baed6", 200, "#2171b5", 1000, "#08306b"],
    "fill-opacity": 0.8
  }
}
```

### line

Lines, and polygon outlines.

- Layout: `line-cap` (`butt`, `round`, `square`), `line-join` (`miter`, `bevel`, `round`), `line-miter-limit`, `line-round-limit`, `line-sort-key`.
- Paint: `line-color`, `line-width`, `line-opacity`, `line-gap-width` (casing), `line-offset` (positive = right of the line direction), `line-blur`, `line-dasharray`, `line-pattern`, `line-gradient`, `line-translate`, `line-layer-opacity`, `line-layer-blend`.
- `line-dasharray` lengths are in multiples of the line width: `[2, 1]` = dash two widths long, gap one width. Zoom expressions work everywhere; data-driven dash arrays need GL JS 5.8.0 and are not yet in MapLibre Native.
- `line-gradient` colors along the line with `["line-progress"]` (0 at start, 1 at end) and needs a GeoJSON source with `"lineMetrics": true`:

```json
"paint": {
  "line-width": 6,
  "line-gradient": ["interpolate", ["linear"], ["line-progress"], 0, "blue", 0.5, "lime", 1, "red"]
}
```

Zoom-scaled road widths usually use exponential interpolation: `["interpolate", ["exponential", 1.5], ["zoom"], 5, 0.5, 12, 2, 18, 24]`.

### symbol

Text labels and icons, for points, along lines, or at polygon anchors. Almost everything is a layout property.

- **Placement:** `symbol-placement` (`point`, `line` = repeated along the line every `symbol-spacing` px, `line-center` = once at the midpoint).
- **Text:** `text-field` (string or `format` expression), `text-font` (font stack; see [glyphs-and-fonts.md](glyphs-and-fonts.md)), `text-size`, `text-max-width` (ems, for wrapping), `text-line-height`, `text-letter-spacing`, `text-justify`, `text-anchor`, `text-offset` (ems), `text-radial-offset`, `text-variable-anchor`, `text-variable-anchor-offset`, `text-rotate`, `text-transform`, `text-max-angle` (curved line labels), `text-keep-upright`, `text-writing-mode`, `text-padding`.
- **Icons:** `icon-image` (sprite image name or `image` expression), `icon-size` (scale factor), `icon-anchor`, `icon-offset`, `icon-rotate`, `icon-text-fit` (`width`, `height`, `both`) with `icon-text-fit-padding` (for shields), `icon-padding`, `icon-keep-upright`.
- **Alignment:** `*-rotation-alignment` and `*-pitch-alignment` (`map`, `viewport`, `auto`) control whether symbols rotate and tilt with the map.
- **Collision:** `*-allow-overlap`, `*-overlap` (`never`, `always`, `cooperative`; takes precedence over `*-allow-overlap`), `*-ignore-placement`, `*-optional`, `symbol-avoid-edges`, `symbol-sort-key`, `symbol-z-order`.
- **Paint:** `text-color`, `text-opacity`, `text-halo-color`, `text-halo-width` (max ¼ of the font size), `text-halo-blur`, `text-translate`, and the same for `icon-*`. `icon-color` and `icon-halo-*` work only with SDF icons.
- **Elevated symbols:** `symbol-height-offset` (meters) with `symbol-height-anchor` (`ground`, `absolute`), GL JS 6.6.0+.

```json
{
  "id": "place-labels", "type": "symbol", "source": "openmaptiles", "source-layer": "place",
  "filter": ["in", ["get", "class"], ["literal", ["city", "town"]]],
  "layout": {
    "text-field": ["coalesce", ["get", "name:en"], ["get", "name"]],
    "text-font": ["Open Sans Semibold"],
    "text-size": ["interpolate", ["linear"], ["zoom"], 4, 11, 10, 18],
    "text-variable-anchor": ["top", "bottom", "left", "right"],
    "text-radial-offset": 0.6,
    "text-justify": "auto",
    "icon-image": "circle_11",
    "symbol-sort-key": ["-", ["to-number", ["get", "population"], 0]]
  },
  "paint": {
    "text-color": "#333",
    "text-halo-color": "rgba(255, 255, 255, 0.9)",
    "text-halo-width": 1.5
  }
}
```

### circle

Points drawn as circles: `circle-radius`, `circle-color`, `circle-opacity`, `circle-blur`, `circle-stroke-width` (outside the radius), `circle-stroke-color`, `circle-stroke-opacity`, `circle-translate`, `circle-pitch-scale`, `circle-pitch-alignment`, `circle-sort-key`. Lines and polygons in the source render circles at their vertices, so filter to points.

Graduated circles by area: `"circle-radius": ["interpolate", ["linear"], ["sqrt", ["get", "value"]], 0, 2, 100, 30]`.

### heatmap

Point density as a continuous surface.

- `heatmap-weight` (per point, data-driven), `heatmap-intensity` (global multiplier, usually by zoom), `heatmap-radius` (px), `heatmap-opacity`.
- `heatmap-color` must be an expression on `["heatmap-density"]` (0-1). Make the 0 stop transparent.

```json
"paint": {
  "heatmap-weight": ["interpolate", ["linear"], ["get", "mag"], 0, 0, 6, 1],
  "heatmap-intensity": ["interpolate", ["linear"], ["zoom"], 0, 1, 9, 3],
  "heatmap-radius": ["interpolate", ["linear"], ["zoom"], 0, 2, 9, 20],
  "heatmap-color": ["interpolate", ["linear"], ["heatmap-density"],
    0, "rgba(33,102,172,0)", 0.2, "rgb(103,169,207)", 0.4, "rgb(209,229,240)",
    0.6, "rgb(253,219,199)", 0.8, "rgb(239,138,98)", 1, "rgb(178,24,43)"],
  "heatmap-opacity": ["interpolate", ["linear"], ["zoom"], 7, 1, 9, 0]
}
```

### fill-extrusion

Polygons extruded into 3D, lit by the root [`light`](root-properties.md#light).

- `fill-extrusion-height` and `fill-extrusion-base` in meters (base ≤ height; negative values go underground).
- `fill-extrusion-color` (can use `["elevation"]`, the height above ground, where supported), `fill-extrusion-pattern`, `fill-extrusion-opacity` (whole layer; **not** data-driven), `fill-extrusion-vertical-gradient`, `fill-extrusion-translate`.
- Layout: `fill-extrusion-rounded-corner-distance` (meters; GL JS 6.2.0, Android 13.4.0).

```json
{
  "id": "3d-buildings", "type": "fill-extrusion", "source": "openmaptiles", "source-layer": "building",
  "minzoom": 14,
  "paint": {
    "fill-extrusion-color": "#d6d0c8",
    "fill-extrusion-height": ["interpolate", ["linear"], ["zoom"], 14, 0, 14.5, ["get", "render_height"]],
    "fill-extrusion-base": ["interpolate", ["linear"], ["zoom"], 14, 0, 14.5, ["get", "render_min_height"]],
    "fill-extrusion-opacity": 0.85
  }
}
```

### raster

Draws `raster`, `image`, or `video` sources. All properties are paint and accept only zoom expressions: `raster-opacity`, `raster-hue-rotate`, `raster-brightness-min`, `raster-brightness-max`, `raster-saturation`, `raster-contrast`, `raster-fade-duration`, and `resampling` (`linear` or `nearest`; GL JS 5.20.0). The older `raster-resampling` does the same thing; do not set both.

### hillshade

Client-side shading from a `raster-dem` source.

- `hillshade-method`: `standard` (default), `basic`, `combined`, `igor`, `multidirectional`.
- `hillshade-illumination-direction` (0-359, default 335) and `hillshade-illumination-altitude` (0-90, default 45); with `multidirectional` both accept arrays for several lights, matched by `hillshade-highlight-color` and `hillshade-shadow-color` arrays.
- `hillshade-illumination-anchor` (`viewport` default, or `map`), `hillshade-exaggeration` (0-1), `hillshade-accent-color`, `resampling`.

```json
{
  "id": "hillshade", "type": "hillshade", "source": "dem",
  "paint": {
    "hillshade-method": "multidirectional",
    "hillshade-illumination-direction": [270, 315, 0],
    "hillshade-illumination-altitude": [30, 30, 30],
    "hillshade-highlight-color": ["#ffffff", "#ffffff", "#ffffff"],
    "hillshade-shadow-color": ["#5a5a5a", "#404040", "#5a5a5a"],
    "hillshade-exaggeration": 0.4
  }
}
```

### color-relief

Hypsometric tinting from a `raster-dem` source (GL JS 5.6.0, Android 13.0.0, iOS 6.24.0). `color-relief-color` must be an expression on `["elevation"]` (meters); also `color-relief-opacity` and `resampling`.

```json
"paint": {
  "color-relief-color": ["interpolate", ["linear"], ["elevation"],
    0, "#2e7d32", 500, "#c0ca33", 1500, "#a1887f", 3000, "#ffffff"],
  "color-relief-opacity": 0.6
}
```

## Symbol placement and collision

Each frame, the renderer places symbols layer by layer from the **top** of the stack down, and feature by feature within a layer. A symbol is hidden if its box collides with an already-placed symbol, unless overlap is allowed.

| Property | Effect |
|---|---|
| `text-allow-overlap` / `icon-allow-overlap` | This symbol shows even if it collides with earlier symbols. |
| `text-overlap` / `icon-overlap` | `never`, `always`, or `cooperative` (shows over symbols placed with `always`/`cooperative`, not over `never`). Overrides `*-allow-overlap`. |
| `text-ignore-placement` / `icon-ignore-placement` | Later symbols may collide with this one (it does not block others). |
| `text-optional` / `icon-optional` | If one part collides, show the other part alone. |
| `text-padding` / `icon-padding` | Extra collision margin in px. |
| `symbol-sort-key` | Lower values are placed first (higher priority) within the layer. |
| `symbol-z-order` | `auto`, `viewport-y`, or `source` draw order for overlapping symbols. |
| `symbol-avoid-edges` | Prevents symbols crossing tile edges; helps when tiles lack buffer. |

Labels that flicker or vanish when panning are usually collisions; increase priority (move the layer up, lower its sort key) or allow overlap. Labels cut off at tile edges point to too little tile buffer.

## Feature state and interactivity

Feature state is runtime data attached to a feature by ID (GL JS `map.setFeatureState({source, sourceLayer, id}, {hover: true})`). Read it in paint properties:

```json
"paint": {
  "fill-color": ["case", ["boolean", ["feature-state", "selected"], false], "#ff7f00", "#4a90d9"],
  "fill-opacity": ["case", ["boolean", ["feature-state", "hover"], false], 0.9, 0.5]
}
```

- Wrap with `["boolean", ..., false]` (or `coalesce`) because unset state is `null`.
- Paint properties only, and only those marked `state` in [layer-properties.md](layer-properties.md). Not in layout or filters.
- Features need IDs; see [sources.md](sources.md#feature-ids-and-promoteid).
- For style-wide toggles (themes, unit switches, filters driven by UI), use [`global-state`](expressions.md#lookup) instead; it works in layout, filters, and paint.

## Transitions

Transitionable paint properties (marked `T`) animate when changed at runtime, using the root `transition` or a per-property `<name>-transition` object with `duration` and `delay` in milliseconds:

```json
"paint": { "circle-radius": 6, "circle-radius-transition": { "duration": 500, "delay": 0 } }
```

Layout properties do not transition. Symbol fade-in after collision checks is controlled by the renderer (GL JS `fadeDuration`), not by `transition`.
