---
name: maplibre-style-specification
description: >
  Use this skill when writing, editing, debugging, validating, or explaining a
  MapLibre style document (style.json, version 8), the JSON that MapLibre GL JS
  and MapLibre Native render. It covers the root properties, sources (vector,
  raster, raster-dem, geojson, image, video, TileJSON, clustering, promoteId),
  layers (fill, line, symbol, circle, heatmap, fill-extrusion, raster,
  hillshade, color-relief, background) with every paint and layout property,
  filters, and expressions (get, match, case, step, interpolate, zoom,
  feature-state, global-state, format, image, and the rest), plus glyphs and
  font stacks, sprites and icons, light, sky, terrain, projection, and
  transitions. Use it for data-driven or zoom-dependent styling, labels that
  don't show up, missing icons, choropleths, clusters, heatmaps, 3D buildings,
  hillshading, or migrating legacy functions and filters, even if the user only
  says "style this layer", "make a basemap", or "my map is blank". Also fits
  Mapbox GL v8 styles, which share the core format.
license: MIT
metadata:
  title: MapLibre Style Specification
  sources: https://maplibre.org/maplibre-style-spec/ ; https://github.com/maplibre/maplibre-style-spec (src/reference/v8.json and docs/, npm @maplibre/maplibre-gl-style-spec 26.5.1, November 2026)
  version: "1.0"
---

# MapLibre Style Specification

