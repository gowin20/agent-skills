# Layer property reference

Generated from `src/reference/v8.json` in maplibre/maplibre-style-spec (npm `@maplibre/maplibre-gl-style-spec` 26.5.1, style spec version 8).
Read the section for one layer type at a time.

Column key:

- **Kind**: `L` = layout (goes in `layout`), `P` = paint (goes in `paint`).
- **Expr**: inputs an expression may use. `zoom` = camera expression allowed; `feature` = data-driven (`get`, `match` on properties, etc.); `state` = `feature-state` allowed; `(interp)` = value can be interpolated; `T` = transitionable (a `<name>-transition` property exists). `-` = constant value only.
- **Range/units**: `min..max` and units, when the spec gives them.
- **JS** / **Native**: first MapLibre GL JS / MapLibre Native (Android) version with basic support, or `no` if not implemented there yet. `dd` gives the version that supports data-driven (feature) expressions, when the spec tracks it separately. iOS versions are in the online docs.

Every layer type also has the layout property `visibility` (`visible` or `none`, default `visible`). It cannot use zoom or feature data, but it can use a `global-state` expression (GL JS 5.16.0, Native Android 13.7.1).

## background

| Property | Kind | Type | Default | Range/units | Expr | JS | Native | Notes |
|---|---|---|---|---|---|---|---|---|
| `background-color` | P | color | `"#000000"` |  | zoom (interp) T | 0.10.0 | 2.0.1 | The color with which the background will be drawn. Requires not `background-pattern`. |
| `background-pattern` | P | resolvedImage |  |  | zoom T | 0.10.0 | 2.0.1 | Name of image in sprite to use for drawing an image background. For seamless patterns, image width and height must be a factor of two (2, 4, 8, ..., 512). |
| `background-opacity` | P | number | `1` | 0..1 | zoom (interp) T | 0.10.0 | 2.0.1 | The opacity at which the background will be drawn. |

## fill

| Property | Kind | Type | Default | Range/units | Expr | JS | Native | Notes |
|---|---|---|---|---|---|---|---|---|
| `fill-sort-key` | L | number |  |  | zoom,feature | 1.2.0; dd 1.2.0 | 9.1.0; dd 9.1.0 | Sorts features in ascending order based on this value. Features with a higher sort key will appear above features with a lower sort key. |
| `fill-antialias` | P | boolean | `true` |  | zoom | 0.10.0 | 2.0.1 | Whether or not the fill should be antialiased. |
| `fill-opacity` | P | number | `1` | 0..1 | zoom,feature,state (interp) T | 0.10.0; dd 0.21.0 | 2.0.1; dd 5.0.0 | The opacity of the entire fill layer. In contrast to the [`fill-color`], this value will also affect the 1px stroke around the fill, if the stroke is used. |
| `fill-layer-opacity` | P | number | `1` | 0..1 | zoom,global-state (interp) T | 6.0.0 | no | Opacity applied to the layer as a whole. Unlike [`fill-opacity`] and alpha values in [`fill-color`], which are applied per feature and accumulate where fills overlap, `fill-layer-opacity` is applied once so overlapping fills appear as a single surface at the given opacity. |
| `fill-layer-blend` | P | enum | `"normal"` |  | - | no | no | Controls how the layer is composited with the content already rendered beneath it. Unlike per-feature opacity properties such as `fill-opacity`, this property affects the final composited layer output and determines how it interacts with the existing map contents. Values: `normal`, `multiply`, `color`, `luminosity`, `plus-lighter`, `screen`, `overlay`. |
| `fill-color` | P | color | `"#000000"` |  | zoom,feature,state (interp) T | 0.10.0; dd 0.19.0 | 2.0.1; dd 5.0.0 | The color of the filled part of this layer. This color can be specified as `rgba` with an alpha component and the color's opacity will not affect the opacity of the 1px stroke, if it is used. |
| `fill-outline-color` | P | color |  |  | zoom,feature,state (interp) T | 0.10.0; dd 0.19.0 | 2.0.1; dd 5.0.0 | The outline color of the fill. Matches the value of [`fill-color`] if unspecified. Requires not `fill-pattern`, `fill-antialias`=true. |
| `fill-translate` | P | array<number, 2> | `[0, 0]` | pixels | zoom (interp) T | 0.10.0 | 2.0.1 | The geometry's offset. Values are `[x, y]` where negatives indicate left and up, respectively. |
| `fill-translate-anchor` | P | enum | `"map"` |  | zoom | 0.10.0 | 2.0.1 | Controls the frame of reference for [`fill-translate`]. Values: `map`, `viewport`. Requires `fill-translate`. |
| `fill-pattern` | P | resolvedImage |  |  | zoom,feature T | 0.10.0; dd 0.49.0 | 2.0.1; dd 6.5.0 | Name of image in sprite to use for drawing image fills. For seamless patterns, image width and height must be a factor of two (2, 4, 8, ..., 512). |

