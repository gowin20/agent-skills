# Value types

The types used for property values and expression results. The **Type** column in [layer-properties.md](layer-properties.md) uses these names.

## Primitive types

| Type | Example | Notes |
|---|---|---|
| string | `"source": "mySource"` | Quoted text. |
| number | `"text-size": 24` | Integer or decimal, unquoted. |
| boolean | `"fill-antialias": true` | `true` or `false`. |
| enum | `"line-cap": "round"` | One of a closed set of strings. Any other value is an error. |

## color

A color in sRGB, written as a string:

```json
"line-color": "#ff0"
"line-color": "#ffff00"
"line-color": "#ffff0080"
"line-color": "rgb(255, 255, 0)"
"line-color": "rgba(255, 255, 0, 0.5)"
"line-color": "hsl(60, 100%, 50%)"
"line-color": "hsla(60, 100%, 50%, 0.5)"
"line-color": "yellow"
"line-color": "transparent"
```

The parser follows CSS Color 4: hex with 3, 4, 6, or 8 digits, `rgb()`/`rgba()` and `hsl()`/`hsla()` with comma or space syntax (`rgb(255 255 0 / 50%)`), named colors, and `transparent`. Hue angles accept only plain numbers or `deg`. Old non-standard forms such as `hsl(900, 0.15, 90%)` (fractions instead of percentages) are rejected; `gl-style-migrate` fixes them.

Expressions create colors with `rgb`, `rgba`, `to-color`, and interpolation. A string from feature data converts to a color automatically only where a color is expected.

## array

An ordered list. As a plain property value, write it directly: `"line-dasharray": [2, 4]`. **Inside an expression**, wrap literal arrays: `["in", 1, ["literal", [1, 2, 3]]]`.

## point

A two-number array `[x, y]`; units depend on the property (pixels for `*-translate`, ems for `text-offset`). Positive x is right, positive y is down.

```json
"line-translate": [2, 4]
```

## padding

CSS-like padding for collision boxes (`icon-padding`):

| Form | Meaning |
|---|---|
| `[2]` | All four sides. |
| `[2, 3]` | Top/bottom, left/right. |
| `[2, 3, 1]` | Top, left/right, bottom. |
| `[2, 3, 1, 0]` | Top, right, bottom, left. |

A bare number is accepted and treated like a one-element array. Arrays need GL JS 2.2.0, Android 11.5.0, iOS 6.7.0.

## numberArray and colorArray

A single value or an array of values, used by multi-light hillshading:

```json
"hillshade-illumination-direction": [45, 57.3]
"hillshade-highlight-color": ["#ffff00", "rgb(255, 255, 0)", "yellow"]
```

## formatted

Text split into sections, each with its own formatting. Produced by the `format` expression and used by `text-field`. A plain string is treated as a single unformatted section.

```json
"text-field": ["format", "foo", {"font-scale": 1.2}, "bar", {"font-scale": 0.8}]
```

## resolvedImage

An image reference checked against the images available in the style (sprite or runtime images). Produced by `["image", name]`; a plain string is also accepted where a `resolvedImage` is expected. `image` returns `null` for unavailable images, which enables fallbacks:

```json
"icon-image": ["coalesce", ["image", "myImage"], ["image", "fallbackImage"]]
```

## variableAnchorOffsetCollection

Used by `text-variable-anchor-offset`: an even-length array alternating anchor names and `[x, y]` offsets in ems.

```json
"text-variable-anchor-offset": ["top", [0, 4], "left", [3, 0], "bottom", [1, 1]]
```

The renderer tries each anchor in order and uses its paired offset. Anchors may repeat with different offsets. When set, it overrides `text-anchor`, `text-variable-anchor`, `text-offset`, and `text-radial-offset`.

## projectionDefinition

The value of `projection.type`:

| Form | Example |
|---|---|
| Name | `"mercator"`, `"vertical-perspective"` |
| Preset | `"globe"` = `["interpolate", ["linear"], ["zoom"], 10, "vertical-perspective", 12, "mercator"]` |
| Zoom step | `["step", ["zoom"], "vertical-perspective", 11, "mercator"]` |
| Zoom interpolation | `["interpolate", ["linear"], ["zoom"], 10, "vertical-perspective", 12, "mercator"]` |
| Transition state | `["vertical-perspective", "mercator", 0.7]` (`[from, to, t]`, t from 0 to 1) |

The renderer always receives `[from, to, transition]`. With the interpolation above, zoom 11 gives `["vertical-perspective", "mercator", 0.5]`.

## Units used in the spec

| Unit | Where |
|---|---|
| pixels | Widths, radii, offsets, translate, padding, blur (CSS pixels, not device pixels). |
| ems | `text-offset`, `text-radial-offset`, `text-max-width`, `text-line-height`, `text-letter-spacing` (multiples of `text-size`). |
| line widths | `line-dasharray` (multiples of `line-width`). |
| degrees | Rotations, bearings, illumination direction. |
| meters | `fill-extrusion-height`, `fill-extrusion-base`, `symbol-height-offset`, `elevation`. |
| milliseconds | Transitions, `raster-fade-duration`. |