A MapLibre **style** is a JSON document that tells a renderer what data to draw and how to draw it. MapLibre GL JS (web) and MapLibre Native (Android, iOS, desktop) both read it. The current and only supported version is **8**. The normative definition is `src/reference/v8.json` in the [maplibre-style-spec](https://github.com/maplibre/maplibre-style-spec) repository; the [website](https://maplibre.org/maplibre-style-spec/) is generated from it.

## The style document

```json
{
  "version": 8,
  "name": "My style",
  "glyphs": "https://demotiles.maplibre.org/font/{fontstack}/{range}.pbf",
  "sprite": "https://demotiles.maplibre.org/styles/osm-bright-gl-style/sprite",
  "center": [0, 20],
  "zoom": 1.5,
  "sources": {
    "maplibre": { "type": "vector", "url": "https://demotiles.maplibre.org/tiles/tiles.json" }
  },
  "layers": [
    { "id": "background", "type": "background", "paint": { "background-color": "#d8e8f0" } },
    {
      "id": "countries", "type": "fill", "source": "maplibre", "source-layer": "countries",
      "paint": { "fill-color": "#f2efe9", "fill-outline-color": "#9e9e9e" }
    }
  ]
}
```

Only `version`, `sources`, and `layers` are required. The mental model:

1. **Sources** say *where data comes from*. Each has an ID (its key in `sources`) and a `type`. A source draws nothing by itself.
2. **Layers** say *how to draw* one source's features. Each layer names a `source` (and, for vector tiles, a `source-layer`), may `filter` the features, and sets `layout` and `paint` properties. The `layers` array is the **draw order**: the first layer is drawn at the bottom and the last on top.
3. **Expressions** compute any property value (and filters) from zoom, feature properties, feature state, or global state.
4. **Glyphs** (font PBFs) and the **sprite** (icon atlas) are shared resources that symbol layers and patterns draw from.

### Root properties

| Property | Type | Purpose |
|---|---|---|
| `version` | `8` | **Required.** Must be the number 8. |
| `sources` | object | **Required.** Map of source ID to source object. |
| `layers` | array | **Required.** Layer objects in draw order. |
| `name` | string | Human-readable name. |
| `metadata` | any | Free-form; does not affect rendering. Prefix keys (`"myapp:foo"`). |
| `center`, `zoom`, `bearing`, `pitch`, `roll`, `centerAltitude` | numbers | Initial camera, used only if the map is not positioned by other means (map options win). `center` is `[lng, lat]`. |
| `glyphs` | string | URL template for SDF font PBFs; must contain `{fontstack}` and `{range}`. See [Glyphs](#glyphs-and-fonts). |
| `sprite` | string or array | Base URL(s) of the icon/pattern atlas. See [Sprites](#sprites). |
| `font-faces` | object | Font files (TTF etc.) for complex scripts, keyed by `text-font` names. |
| `state` | object | Defaults for `global-state` expressions: `{"name": {"default": value}}`. |
| `light` | object | Light for `fill-extrusion` layers (`anchor`, `position`, `color`, `intensity`). |
| `sky` | object | Sky, horizon, fog, and atmosphere colors and blends. |
| `terrain` | object | 3D terrain: `{"source": "<raster-dem id>", "exaggeration": 1}`. |
| `projection` | object | `{"type": "mercator"}`, `"vertical-perspective"`, the preset `"globe"`, or a zoom expression. |
| `transition` | object | Default `{"duration": 300, "delay": 0}` (ms) for property changes. |

Read [references/root-properties.md](references/root-properties.md) for full details of `light`, `sky`, `terrain`, `projection`, `state`, `font-faces`, `transition`, and the camera fields.

## Sources

| `type` | Data | Key properties |
|---|---|---|
| `vector` | Vector tiles (MVT, or MLT with `"encoding": "mlt"`) | `url` (TileJSON) **or** `tiles` (`[".../{z}/{x}/{y}.pbf"]`), `minzoom`, `maxzoom`, `bounds`, `scheme`, `promoteId`, `attribution` |
| `raster` | Image tiles (PNG/JPEG/WebP, WMS) | `url` or `tiles`, `tileSize` (default **512**), `minzoom`, `maxzoom`, `scheme` |
| `raster-dem` | Elevation tiles | `url` or `tiles`, `encoding` (`mapbox` default, `terrarium`, `custom`), `tileSize` |
| `geojson` | GeoJSON URL or inline object | `data` (**required**), `cluster`, `clusterRadius`, `clusterMaxZoom`, `clusterProperties`, `lineMetrics`, `generateId`, `promoteId`, `maxzoom` (18), `buffer`, `tolerance`, `filter` |
| `image` | One georeferenced image | `url`, `coordinates` (4 corners, clockwise from top-left, `[lng, lat]`) |
| `video` | Georeferenced video | `urls` (array, by format preference), `coordinates` |

Tile URL tokens: `{z}`, `{x}`, `{y}`, `{quadkey}`, `{bbox-epsg-3857}` (for WMS), `{prefix}`, `{ratio}`. A layer that uses a vector source **must** set `source-layer` to a layer name inside the tiles (find them in the TileJSON's `vector_layers`). Read [references/sources.md](references/sources.md) for every property, TileJSON keys, clustering, feature IDs, and examples of each type.

## Layers

```json
{
  "id": "roads-major",
  "type": "line",
  "source": "openmaptiles",
  "source-layer": "transportation",
  "minzoom": 5,
  "filter": ["match", ["get", "class"], ["motorway", "trunk"], true, false],
  "layout": { "line-join": "round", "line-cap": "round" },
  "paint": {
    "line-color": "#e892a2",
    "line-width": ["interpolate", ["exponential", 1.5], ["zoom"], 5, 0.5, 18, 24]
  }
}
```

| Field | Notes |
|---|---|
| `id` | **Required**, unique. |
| `type` | **Required**: `background`, `fill`, `line`, `symbol`, `circle`, `heatmap`, `fill-extrusion`, `raster`, `hillshade`, `color-relief`. |
| `source` | Source ID. Required for every type except `background`. |
| `source-layer` | Required for `vector` sources; **not allowed** for any other source type, including GeoJSON. |
| `minzoom` / `maxzoom` | 0-24. Hidden below `minzoom` and **at or above** `maxzoom`. |
| `filter` | Boolean expression; only matching features are drawn. |
| `layout` | Properties evaluated when tiles are laid out (placement, text, caps, `visibility`). |
| `paint` | Properties applied at draw time (color, opacity, width). Cheap to change. |
| `metadata` | Free-form. |

Choosing a type: polygons → `fill` (+ a `line` layer for real strokes; `fill-outline-color` is only 1 px); lines → `line`; points → `circle` or `symbol`; labels and icons → `symbol`; point density → `heatmap`; 3D buildings → `fill-extrusion`; imagery → `raster`; terrain shading → `hillshade` or `color-relief` on a `raster-dem` source.

Read [references/layers.md](references/layers.md) for per-type guidance, draw order, filters, symbol placement and collision, feature state, and transitions. Read [references/layer-properties.md](references/layer-properties.md) for the generated table of **every** layout and paint property with type, default, range, units, expression support, and SDK support; open only the section for the layer type you need.

## Expressions

An expression is a JSON array whose first element is an operator name: `["operator", arg1, arg2, ...]`. Arguments are literals or nested expressions. Expressions work in any `layout` or `paint` property and in `filter`.

```json
"circle-color": ["match", ["get", "kind"], "cafe", "#c47a3a", ["bar", "pub"], "#7a3ac4", "#888888"],
"circle-radius": ["interpolate", ["linear"], ["zoom"], 8, 2, 16, ["*", 2, ["sqrt", ["get", "seats"]]]],
"fill-opacity": ["case", ["boolean", ["feature-state", "hover"], false], 0.9, 0.5],
"text-field": ["coalesce", ["get", "name:en"], ["get", "name"]]
```

The operators you will use most:

| Need | Operator |
|---|---|
| Read a feature property | `["get", "name"]`, `["has", "name"]`, `["id"]`, `["geometry-type"]` |
| Categorical mapping | `["match", input, label, out, ..., fallback]` (labels must be literals) |
| Conditions | `["case", cond, out, ..., fallback]`, `==`, `!=`, `<`, `>`, `all`, `any`, `!`, `in` |
| Classed (stepped) values | `["step", input, out0, stop1, out1, ...]` |
| Smooth ramps | `["interpolate", ["linear"], input, stop, out, ...]`; also `["exponential", base]`, `["cubic-bezier", x1, y1, x2, y2]`, `interpolate-hcl`, `interpolate-lab` |
| Zoom | `["zoom"]` (see the rule below) |
| First non-null | `["coalesce", a, b, ...]` |
| Math | `+ - * / % ^ sqrt abs round floor ceil min max ln log10 log2 sin cos tan`... |
| Strings | `concat`, `upcase`, `downcase`, `slice`, `split`, `join`, `index-of`, `number-format`, `to-string` |
| Types | `literal`, `number`, `string`, `boolean`, `to-number`, `to-color`, `to-boolean` |
| Rich labels / images | `["format", ...]`, `["image", "name"]` |
| Interactive state | `["feature-state", "hover"]` (paint only), `["global-state", "key"]` |
| Spatial | `["within", geojson]`, `["distance", geojson]` |
| Variables | `["let", "v", value, ["var", "v"]]` |

**Rules that cause most expression errors:**

- `["zoom"]` may appear only as the input of a **top-level** `interpolate` or `step` (optionally wrapped in `let`). Put data inside the stop outputs, not around the zoom curve.
- Array literals inside an expression need `["literal", [...]]`: `["in", ["get", "class"], ["literal", ["a", "b"]]]`.
- Stop inputs in `step` and `interpolate` must be number literals in strictly ascending order.
- Comparisons are strictly typed: `"5"` does not equal `5`. Convert with `to-number` or `to-string`, or assert with `number` or `string`. When the types do not match, `match` returns its fallback, and a failed type check makes the property use its default.
- `feature-state` works only in paint properties that support data-driven styling, never in filters or layout.
- Layout properties re-evaluate zoom only at integer zoom levels; paint properties re-evaluate continuously.

Read [references/expressions.md](references/expressions.md) for every operator with its signature, behavior, and examples, plus recipes (choropleth, graduated circles, label fallbacks, hover).

## Glyphs and fonts

Symbol layers with `text-field` need glyphs. Set the root `glyphs` to an **absolute** URL with both tokens:

```json
"glyphs": "https://demotiles.maplibre.org/font/{fontstack}/{range}.pbf"
```

- `{fontstack}` becomes the layer's `text-font` array joined with commas, e.g. `["Open Sans Semibold", "Arial Unicode MS Regular"]` → `Open Sans Semibold,Arial Unicode MS Regular`.
- `{range}` becomes a 256-code-point block such as `0-255`. The renderer fetches only the blocks the visible text needs.
- Font names in `text-font` must match the font names on the glyph server exactly. The default `text-font` is `["Open Sans Regular", "Arial Unicode MS Regular"]`; if your server lacks those, labels silently fail to load.
- Glyph PBFs are signed-distance fields, so one file serves every size, color, and halo.
- `font-faces` maps a `text-font` name to TTF/OTF files with optional `unicode-range`, for complex scripts (Devanagari, Khmer, Myanmar). Characters not covered fall back to `glyphs`.
- In GL JS 5.11.0+ you may omit `glyphs` and use fonts installed locally; Native requires `glyphs`.

Read [references/glyphs-and-fonts.md](references/glyphs-and-fonts.md) for font stack resolution, building and hosting glyph PBFs, CJK and right-to-left text, `font-faces`, and troubleshooting.

## Sprites

A sprite is a PNG atlas plus a JSON index. Give the **base URL without an extension**; the renderer appends `.json` and `.png`, and `@2x.json`/`@2x.png` on high-DPI screens:

```json
"sprite": "https://demotiles.maplibre.org/styles/osm-bright-gl-style/sprite"
```

The index maps image names to rectangles: `{"airport_11": {"x": 0, "y": 21, "width": 17, "height": 17, "pixelRatio": 1}}`. Optional keys: `sdf` (recolor with `icon-color` and `icon-halo-color`), `stretchX`, `stretchY`, `content` (for `icon-text-fit` shields), and `textFitWidth`/`textFitHeight`.

Reference images by name in `icon-image`, `fill-pattern`, `line-pattern`, `fill-extrusion-pattern`, and `background-pattern`, e.g. `"icon-image": ["concat", ["get", "class"], "_11"]`. With several sprites, use an array of `{"id", "url"}` objects and prefix names with `id:` (`"roadsigns:stop"`). The sprite with id `default` needs no prefix.

Read [references/sprites.md](references/sprites.md) for the full index format, stretchable shields, SDF icons, multiple sprites, generating sprites with spreet, and runtime images.

## Default procedure for writing or changing a style

1. **Inventory the data.** For each source, list the source-layer names and attribute fields (TileJSON `vector_layers`, or the GeoJSON properties). Note value types: is `population` a number or a string?
2. **Declare sources** with the right `type`. For raster tiles that are 256 px, set `"tileSize": 256`.
3. **Plan the layer stack** from bottom to top: background, landcover and water fills, hillshade, lines (casings before the roads they surround), buildings, points, then labels last.
4. **Write each layer**: `id`, `type`, `source`, `source-layer`, `filter`, then `layout` and `paint`. Put each property in its correct object (`layout` vs `paint`); the property table says which.
5. **Add expressions** for zoom and data styling. Use `match` for categories, `step` for classes, and `interpolate` for continuous ramps.
6. **Wire up shared resources**: `glyphs` if any layer has `text-field`, `sprite` if any layer uses icons or patterns. Check that every `text-font` and image name exists.
7. **Validate** (see below) and fix every error, then view the map at several zoom levels.

## Gotchas

- **Vector layers without `source-layer` draw nothing.** The spec also forbids `source-layer` on GeoJSON layers; the validator does not flag it, but the renderer then looks for a tile layer that does not exist and draws nothing.
- **Source and layer types must pair up**: `raster` sources feed only `raster` layers; `raster-dem` sources feed only `hillshade` and `color-relief` layers (and `terrain`); `vector` and `geojson` sources feed the other types.
- **Label priority**: symbol layers later in `layers` are placed first and win collisions. Within a layer, lower `symbol-sort-key` values are placed first. Use `text-allow-overlap`/`icon-allow-overlap` (or `*-overlap`) and `*-ignore-placement` to force placement.
- **Blank labels** usually mean a missing `glyphs` URL, a `text-font` name the server does not have, or a `text-field` that resolves to an empty string or null.
- **Missing icons** usually mean the sprite URL includes `.json`/`.png`, the name is misspelled or lacks its `id:` prefix, or the image was never added. GL JS logs `Image "<name>" could not be loaded`.
- **Legacy syntax still parses but is ambiguous.** `["==", "class", "park"]` is a legacy filter (key, value); in expression form write `["==", ["get", "class"], "park"]`. Do not mix the two forms in one filter. `{stops: ...}` functions and `"{name}"` tokens are deprecated; use expressions.
- **`maxzoom` is exclusive** on layers. On sources, `maxzoom` is the highest zoom of the tiles; the map overzooms beyond it, so do not set it higher than the data really has.
- **Pattern images** must have power-of-two widths and heights (2, 4, ..., 512) to tile seamlessly.
- **`fill-outline-color`** draws only a 1 px antialiased edge; use a separate `line` layer for thicker strokes.
- **`line-gradient`** needs a GeoJSON source with `"lineMetrics": true` and cannot be combined with `line-dasharray` or `line-pattern`.
- **Feature state needs feature IDs**: an integer `id` (or a string castable to one), `generateId: true` on GeoJSON, or `promoteId`.
- **`text-halo-width`** is capped at a quarter of the font size.
- **Colors** are strings: hex, `rgb()`, `rgba()`, `hsl()`, `hsla()`, or CSS names. Strings turn into colors automatically only where a color is expected.
- **Not every renderer supports every property.** Newer properties (for example `fill-layer-blend`, `fill-layer-opacity`, `symbol-height-offset`) may be missing from MapLibre Native or GL JS. Check the JS/Native columns in the property table.
- **Terrain and hillshade** both need a `raster-dem` source. GL JS recommends separate source objects for `terrain` and for `hillshade` layers, even if they point at the same tiles.
- **Mapbox styles** share the format, but `mapbox://` URLs, Mapbox-only expressions (`config`, `measure-light`), `fog`, `imports`, and `slot` are not part of MapLibre. Replace them before use.

## Validation

Validate with the official CLI (Node.js required):

```bash
npx --package @maplibre/maplibre-gl-style-spec gl-style-validate style.json        # human-readable
npx --package @maplibre/maplibre-gl-style-spec gl-style-validate --json style.json # JSON errors
npx --package @maplibre/maplibre-gl-style-spec gl-style-format style.json          # normalize formatting
npx --package @maplibre/maplibre-gl-style-spec gl-style-migrate old.json > new.json # v7 to v8, legacy syntax to expressions
```

In JavaScript, `import {validate} from '@maplibre/maplibre-gl-style-spec'` returns an array of errors (empty if valid). No output and exit code 0 mean the style is valid; otherwise each error is printed as `file:line: path: message`. The validator catches unknown properties (a paint property placed in `layout` is reported as `unknown property`), wrong types, bad enums, unknown source IDs, source/layer type mismatches, missing `source-layer` on vector layers, `line-gradient` without `lineMetrics`, glyph URLs without both tokens, and malformed expressions. It does **not** check that tile URLs, source-layer names, fonts, or sprite images exist; check those by viewing the map and reading the console.

If you cannot run the validator, check by hand: `version` is 8; every layer has a unique `id`; every `source` exists; vector layers have `source-layer`; no other layers do; properties are in the right object; enums are spelled exactly; every `["zoom"]` sits at the top level of an `interpolate` or `step`; array literals are wrapped in `literal`.

## Resources

- [references/root-properties.md](references/root-properties.md): read for camera defaults, `light`, `sky`, `terrain`, `projection`, `state`, `font-faces`, or `transition`.
- [references/sources.md](references/sources.md): read when adding or debugging any source, TileJSON, clustering, or feature IDs.
- [references/layers.md](references/layers.md): read when choosing a layer type, ordering layers, writing filters, tuning label placement, or using feature state.
- [references/layer-properties.md](references/layer-properties.md): read for the exact name, type, default, range, or support of a paint/layout property.
- [references/expressions.md](references/expressions.md): read when writing any non-trivial expression or looking up an operator.
- [references/glyphs-and-fonts.md](references/glyphs-and-fonts.md): read when labels don't render, when hosting fonts, or for non-Latin scripts.
- [references/sprites.md](references/sprites.md): read when building, hosting, or debugging sprites, icons, shields, or patterns.
- [references/types.md](references/types.md): read for value formats such as color, padding, formatted, resolvedImage, and projectionDefinition.
- [references/deprecated-syntax.md](references/deprecated-syntax.md): read when maintaining an old style that uses `stops` functions or legacy filters.
- [assets/minimal-style.json](assets/minimal-style.json): a valid starter style on the MapLibre demo tiles, with fills, lines, labels, and an icon layer.