## line

| Property | Kind | Type | Default | Range/units | Expr | JS | Native | Notes |
|---|---|---|---|---|---|---|---|---|
| `line-cap` | L | enum | `"butt"` |  | zoom,feature | 0.10.0; dd 5.22.0 | 2.0.1; dd no | The display of line endings. Values: `butt`, `round`, `square`. |
| `line-join` | L | enum | `"miter"` |  | zoom,feature | 0.10.0; dd 0.40.0 | 2.0.1; dd 5.2.0 | The display of lines when joining. Values: `bevel`, `round`, `miter`. |
| `line-miter-limit` | L | number | `2` |  | zoom,feature (interp) | 0.10.0; dd 5.22.0 | 2.0.1; dd no | Used to automatically convert miter joins to bevel joins for sharp angles. Requires `line-join`="miter". |
| `line-round-limit` | L | number | `1.05` |  | zoom,feature (interp) | 0.10.0; dd 5.22.0 | 2.0.1; dd no | Used to automatically convert round joins to miter joins for shallow angles. Requires `line-join`="round". |
| `line-sort-key` | L | number |  |  | zoom,feature | 1.2.0; dd 1.2.0 | 9.1.0; dd 9.1.0 | Sorts features in ascending order based on this value. Features with a higher sort key will appear above features with a lower sort key. |
| `line-opacity` | P | number | `1` | 0..1 | zoom,feature,state (interp) T | 0.10.0; dd 0.29.0 | 2.0.1; dd 5.0.0 | The opacity at which the line will be drawn. |
| `line-layer-opacity` | P | number | `1` | 0..1 | zoom,global-state (interp) T | 6.0.0 | no | Opacity applied to the layer as a whole. Unlike [`line-opacity`] and alpha values in [`line-color`], which are applied per feature and accumulate where lines overlap, `line-layer-opacity` is applied once so overlapping lines appear as a single surface at the given opacity. |
| `line-layer-blend` | P | enum | `"normal"` |  | - | no | no | Controls how the layer is composited with the content already rendered beneath it. Unlike per-feature opacity properties such as `line-opacity`, this property affects the final composited layer output and determines how it interacts with the existing map contents. Values: `normal`, `multiply`, `color`, `luminosity`, `plus-lighter`, `screen`, `overlay`. |
| `line-color` | P | color | `"#000000"` |  | zoom,feature,state (interp) T | 0.10.0; dd 0.23.0 | 2.0.1; dd 5.0.0 | The color with which the line will be drawn. Requires not `line-pattern`. |
| `line-translate` | P | array<number, 2> | `[0, 0]` | pixels | zoom (interp) T | 0.10.0 | 2.0.1 | The geometry's offset. Values are `[x, y]` where negatives indicate left and up, respectively. |
| `line-translate-anchor` | P | enum | `"map"` |  | zoom | 0.10.0 | 2.0.1 | Controls the frame of reference for [`line-translate`]. Values: `map`, `viewport`. Requires `line-translate`. |
| `line-width` | P | number | `1` | 0.. pixels | zoom,feature,state (interp) T | 0.10.0; dd 0.39.0 | 2.0.1; dd 5.2.0 | Stroke thickness. |
| `line-gap-width` | P | number | `0` | 0.. pixels | zoom,feature,state (interp) T | 0.10.0; dd 0.29.0 | 2.0.1; dd 5.0.0 | Draws a line casing outside of a line's actual path. Value indicates the width of the inner gap. |
| `line-offset` | P | number | `0` | pixels | zoom,feature,state (interp) T | 0.12.1; dd 0.29.0 | 3.0.0; dd 5.0.0 | The line's offset. For linear features, a positive value offsets the line to the right, relative to the direction of the line, and a negative value to the left. |
| `line-blur` | P | number | `0` | 0.. pixels | zoom,feature,state (interp) T | 0.10.0; dd 0.29.0 | 2.0.1; dd 5.0.0 | Blur applied to the line, in pixels. |
| `line-dasharray` | P | array<number> |  | 0.. line widths | zoom,feature T | 0.10.0; dd 5.8.0 | 2.0.1; dd no | Specifies the lengths of the alternating dashes and gaps that form the dash pattern. The lengths are later scaled by the line width. Requires not `line-pattern`. |
| `line-pattern` | P | resolvedImage |  |  | zoom,feature T | 0.10.0; dd 0.49.0 | 2.0.1; dd 6.5.0 | Name of image in sprite to use for drawing image lines. For seamless patterns, image width must be a factor of two (2, 4, 8, ..., 512). |
| `line-gradient` | P | color |  |  | line-progress (interp) | 0.45.0 | 6.5.0 | Defines a gradient with which to color a line feature. Can only be used with GeoJSON sources that specify `"lineMetrics": true`. Requires not `line-dasharray`, not `line-pattern`, `source`="geojson", `has`={"lineMetrics": true}. |

