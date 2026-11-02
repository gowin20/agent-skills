# Root properties

All top-level keys of a style. Only `version`, `sources`, and `layers` are required. `sources` and `layers` are covered in [sources.md](sources.md) and [layers.md](layers.md), glyphs and `font-faces` in [glyphs-and-fonts.md](glyphs-and-fonts.md), and `sprite` in [sprites.md](sprites.md).

## Contents

- [Identity and metadata](#identity-and-metadata)
- [Camera defaults](#camera-defaults)
- [state](#state)
- [light](#light)
- [sky](#sky)
- [terrain](#terrain)
- [projection](#projection)
- [transition](#transition)
- [font-faces](#font-faces)

## Identity and metadata

| Key | Type | Notes |
|---|---|---|
| `version` | enum `8` | Required. Must be the number `8`, not the string. |
| `name` | string | Human-readable name, e.g. `"Bright"`. |
| `metadata` | any | Arbitrary data that does not affect rendering. Prefix keys to avoid collisions, e.g. `"maplibre:"` or `"styleeditor:comment"`. |

## Camera defaults

These set the initial view **only if the map has not been positioned by other means** (map constructor options, a URL hash, or user interaction take precedence).

| Key | Type | Default | Units / notes |
|---|---|---|---|
| `center` | `[lng, lat]` | | e.g. `[-73.9749, 40.7736]` |
| `centerAltitude` | number | | Meters above sea level of the point the camera looks at. GL JS 5.0.0+. |
| `zoom` | number | | e.g. `12.5` |
| `bearing` | number | `0` | Degrees, period 360. 90 puts east at the top. |
| `pitch` | number | `0` | Degrees. 0 looks straight down. GL JS supports 0-60 since 0.8, 0-85 since 2.0, 0-180 since 5.0; Native supports 0-60. |
| `roll` | number | `0` | Degrees counterclockwise about the view axis. GL JS 5.0.0+. |

## state

Default values for the [`global-state`](expressions.md#lookup) expression. Each key maps to an object with a `default`:

```json
"state": {
  "chargerType": { "default": ["CCS", "CHAdeMO", "Type2"] },
  "minPreferredChargingSpeed": { "default": 50 },
  "showLabels": { "default": true }
}
```

Use it from layers, then change it at runtime with the platform API (GL JS `map.setGlobalStateProperty("showLabels", false)`):

```json
"layout": { "visibility": ["case", ["global-state", "showLabels"], "visible", "none"] },
"filter": [">=", ["get", "kw"], ["global-state", "minPreferredChargingSpeed"]]
```

`global-state` returns `null` when neither a runtime value nor a default exists. Support: GL JS 5.6.0, Native Android 13.7.1, iOS 7.0.0. In `visibility` it needs GL JS 5.16.0.

## light

The global light, used to shade `fill-extrusion` layers.

| Key | Type | Default | Notes |
|---|---|---|---|
| `anchor` | `map` \| `viewport` | `viewport` | `map`: light rotates with the map. `viewport`: fixed to the screen. |
| `position` | `[r, a, p]` | `[1.15, 210, 30]` | Radial distance, azimuth (degrees clockwise from the top of the viewport or from north), and polar angle (0 = directly above, 180 = directly below). Transitionable. |
| `color` | color | `#ffffff` | Tint. Transitionable. |
| `intensity` | number 0-1 | `0.5` | Higher means more contrast. Transitionable. |

```json
"light": { "anchor": "viewport", "color": "white", "intensity": 0.4 }
```

All light properties accept zoom expressions (`interpolate`/`step` on `["zoom"]`), but not data expressions.

## sky

Sky, horizon, fog, and atmosphere. Marked **experimental** in the spec; implemented in GL JS 4.5.0+ and not yet in Native.

| Key | Type | Default | Notes |
|---|---|---|---|
| `sky-color` | color | `#88C6FC` | Base sky color. |
| `horizon-color` | color | `#ffffff` | Color at the horizon. |
| `fog-color` | color | `#ffffff` | Fog color. Requires 3D terrain. |
| `fog-ground-blend` | 0-1 | `0.5` | Fog over terrain: 0 = map center, 1 = horizon. |
| `horizon-fog-blend` | 0-1 | `0.8` | 0 = horizon color only, 1 = fog color only. |
| `sky-horizon-blend` | 0-1 | `0.8` | 1 = blend at mid-sky, 0 = sky color only. |
| `atmosphere-blend` | 0-1 | `0.8` | Atmosphere glow around the globe: 1 visible, 0 hidden. Interpolate it by zoom with the globe projection. |

```json
"sky": {
  "sky-color": "#199EF3",
  "sky-horizon-blend": 0.5,
  "horizon-color": "#ffffff",
  "horizon-fog-blend": 0.5,
  "fog-color": "#0000ff",
  "fog-ground-blend": 0.5,
  "atmosphere-blend": ["interpolate", ["linear"], ["zoom"], 0, 1, 10, 1, 12, 0]
}
```

All sky properties are transitionable and accept zoom expressions.

## terrain

Renders the map draped over 3D elevation.

| Key | Type | Default | Notes |
|---|---|---|---|
| `source` | string | | **Required.** ID of a `raster-dem` source. |
| `exaggeration` | number ≥ 0 | `1.0` | Vertical scale factor. |

```json
"sources": {
  "terrain-dem": { "type": "raster-dem", "url": "https://example.com/terrain/tiles.json", "tileSize": 256 }
},
"terrain": { "source": "terrain-dem", "exaggeration": 1.5 }
```

Support: GL JS 2.2.0+. Not implemented in Native at the time of writing. GL JS warns when the same source object feeds both `terrain` and a `hillshade` layer; declare two sources that point at the same tiles.

## projection

```json
"projection": { "type": "globe" }
```

`type` is a [projectionDefinition](types.md#projectiondefinition), default `"mercator"`:

- `"mercator"`: Web Mercator.
- `"vertical-perspective"`: a globe-like perspective view.
- `"globe"`: a preset equal to `["interpolate", ["linear"], ["zoom"], 10, "vertical-perspective", 12, "mercator"]`, a globe that flattens to Mercator between zoom 10 and 12.
- A zoom expression: `["step", ["zoom"], "vertical-perspective", 11, "mercator"]`.
- An explicit transition state: `["vertical-perspective", "mercator", 0.7]`.

GL JS 6.15+ also accepts, as a plain string only (never inside an expression), the experimental `"identity"` (lng/lat used as plane coordinates) and names registered with `addProjection` that contain a colon, such as `"EPSG:2193"`.

## transition

Default timing for animated changes to transitionable paint properties:

```json
"transition": { "duration": 300, "delay": 0 }
```

| Key | Type | Default | Units |
|---|---|---|---|
| `duration` | number ≥ 0 | `300` | milliseconds |
| `delay` | number ≥ 0 | `0` | milliseconds |

Override per property with `<property>-transition` inside `paint`:

```json
"paint": {
  "fill-color": "#4a90d9",
  "fill-color-transition": { "duration": 1000, "delay": 0 }
}
```

Only properties marked `T` in [layer-properties.md](layer-properties.md) have a `-transition` companion. Symbol collision fading is controlled separately (GL JS map option `fadeDuration`).

## font-faces

Maps `text-font` names to font files, for scripts that need complex shaping. See [glyphs-and-fonts.md](glyphs-and-fonts.md#font-faces) for resolution rules.

```json
"font-faces": {
  "Noto Sans Regular": [
    { "url": "https://example.com/fonts/NotoSansKhmer-Regular.ttf", "unicode-range": ["U+1780-17FF"] },
    { "url": "https://example.com/fonts/NotoSansDevanagari-Regular.ttf", "unicode-range": ["U+0900-097F"] }
  ],
  "Unifont": "https://example.com/fonts/unifont-15.0.01.ttf"
}
```

Support: MapLibre Native Android 11.13.0, iOS 6.18.0. Not yet implemented in GL JS.
