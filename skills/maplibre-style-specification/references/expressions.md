# Expressions

An expression is a JSON array: `[operator, arg0, arg1, ...]`. Each argument is a literal (string, number, boolean, `null`) or another expression. Any `layout` property, `paint` property, or `filter` may be an expression, subject to the limits in [layer-properties.md](layer-properties.md) (the **Expr** column).

## Contents

- [Kinds of expressions](#kinds-of-expressions)
- [Type system](#type-system)
- [Operator reference](#operator-reference): variable binding, types, lookup, feature data, decision, ramps, math, color, string, camera, heatmap and color relief
- [Recipes](#recipes)
- [Common errors](#common-errors)

## Kinds of expressions

**Data expressions** use feature data: `get`, `has`, `id`, `geometry-type`, `properties`, or `feature-state`. They vary styling per feature. Allowed in `filter` and in properties marked `feature`. `feature-state` is allowed only in paint properties marked `state`.

```json
"circle-color": ["rgb", ["get", "temperature"], 0, ["-", 100, ["get", "temperature"]]]
```

**Camera expressions** use `["zoom"]`. In a layout or paint property, `["zoom"]` may appear **only** as the input of an outermost `interpolate` or `step`, optionally inside `let`:

```json
["interpolate", interpolation, ["zoom"], ...]
["step", ["zoom"], ...]
["let", "name", value, ["interpolate", interpolation, ["zoom"], ...]]
```

Paint properties re-evaluate camera expressions continuously (zoom 4.1 → 4.6 changes the value); layout properties and filters re-evaluate only at integer zooms.

**Composite expressions** combine both. Put the data part in the stop outputs:

```json
"circle-radius": ["interpolate", ["linear"], ["zoom"],
  0, ["get", "rating"],
  10, ["*", 4, ["get", "rating"]]
]
```

**Global-state expressions** use `["global-state", "key"]`, with defaults in the root [`state`](root-properties.md#state). They are constant across features, so they work almost anywhere, including `visibility` (GL JS 5.16.0+).

## Type system

Types: `string`, `number`, `boolean`, `color`, `object`, `array` (optionally `array<type, length>`), `null`, `value` (any), plus `formatted`, `resolvedImage`, `collator`, `padding`, `numberArray`, `colorArray`, `variableAnchorOffsetCollection`, and `projectionDefinition` (see [types.md](types.md)).

- Expressions are type-checked when parsed. A filter must return `boolean`; `"+"` needs numbers.
- Feature property types are unknown until evaluation. If a value has the wrong type at runtime, the expression fails and the property uses its **default value** (logged as a warning in GL JS).
- Where the type cannot be inferred (`["<", ["get", "a"], ["get", "b"]]`), assert it: `["<", ["number", ["get", "a"]], ["number", ["get", "b"]]]`.
- The only implicit conversion is string → color where a color is expected. Otherwise convert with `to-number`, `to-string`, `to-boolean`, or `to-color`.
- Array literals inside expressions must be wrapped: `["in", 1, ["literal", [1, 2, 3]]]`. A bare `[1, 2, 3]` is parsed as an expression and fails. Arrays returned by sub-expressions need no wrapping.

## Operator reference

Signatures use `?` for optional arguments and `...` for repetition. Support versions are MapLibre GL JS / Native Android / Native iOS. Operators without a version note have been supported since GL JS 0.41-0.45, Android 6.0, iOS 4.0.

### Variable binding

| Operator | Signature | Notes |
|---|---|---|
| `let` | `["let", name1, value1, ..., nameN, valueN, body]` | Binds names for use in `body`. |
| `var` | `["var", name]` | References a `let` binding. |

```json
["let", "density", ["/", ["get", "population"], ["get", "sq-km"]],
  ["interpolate", ["linear"], ["zoom"],
    8, ["interpolate", ["linear"], ["var", "density"], 274, "#edf8e9", 1551, "#006d2c"],
    10, ["interpolate", ["linear"], ["var", "density"], 274, "#eff3ff", 1551, "#08519c"]]]
```

### Types

| Operator | Signature | Returns | Notes |
|---|---|---|---|
| `literal` | `["literal", jsonArrayOrObject]` | array / object | Treats nested values as literals. |
| `semiliteral` | `["semiliteral", jsonArray]` | array | Like `literal` but evaluates expressions inside the elements: `["semiliteral", [["get", "x"], ["get", "y"]]]`. GL JS 6.6.0; not in Native. |
| `array` | `["array", value]`, `["array", type, value]`, `["array", type, length, value]` | array | Asserts array (of `string`/`number`/`boolean`, optional length); aborts the expression otherwise. |
| `string` | `["string", v1, ..., vN]` | string | Asserts string; with several inputs, returns the first that is a string. |
| `number` | `["number", v1, ..., vN]` | number | Asserts number. Common fallback form: `["number", ["get", "x"], 0]`. |
| `boolean` | `["boolean", v1, ..., vN]` | boolean | Asserts boolean. `["boolean", ["feature-state", "hover"], false]`. |
| `object` | `["object", v1, ..., vN]` | object | Asserts object. |
| `typeof` | `["typeof", value]` | string | Type name of a value. |
| `to-string` | `["to-string", value]` | string | `null` → `""`; booleans → `"true"`/`"false"`; numbers per ECMAScript; colors → `"rgba(r,g,b,a)"`; else JSON. |
| `to-number` | `["to-number", v1, ..., vN]` | number | `null`/`false` → 0, `true` → 1, strings parsed as numbers. Tries each input in turn; errors if none converts. |
| `to-boolean` | `["to-boolean", value]` | boolean | False for `""`, `0`, `false`, `null`, `NaN`; true otherwise. |
| `to-color` | `["to-color", v1, ..., vN]` | color | First input that converts to a color. |
| `collator` | `["collator", {"case-sensitive": bool, "diacritic-sensitive": bool, "locale": "fr"}]` | collator | For locale-aware comparisons (`==`, `<`, ...). |
| `format` | `["format", input1, options1?, ..., inputN, optionsN?]` | formatted | Rich text for `text-field`. Inputs are strings or `image` expressions. Options: `text-font`, `text-color`, `font-scale`, `vertical-align` (`bottom`, `center`, `top`; GL JS 5.1.0). |
| `image` | `["image", name]` | resolvedImage | Returns the image if it exists in the style, otherwise `null`; use with `coalesce` for fallbacks. Usable in `icon-image`, `*-pattern`, and `format`. |
| `number-format` | `["number-format", number, {locale, currency, unit, min-fraction-digits, max-fraction-digits}]` | string | Locale-aware number formatting. `unit` (e.g. `"meter"`) needs GL JS 5.20.0. |

```json
"text-field": ["format",
  ["upcase", ["get", "name"]], {"font-scale": 1.0, "text-font": ["literal", ["Open Sans Semibold"]]},
  "\n", {},
  ["number-format", ["get", "elevation"], {"max-fraction-digits": 0}], {"font-scale": 0.75, "text-color": "#666"}
]
```

### Lookup

| Operator | Signature | Returns | Notes |
|---|---|---|---|
| `get` | `["get", name]`, `["get", name, object]` | value | Feature property (or a key of `object`). `null` if missing. |
| `has` | `["has", name]`, `["has", name, object]` | boolean | Whether the property exists. |
| `at` | `["at", index, array]` | item | Zero-based. |
| `in` | `["in", needle, arrayOrString]` | boolean | Item in array, or substring in string. GL JS 1.6.0. |
| `index-of` | `["index-of", needle, arrayOrString, fromIndex?]` | number | `-1` if not found. GL JS 1.10.0. |
| `slice` | `["slice", arrayOrString, start, end?]` | array / string | End exclusive. GL JS 1.10.0. |
| `length` | `["length", arrayOrString]` | number | |
| `global-state` | `["global-state", name]` | value | Style-wide runtime value, defaults from root `state`. `null` if unset. GL JS 5.6.0 / 13.7.1 / 7.0.0. `name` must be a string literal. |

In strings, `index-of`, `slice`, and `length` count a UTF-16 surrogate pair as one position.

### Feature data

| Operator | Signature | Returns | Notes |
|---|---|---|---|
| `properties` | `["properties"]` | object | All properties. `get` is usually more efficient. |
| `id` | `["id"]` | value | The feature ID, if any. |
| `geometry-type` | `["geometry-type"]` | string | `Point`, `LineString`, or `Polygon` (multi-types return the single form). |
| `feature-state` | `["feature-state", name]` | value | Runtime state set on the feature by ID. Paint properties only; not filters or layout. `null` if unset. Not yet in Native. |
| `line-progress` | `["line-progress"]` | number | 0-1 along the line. Only in `line-gradient`. |
| `accumulated` | `["accumulated"]` | value | Running value in a GeoJSON `clusterProperties` reducer only. |

### Decision

| Operator | Signature | Notes |
|---|---|---|
| `case` | `["case", cond1, out1, ..., condN, outN, fallback]` | First output whose condition is true. |
| `match` | `["match", input, label1, out1, ..., labelN, outN, fallback]` | `input` is a string or number expression. Each label is a **literal** or an array of literals (all strings or all numbers), and labels must be unique. A type mismatch between input and labels returns the fallback. |
| `coalesce` | `["coalesce", e1, ..., eN]` | First non-null value. |
| `==`, `!=` | `["==", a, b, collator?]` | Strictly typed: different runtime types are never equal; types known to differ at parse time are a parse error. |
| `<`, `<=`, `>`, `>=` | `["<", a, b, collator?]` | Both strings or both numbers. |
| `all` | `["all", b1, ..., bN]` | Logical AND, short-circuits. `["all"]` is `true`. |
| `any` | `["any", b1, ..., bN]` | Logical OR, short-circuits. `["any"]` is `false`. |
| `!` | `["!", b]` | Logical NOT. |
| `within` | `["within", geojson]` | True if the feature lies fully inside a `Polygon`/`MultiPolygon` (or a Feature/FeatureCollection of them). Points on the boundary and lines touching it are false. Works for points and lines. GL JS 1.9.0. |

```json
"line-color": ["match", ["get", "class"],
  "motorway", "#e892a2",
  ["trunk", "primary"], "#fcd6a4",
  ["secondary", "tertiary"], "#f7fabf",
  "#ffffff"]
```

### Ramps, scales, curves

| Operator | Signature | Notes |
|---|---|---|
| `step` | `["step", input, out0, stop1, out1, ..., stopN, outN]` | Piecewise constant. `out0` applies below `stop1`; each output applies from its stop up to the next. Stops: number literals, strictly ascending. Any output type. |
| `interpolate` | `["interpolate", interpolation, input, stop1, out1, ..., stopN, outN]` | Smooth. Outputs: `number`, `array<number>`, `color`, `array<color>`, or `projection`. Clamps outside the stop range. |
| `interpolate-hcl` | same as `interpolate` | Colors only, interpolated in Hue-Chroma-Luminance. GL JS 0.49.0; not in Native. |
| `interpolate-lab` | same as `interpolate` | Colors only, interpolated in CIELAB. GL JS 0.49.0; not in Native. |

Interpolation types:

| Type | Form | Behavior |
|---|---|---|
| linear | `["linear"]` | Straight line between stops. |
| exponential | `["exponential", base]` | `base` > 1 grows faster toward the high end (good for zoom-scaled widths, e.g. 1.5 or 2); `base` = 1 is linear; < 1 grows faster at the low end. |
| cubic-bezier | `["cubic-bezier", x1, y1, x2, y2]` | Easing curve; `x1` and `x2` must be between 0 and 1 for a monotonic curve. |

```json
"circle-radius": ["step", ["get", "point_count"], 20, 100, 30, 750, 40]
"line-width": ["interpolate", ["exponential", 2], ["zoom"], 10, 1, 18, 32]
```

### Math

| Operator | Signature | Notes |
|---|---|---|
| `+` | `["+", n1, ..., nN]` | Sum. |
| `*` | `["*", n1, ..., nN]` | Product. |
| `-` | `["-", a, b]`, `["-", a]` | Difference, or negation. |
| `/` | `["/", a, b]` | Floating-point division. |
| `%` | `["%", a, b]` | Remainder. |
| `^` | `["^", base, exponent]` | Power. |
| `sqrt` | `["sqrt", n]` | |
| `abs`, `ceil`, `floor` | `["abs", n]` | |
| `round` | `["round", n]` | Halves round away from zero (`-1.5` → `-2`). |
| `min`, `max` | `["min", n1, ..., nN]` | |
| `ln`, `log10`, `log2` | `["ln", n]` | |
| `sin`, `cos`, `tan`, `asin`, `acos`, `atan` | `["sin", n]` | Radians. |
| `pi`, `e`, `ln2` | `["pi"]` | Constants. |
| `distance` | `["distance", geojson]` | Shortest distance in meters from the feature to a GeoJSON geometry, Feature, or FeatureCollection. Less precise below zoom 13. GL JS 4.2.0 / 9.2.0 / 5.9.0. |

### Color

| Operator | Signature | Notes |
|---|---|---|
| `rgb` | `["rgb", r, g, b]` | 0-255 each; alpha 1. Out of range is an error. |
| `rgba` | `["rgba", r, g, b, a]` | Alpha 0-1. |
| `to-rgba` | `["to-rgba", color]` | `[r, g, b, a]` array. |
| `to-color` | see [Types](#types) | |

### String

| Operator | Signature | Notes |
|---|---|---|
| `concat` | `["concat", v1, ..., vN]` | Each input converted as by `to-string`. |
| `upcase`, `downcase` | `["upcase", s]` | Unicode default case conversion. |
| `split` | `["split", s, separator]` | Array of strings. GL JS 5.20.0 / 13.5.0 / 6.29.0. |
| `join` | `["join", array, separator]` | String. Same support as `split`. |
| `is-supported-script` | `["is-supported-script", s]` | False if the text needs shaping the renderer cannot do (e.g. RTL without the rtl-text plugin, some Indic scripts). |
| `resolved-locale` | `["resolved-locale", collator]` | The locale the collator actually uses. |

Also see `slice`, `index-of`, `length`, `in` (lookup), `number-format` and `format` (types).

### Camera

| Operator | Signature | Notes |
|---|---|---|
| `zoom` | `["zoom"]` | Current zoom. In layout/paint properties only as the input of a top-level `step` or `interpolate`. |

### Heatmap and color relief

| Operator | Signature | Notes |
|---|---|---|
| `heatmap-density` | `["heatmap-density"]` | 0-1 kernel density; only in `heatmap-color`. |
| `elevation` | `["elevation"]` | Meters. In `color-relief-color` (DEM elevation) and, where supported, `fill-extrusion-color` (height above ground). GL JS 5.6.0 / 13.0.0 / 6.24.0. |

## Recipes

### Label with fallback language and abbreviation at low zoom

```json
"text-field": ["step", ["zoom"],
  ["coalesce", ["get", "abbrev"], ["get", "name"]],
  5, ["coalesce", ["get", "name:en"], ["get", "name:latin"], ["get", "name"]]]
```

### Icon from a property, with a fallback image

```json
"icon-image": ["coalesce", ["image", ["concat", ["get", "class"], "_11"]], ["image", "circle_11"]]
```

### Numeric property stored as a string

```json
"fill-color": ["interpolate", ["linear"], ["to-number", ["get", "pop_density"], 0],
  0, "#ffffcc", 100, "#a1dab4", 1000, "#41b6c4", 10000, "#225ea8"]
```

### Hide features with a missing value, style the rest

```json
"filter": ["all", ["has", "height"], [">", ["to-number", ["get", "height"]], 0]]
```

### Fade a layer in by zoom (paint)

```json
"fill-opacity": ["interpolate", ["linear"], ["zoom"], 9, 0, 10, 0.7]
```

### Size by zoom and data together

```json
"circle-radius": ["interpolate", ["linear"], ["zoom"],
  5, ["max", 2, ["/", ["sqrt", ["get", "pop"]], 200]],
  12, ["max", 4, ["/", ["sqrt", ["get", "pop"]], 40]]]
```

### Hover highlight with feature state

```json
"line-width": ["case", ["boolean", ["feature-state", "hover"], false], 4, 1.5]
```

### Spatial filter

```json
"filter": ["within", {"type": "Polygon", "coordinates": [[[0, 0], [0, 5], [5, 5], [5, 0], [0, 0]]]}]
```

## Common errors

| Message (GL JS / validator) | Cause and fix |
|---|---|
| `"zoom" expression may only be used as input to a top-level "step" or "interpolate" expression` | Move `["zoom"]` to the outermost `interpolate`/`step`; push other logic into the stop outputs. |
| `Expression name must be a string, but found number instead. If you wanted a literal array, use ["literal", [...]].` / `Unknown expression "..."` | An array literal was not wrapped in `["literal", ...]`, or the first element is not an operator. |
| `Input/output pairs for "interpolate" expressions must be arranged with input values in strictly ascending order` | Sort stops; remove duplicates. |
| `Branch labels must be numbers or strings` / `Branch labels must be unique` | `match` labels must be literals; merge duplicates into an array label. |
| `Expected number but found string instead` | Property stored as text; use `to-number`. |
| `Expected value to be of type boolean, but found null instead` | Unset `feature-state` or a missing property; wrap with `["boolean", ..., false]` or `coalesce`. |
| `data expressions not supported` | The property does not accept `feature` inputs; check the Expr column. |
| `"feature-state" data expressions are not supported with layout properties` | Move the logic to a paint property, or use `global-state`. |