## symbol

| Property | Kind | Type | Default | Range/units | Expr | JS | Native | Notes |
|---|---|---|---|---|---|---|---|---|
| `symbol-placement` | L | enum | `"point"` |  | zoom | 0.10.0 | 2.0.1 | Label placement relative to its geometry. Values: `point`, `line`, `line-center`. |
| `symbol-spacing` | L | number | `250` | 1.. pixels | zoom (interp) | 0.10.0 | 2.0.1 | Distance between two symbol anchors. Requires `symbol-placement`="line". |
| `symbol-avoid-edges` | L | boolean | `false` |  | zoom | 0.10.0 | 2.0.1 | If true, the symbols will not cross tile edges to avoid mutual collisions. Recommended in layers that don't have enough padding in the vector tile to prevent collisions, or if it is a point symbol layer placed after a line symbol layer. |
| `symbol-sort-key` | L | number |  |  | zoom,feature | 0.53.0; dd 0.53.0 | 7.4.0; dd 7.4.0 | Sorts features in ascending order based on this value. Features with lower sort keys are drawn and placed first. |
| `symbol-z-order` | L | enum | `"auto"` |  | zoom | 0.49.0 | 6.6.0 | Determines whether overlapping symbols in the same layer are rendered in the order that they appear in the data source or by their y-position relative to the viewport. To control the order and prioritization of symbols otherwise, use [`symbol-sort-key`]. Values: `auto`, `viewport-y`, `source`. |
| `icon-allow-overlap` | L | boolean | `false` |  | zoom | 0.10.0 | 2.0.1 | If true, the icon will be visible even if it collides with other previously drawn symbols. Requires `icon-image`, not `icon-overlap`. |
| `icon-overlap` | L | enum |  |  | zoom | 2.1.0 | no | Allows for control over whether to show an icon when it overlaps other symbols on the map. If `icon-overlap` is not set, [`icon-allow-overlap`] is used instead. Values: `never`, `always`, `cooperative`. Requires `icon-image`. |
| `icon-ignore-placement` | L | boolean | `false` |  | zoom | 0.10.0 | 2.0.1 | If true, other symbols can be visible even if they collide with the icon. Requires `icon-image`. |
| `icon-optional` | L | boolean | `false` |  | zoom | 0.10.0 | 2.0.1 | If true, text will display without their corresponding icons when the icon collides with other symbols and the text does not. Requires `icon-image`, `text-field`. |
| `icon-rotation-alignment` | L | enum | `"auto"` |  | zoom,feature | 0.10.0; dd no | 2.0.1; dd no | In combination with [`symbol-placement`], determines the rotation behavior of icons. Values: `map`, `viewport`, `auto`. Requires `icon-image`. |
| `icon-size` | L | number | `1` | 0.. factor of the original icon size | zoom,feature (interp) | 0.10.0; dd 0.35.0 | 2.0.1; dd 5.1.0 | Scales the original size of the icon by the provided factor. The new pixel size of the image will be the original pixel size multiplied by `icon-size`. 1 is the original size; 3 triples the size of the image. Requires `icon-image`. |
| `icon-text-fit` | L | enum | `"none"` |  | zoom | 0.21.0 | 4.2.0 | Scales the icon to fit around the associated text. Values: `none`, `width`, `height`, `both`. Requires `icon-image`, `text-field`. |
| `icon-text-fit-padding` | L | array<number, 4> | `[0, 0, 0, 0]` | pixels | zoom (interp) | 0.21.0 | 4.2.0 | Size of the additional area added to dimensions determined by [`icon-text-fit`], in clockwise order: top, right, bottom, left. Requires `icon-image`, `text-field`, `icon-text-fit`=["both", "width", "height"]. |
| `icon-image` | L | resolvedImage |  |  | zoom,feature | 0.10.0; dd 0.35.0 | 2.0.1; dd 5.1.0 | Name of image in sprite to use for drawing an image background. |
| `icon-rotate` | L | number | `0` | degrees | zoom,feature (interp) | 0.10.0; dd 0.21.0 | 2.0.1; dd 5.0.0 | Rotates the icon clockwise. Requires `icon-image`. |
| `icon-padding` | L | padding | `[2]` | pixels | zoom,feature (interp) | 0.10.0; dd 2.2.0 | 2.0.1; dd no | Size of additional area round the icon bounding box used for detecting symbol collisions. Requires `icon-image`. |
| `icon-keep-upright` | L | boolean | `false` |  | zoom | 0.10.0 | 2.0.1 | If true, the icon may be flipped to prevent it from being rendered upside-down. Requires `icon-image`, `icon-rotation-alignment`="map", `symbol-placement`=["line", "line-center"]. |
| `icon-offset` | L | array<number, 2> | `[0, 0]` |  | zoom,feature (interp) | 0.10.0; dd 0.29.0 | 2.0.1; dd 5.0.0 | Offset distance of icon from its anchor. Positive values indicate right and down, while negative values indicate left and up. Requires `icon-image`. |
| `icon-anchor` | L | enum | `"center"` |  | zoom,feature | 0.40.0; dd 0.40.0 | 5.2.0; dd 5.2.0 | Part of the icon placed closest to the anchor. Values: `center`, `left`, `right`, `top`, `bottom`, `top-left`, `top-right`, `bottom-left`, `bottom-right`. Requires `icon-image`. |
| `icon-pitch-alignment` | L | enum | `"auto"` |  | zoom | 0.39.0 | 5.2.0 | Orientation of icon when map is pitched. Values: `map`, `viewport`, `auto`. Requires `icon-image`. |
| `text-pitch-alignment` | L | enum | `"auto"` |  | zoom | 0.21.0 | 4.2.0 | Orientation of text when map is pitched. Values: `map`, `viewport`, `auto`. Requires `text-field`. |
| `text-rotation-alignment` | L | enum | `"auto"` |  | zoom | 0.10.0 | 2.0.1 | In combination with [`symbol-placement`], determines the rotation behavior of the individual glyphs forming the text. Values: `map`, `viewport`, `viewport-glyph`, `auto`. Requires `text-field`. |
| `text-field` | L | formatted | `""` |  | zoom,feature | 0.10.0; dd 0.33.0 | 2.0.1; dd 5.0.0 | Value to use for a text label. If a plain `string` is provided, it will be treated as a `formatted` with default/inherited formatting options. |
| `text-font` | L | array<string> | `["Open Sans Regular", "Arial Unicode MS Regular"]` |  | zoom,feature | 0.10.0; dd 0.43.0 | 2.0.1; dd 6.0.0 | Fonts to use for displaying text. If the `glyphs` root property is specified, this array is joined together and interpreted as a font stack name. Requires `text-field`. |
| `text-size` | L | number | `16` | 0.. pixels | zoom,feature (interp) | 0.10.0; dd 0.35.0 | 2.0.1; dd 5.1.0 | Font size. Requires `text-field`. |
| `text-max-width` | L | number | `10` | 0.. ems | zoom,feature (interp) | 0.10.0; dd 0.40.0 | 2.0.1; dd 5.2.0 | The maximum line width for text wrapping. Requires `text-field`. |
| `text-line-height` | L | number | `1.2` | ems | zoom (interp) | 0.10.0 | 2.0.1 | Text leading value for multi-line text. Requires `text-field`. |
| `text-letter-spacing` | L | number | `0` | ems | zoom,feature (interp) | 0.10.0; dd 0.40.0 | 2.0.1; dd 5.2.0 | Text tracking amount. Requires `text-field`. |
| `text-justify` | L | enum | `"center"` |  | zoom,feature | 0.10.0; dd 0.39.0 | 2.0.1; dd 5.2.0 | Text justification options. Values: `auto`, `left`, `center`, `right`. Requires `text-field`. |
| `text-radial-offset` | L | number | `0` | ems | zoom,feature (interp) | 0.54.0; dd 0.54.0 | 7.4.0; dd 7.4.0 | Radial offset of text, in the direction of the symbol's anchor. Useful in combination with [`text-variable-anchor`], which defaults to using the two-dimensional [`text-offset`] if present. Requires `text-field`. |
| `text-variable-anchor` | L | array<enum> |  |  | zoom | 0.54.0 | 7.4.0 | To increase the chance of placing high-priority labels on the map, you can provide an array of [`text-anchor`] locations: the renderer will attempt to place the label at each location, in order, before moving onto the next label. Use `text-justify: auto` to choose justification based on anchor position. Values: `center`, `left`, `right`, `top`, `bottom`, `top-left`, `top-right`, `bottom-left`, `bottom-right`. Requires `text-field`, `symbol-placement`=["point"]. |
| `text-variable-anchor-offset` | L | variableAnchorOffsetCollection |  |  | zoom,feature (interp) | 3.3.0; dd 3.3.0 | 11.6.0; dd no | To increase the chance of placing high-priority labels on the map, you can provide an array of [`text-anchor`] locations, each paired with an offset value. The renderer will attempt to place the label at each location, in order, before moving on to the next location+offset. Requires `text-field`, `symbol-placement`=["point"]. |
| `text-anchor` | L | enum | `"center"` |  | zoom,feature | 0.10.0; dd 0.39.0 | 2.0.1; dd 5.2.0 | Part of the text placed closest to the anchor. Values: `center`, `left`, `right`, `top`, `bottom`, `top-left`, `top-right`, `bottom-left`, `bottom-right`. Requires `text-field`, not `text-variable-anchor`. |
| `text-max-angle` | L | number | `45` | degrees | zoom (interp) | 0.10.0 | 2.0.1 | Maximum angle change between adjacent characters. Requires `text-field`, `symbol-placement`=["line", "line-center"]. |
| `text-writing-mode` | L | array<enum> |  |  | zoom | 1.3.0 | 8.3.0 | The property allows control over a symbol's orientation. Note that the property values act as a hint, so that a symbol whose language doesn’t support the provided orientation will be laid out in its natural orientation. Values: `horizontal`, `vertical`. Requires `text-field`, `symbol-placement`=["point"]. |
| `text-rotate` | L | number | `0` | degrees | zoom,feature (interp) | 0.10.0; dd 0.35.0 | 2.0.1; dd 5.1.0 | Rotates the text clockwise. Requires `text-field`. |
| `text-padding` | L | number | `2` | 0.. pixels | zoom (interp) | 0.10.0 | 2.0.1 | Size of the additional area around the text bounding box used for detecting symbol collisions. Requires `text-field`. |
| `text-keep-upright` | L | boolean | `true` |  | zoom | 0.10.0 | 2.0.1 | If true, the text may be flipped vertically to prevent it from being rendered upside-down. Requires `text-field`, `text-rotation-alignment`="map", `symbol-placement`=["line", "line-center"]. |
| `text-transform` | L | enum | `"none"` |  | zoom,feature | 0.10.0; dd 0.33.0 | 2.0.1; dd 5.0.0 | Specifies how to capitalize text, similar to the CSS `text-transform` property. Values: `none`, `uppercase`, `lowercase`. Requires `text-field`. |
| `text-offset` | L | array<number, 2> | `[0, 0]` | ems | zoom,feature (interp) | 0.10.0; dd 0.35.0 | 2.0.1; dd 5.1.0 | Offset distance of text from its anchor. Positive values indicate right and down, while negative values indicate left and up. Requires `text-field`, not `text-radial-offset`. |
| `text-allow-overlap` | L | boolean | `false` |  | zoom | 0.10.0 | 2.0.1 | If true, the text will be visible even if it collides with other previously drawn symbols. Requires `text-field`, not `text-overlap`. |
| `text-overlap` | L | enum |  |  | zoom | 2.1.0 | no | Allows for control over whether to show symbol text when it overlaps other symbols on the map. If `text-overlap` is not set, [`text-allow-overlap`] is used instead Values: `never`, `always`, `cooperative`. Requires `text-field`. |
| `text-ignore-placement` | L | boolean | `false` |  | zoom | 0.10.0 | 2.0.1 | If true, other symbols can be visible even if they collide with the text. Requires `text-field`. |
| `text-optional` | L | boolean | `false` |  | zoom | 0.10.0 | 2.0.1 | If true, icons will display without their corresponding text when the text collides with other symbols and the icon does not. Requires `text-field`, `icon-image`. |
| `symbol-height-offset` | L | number | `0` | meters | zoom,feature (interp) | 6.6.0; dd 6.6.0 | no; dd no | The height above the anchor, in meters, at which the symbol is placed. Use [`symbol-height-anchor`] to control what this height is measured from. Requires `symbol-placement`=["point"]. |
| `symbol-height-anchor` | L | enum | `"ground"` |  | zoom | no | no | Defines what [`symbol-height-offset`] is measured from. Values: `ground`, `absolute`. Requires `symbol-height-offset`. |
| `icon-opacity` | P | number | `1` | 0..1 | zoom,feature,state (interp) T | 0.10.0; dd 0.33.0 | 2.0.1; dd 5.0.0 | The opacity at which the icon will be drawn. Requires `icon-image`. |
| `icon-color` | P | color | `"#000000"` |  | zoom,feature,state (interp) T | 0.10.0; dd 0.33.0 | 2.0.1; dd 5.0.0 | The color of the icon. This can only be used with SDF icons. Requires `icon-image`. |
| `icon-halo-color` | P | color | `"rgba(0, 0, 0, 0)"` |  | zoom,feature,state (interp) T | 0.10.0; dd 0.33.0 | 2.0.1; dd 5.0.0 | The color of the icon's halo. Icon halos can only be used with SDF icons. Requires `icon-image`. |
| `icon-halo-width` | P | number | `0` | 0.. pixels | zoom,feature,state (interp) T | 0.10.0; dd 0.33.0 | 2.0.1; dd 5.0.0 | Distance of halo to the icon outline. The unit is in pixels only for SDF sprites that were created with a blur radius of 8, multiplied by the display density. Requires `icon-image`. |
| `icon-halo-blur` | P | number | `0` | 0.. pixels | zoom,feature,state (interp) T | 0.10.0; dd 0.33.0 | 2.0.1; dd 5.0.0 | Fade out the halo towards the outside. Requires `icon-image`. |
| `icon-translate` | P | array<number, 2> | `[0, 0]` | pixels | zoom (interp) T | 0.10.0 | 2.0.1 | Distance that the icon's anchor is moved from its original placement. Positive values indicate right and down, while negative values indicate left and up. Requires `icon-image`. |
| `icon-translate-anchor` | P | enum | `"map"` |  | zoom | 0.10.0 | 2.0.1 | Controls the frame of reference for [`icon-translate`]. Values: `map`, `viewport`. Requires `icon-image`, `icon-translate`. |
| `text-opacity` | P | number | `1` | 0..1 | zoom,feature,state (interp) T | 0.10.0; dd 0.33.0 | 2.0.1; dd 5.0.0 | The opacity at which the text will be drawn. Requires `text-field`. |
| `text-color` | P | color | `"#000000"` |  | zoom,feature,state (interp) T | 0.10.0; dd 0.33.0 | 2.0.1; dd 5.0.0 | The color with which the text will be drawn. Requires `text-field`. |
| `text-halo-color` | P | color | `"rgba(0, 0, 0, 0)"` |  | zoom,feature,state (interp) T | 0.10.0; dd 0.33.0 | 2.0.1; dd 5.0.0 | The color of the text's halo, which helps it stand out from backgrounds. Requires `text-field`. |
| `text-halo-width` | P | number | `0` | 0.. pixels | zoom,feature,state (interp) T | 0.10.0; dd 0.33.0 | 2.0.1; dd 5.0.0 | Distance of halo to the font outline. Max text halo width is 1/4 of the font-size. Requires `text-field`. |
| `text-halo-blur` | P | number | `0` | 0.. pixels | zoom,feature,state (interp) T | 0.10.0; dd 0.33.0 | 2.0.1; dd 5.0.0 | The halo's fadeout distance towards the outside. Requires `text-field`. |
| `text-translate` | P | array<number, 2> | `[0, 0]` | pixels | zoom (interp) T | 0.10.0 | 2.0.1 | Distance that the text's anchor is moved from its original placement. Positive values indicate right and down, while negative values indicate left and up. Requires `text-field`. |
| `text-translate-anchor` | P | enum | `"map"` |  | zoom | 0.10.0 | 2.0.1 | Controls the frame of reference for [`text-translate`]. Values: `map`, `viewport`. Requires `text-field`, `text-translate`. |

## circle

| Property | Kind | Type | Default | Range/units | Expr | JS | Native | Notes |
|---|---|---|---|---|---|---|---|---|
| `circle-sort-key` | L | number |  |  | zoom,feature | 1.2.0; dd 1.2.0 | 9.2.0; dd 9.2.0 | Sorts features in ascending order based on this value. Features with a higher sort key will appear above features with a lower sort key. |
| `circle-radius` | P | number | `5` | 0.. pixels | zoom,feature,state (interp) T | 0.10.0; dd 0.18.0 | 2.0.1; dd 5.0.0 | Circle radius. |
| `circle-color` | P | color | `"#000000"` |  | zoom,feature,state (interp) T | 0.10.0; dd 0.18.0 | 2.0.1; dd 5.0.0 | The fill color of the circle. |
| `circle-blur` | P | number | `0` |  | zoom,feature,state (interp) T | 0.10.0; dd 0.20.0 | 2.0.1; dd 5.0.0 | Amount to blur the circle. 1 blurs the circle such that only the centerpoint is full opacity. |
| `circle-opacity` | P | number | `1` | 0..1 | zoom,feature,state (interp) T | 0.10.0; dd 0.20.0 | 2.0.1; dd 5.0.0 | The opacity at which the circle will be drawn. |
| `circle-translate` | P | array<number, 2> | `[0, 0]` | pixels | zoom (interp) T | 0.10.0 | 2.0.1 | The geometry's offset. Values are `[x, y]` where negatives indicate left and up, respectively. |
| `circle-translate-anchor` | P | enum | `"map"` |  | zoom | 0.10.0 | 2.0.1 | Controls the frame of reference for [`circle-translate`]. Values: `map`, `viewport`. Requires `circle-translate`. |
| `circle-pitch-scale` | P | enum | `"map"` |  | zoom | 0.21.0 | 4.2.0 | Controls the scaling behavior of the circle when the map is pitched. Values: `map`, `viewport`. |
| `circle-pitch-alignment` | P | enum | `"viewport"` |  | zoom | 0.39.0 | 5.2.0 | Orientation of circle when map is pitched. Values: `map`, `viewport`. |
| `circle-stroke-width` | P | number | `0` | 0.. pixels | zoom,feature,state (interp) T | 0.29.0; dd 0.29.0 | 5.0.0; dd 5.0.0 | The width of the circle's stroke. Strokes are placed outside of the [`circle-radius`]. |
| `circle-stroke-color` | P | color | `"#000000"` |  | zoom,feature,state (interp) T | 0.29.0; dd 0.29.0 | 5.0.0; dd 5.0.0 | The stroke color of the circle. |
| `circle-stroke-opacity` | P | number | `1` | 0..1 | zoom,feature,state (interp) T | 0.29.0; dd 0.29.0 | 5.0.0; dd 5.0.0 | The opacity of the circle's stroke. |

## heatmap

| Property | Kind | Type | Default | Range/units | Expr | JS | Native | Notes |
|---|---|---|---|---|---|---|---|---|
| `heatmap-radius` | P | number | `30` | 1.. pixels | zoom,feature,state (interp) T | 0.41.0; dd 0.43.0 | 6.0.0; dd 6.0.0 | Radius of influence of one heatmap point in pixels. Increasing the value makes the heatmap smoother, but less detailed. |
| `heatmap-weight` | P | number | `1` | 0.. | zoom,feature,state (interp) | 0.41.0; dd 0.41.0 | 6.0.0; dd 6.0.0 | A measure of how much an individual point contributes to the heatmap. A value of 10 would be equivalent to having 10 points of weight 1 in the same spot. |
| `heatmap-intensity` | P | number | `1` | 0.. | zoom (interp) T | 0.41.0 | 6.0.0 | Similar to [`heatmap-weight`] but controls the intensity of the heatmap globally. Primarily used for adjusting the heatmap based on zoom level. |
| `heatmap-color` | P | color | `["interpolate", ["linear"], ["heatmap-density"], 0, "rgba(0, 0, 255, 0)", 0.1, "royalblue", 0.3, "cyan", 0.5, "lime", 0.7, "yellow", 1, "red"]` |  | heatmap-density (interp) | 0.41.0 | 6.0.0 | Defines the color of each pixel based on its density value in a heatmap. Should be an expression that uses `["heatmap-density"]` as input. |
| `heatmap-opacity` | P | number | `1` | 0..1 | zoom (interp) T | 0.41.0 | 6.0.0 | The global opacity at which the heatmap layer will be drawn. |

## fill-extrusion

| Property | Kind | Type | Default | Range/units | Expr | JS | Native | Notes |
|---|---|---|---|---|---|---|---|---|
| `fill-extrusion-rounded-corner-distance` | L | number | `0` | 0.. meters | - | 6.2.0 | 13.4.0 | The distance from each fill extrusion corner, measured along the adjacent edges, that is replaced by a rounded corner. A value of `0` disables rounded corners. |
| `fill-extrusion-opacity` | P | number | `1` | 0..1 | zoom (interp) T | 0.27.0 | 5.1.0 | The opacity of the entire fill extrusion layer. This is rendered on a per-layer, not per-feature, basis, and data-driven styling is not available. |
| `fill-extrusion-color` | P | color | `"#000000"` |  | zoom,feature,state,elevation (interp) T | 0.27.0; dd 0.27.0 | 5.1.0; dd 5.1.0 | The base color of the extruded fill. The extrusion's surfaces will be shaded differently based on this color in combination with the root `light` settings. Requires not `fill-extrusion-pattern`. |
| `fill-extrusion-translate` | P | array<number, 2> | `[0, 0]` | pixels | zoom (interp) T | 0.27.0 | 5.1.0 | The geometry's offset. Values are `[x, y]` where negatives indicate left and up (on the flat plane), respectively. |
| `fill-extrusion-translate-anchor` | P | enum | `"map"` |  | zoom | 0.27.0 | 5.1.0 | Controls the frame of reference for [`fill-extrusion-translate`]. Values: `map`, `viewport`. Requires `fill-extrusion-translate`. |
| `fill-extrusion-pattern` | P | resolvedImage |  |  | zoom,feature T | 0.27.0; dd 0.49.0 | 5.1.0; dd 6.5.0 | Name of image in sprite to use for drawing images on extruded fills. For seamless patterns, image width and height must be a factor of two (2, 4, 8, ..., 512). |
| `fill-extrusion-height` | P | number | `0` | meters | zoom,feature,state (interp) T | 0.27.0; dd 0.27.0 | 5.1.0; dd 5.1.0 | The height with which to extrude this layer. Negative values extrude below ground level, so a floor that is entirely underground can be expressed. |
| `fill-extrusion-base` | P | number | `0` | meters | zoom,feature,state (interp) T | 0.27.0; dd 0.27.0 | 5.1.0; dd 5.1.0 | The height with which to extrude the base of this layer. Must be less than or equal to [`fill-extrusion-height`]. Requires `fill-extrusion-height`. |
| `fill-extrusion-vertical-gradient` | P | boolean | `true` |  | zoom | 0.50.0 | 7.0.0 | Whether to apply a vertical gradient to the sides of a fill-extrusion layer. If true, sides will be shaded slightly darker farther down. |

## raster

| Property | Kind | Type | Default | Range/units | Expr | JS | Native | Notes |
|---|---|---|---|---|---|---|---|---|
| `raster-opacity` | P | number | `1` | 0..1 | zoom (interp) T | 0.10.0 | 2.0.1 | The opacity at which the image will be drawn. |
| `raster-hue-rotate` | P | number | `0` | degrees | zoom (interp) T | 0.10.0 | 2.0.1 | Rotates hues around the color wheel. |
| `raster-brightness-min` | P | number | `0` | 0..1 | zoom (interp) T | 0.10.0 | 2.0.1 | Increase or reduce the brightness of the image. The value is the minimum brightness. |
| `raster-brightness-max` | P | number | `1` | 0..1 | zoom (interp) T | 0.10.0 | 2.0.1 | Increase or reduce the brightness of the image. The value is the maximum brightness. |
| `raster-saturation` | P | number | `0` | -1..1 | zoom (interp) T | 0.10.0 | 2.0.1 | Increase or reduce the saturation of the image. |
| `raster-contrast` | P | number | `0` | -1..1 | zoom (interp) T | 0.10.0 | 2.0.1 | Increase or reduce the contrast of the image. |
| `resampling` | P | enum | `"linear"` |  | zoom | 5.20.0 | no | The resampling/interpolation method to use for overscaling, also known as texture magnification filter. Values: `linear`, `nearest`. |
| `raster-resampling` | P | enum | `"linear"` |  | zoom | 0.47.0 | 6.3.0 | The resampling/interpolation method to use for overscaling, also known as texture magnification filter. It is advised to use the generic `resampling` paint property instead. Values: `linear`, `nearest`. |
| `raster-fade-duration` | P | number | `300` | 0.. milliseconds | zoom (interp) | 0.10.0 | 2.0.1 | Fade duration when a new tile is added, or when a video is started or its coordinates are updated. |

## hillshade

| Property | Kind | Type | Default | Range/units | Expr | JS | Native | Notes |
|---|---|---|---|---|---|---|---|---|
| `hillshade-illumination-direction` | P | numberArray | `335` | 0..359 | zoom (interp) | 0.43.0 | 6.0.0 | The direction of the light source(s) used to generate the hillshading with 0 as the top of the viewport if [`hillshade-illumination-anchor`] is set to `viewport` and due north if `hillshade-illumination-anchor` is set to `map`. Only when [`hillshade-method`] is set to `multidirectional` can you specify multiple light sources. |
| `hillshade-illumination-altitude` | P | numberArray | `45` | 0..90 | zoom (interp) | 5.5.0 | 13.0.0 | The altitude of the light source(s) used to generate the hillshading with 0 as sunset and 90 as noon. Only when [`hillshade-method`] is set to `multidirectional` can you specify multiple light sources. |
| `hillshade-illumination-anchor` | P | enum | `"viewport"` |  | zoom | 0.43.0 | 6.0.0 | Direction of light source when map is rotated. Values: `map`, `viewport`. |
| `hillshade-exaggeration` | P | number | `0.5` | 0..1 | zoom (interp) T | 0.43.0 | 6.0.0 | Intensity of the hillshade |
| `hillshade-shadow-color` | P | colorArray | `"#000000"` |  | zoom (interp) T | 0.43.0 | 6.0.0 | The shading color of areas that face away from the light source(s). Only when [`hillshade-method`] is set to `multidirectional` can you specify multiple light sources. |
| `hillshade-highlight-color` | P | colorArray | `"#FFFFFF"` |  | zoom (interp) T | 0.43.0 | 6.0.0 | The shading color of areas that faces towards the light source(s). Only when [`hillshade-method`] is set to `multidirectional` can you specify multiple light sources. |
| `hillshade-accent-color` | P | color | `"#000000"` |  | zoom (interp) T | 0.43.0 | 6.0.0 | The shading color used to accentuate rugged terrain like sharp cliffs and gorges. |
| `hillshade-method` | P | enum | `"standard"` |  | zoom | 5.5.0 | 13.0.0 | The hillshade algorithm to use, one of `standard`, `basic`, `combined`, `igor`, or `multidirectional`. Values: `standard`, `basic`, `combined`, `igor`, `multidirectional`. |
| `resampling` | P | enum | `"linear"` |  | zoom | 5.20.0 | no | The resampling/interpolation method to use for overscaling, also known as texture magnification filter. Values: `linear`, `nearest`. |

## color-relief

| Property | Kind | Type | Default | Range/units | Expr | JS | Native | Notes |
|---|---|---|---|---|---|---|---|---|
| `color-relief-opacity` | P | number | `1` | 0..1 | zoom (interp) T | 5.6.0 | 13.0.0 | The opacity at which the color-relief will be drawn. |
| `color-relief-color` | P | color |  |  | elevation (interp) | 5.6.0 | 13.0.0 | Defines the color of each pixel based on its elevation. Should be an expression that uses the [`elevation`] expression as input. |
| `resampling` | P | enum | `"linear"` |  | zoom | 5.20.0 | no | The resampling/interpolation method to use for overscaling, also known as texture magnification filter. Values: `linear`, `nearest`. |
