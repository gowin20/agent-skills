# Sprites

A **sprite** is an image atlas: one PNG holding many small images, plus a JSON index describing where each image sits. Symbol layers use sprite images as icons, and fill, line, fill-extrusion, and background layers use them as patterns.

## Contents

- [Declaring sprites](#declaring-sprites)
- [Using sprite images](#using-sprite-images)
- [Sprite file format](#sprite-file-format)
- [High-DPI files](#high-dpi-files)
- [Stretchable images and icon-text-fit](#stretchable-images-and-icon-text-fit)
- [SDF icons](#sdf-icons)
- [Patterns](#patterns)
- [Generating sprites](#generating-sprites)
- [Runtime images](#runtime-images)
- [Troubleshooting](#troubleshooting)

## Declaring sprites

**Single sprite.** Give the base URL **without** an extension:

```json
"sprite": "https://demotiles.maplibre.org/styles/osm-bright-gl-style/sprite"
```

The renderer loads `sprite.json` and `sprite.png` (or `sprite@2x.json` and `sprite@2x.png` on high-DPI screens). Image names are used without a prefix.

**Multiple sprites.** Give an array of `{id, url}` objects. IDs and URLs must be unique:

```json
"sprite": [
  { "id": "roadsigns", "url": "https://example.com/myroadsigns" },
  { "id": "shops", "url": "https://example2.com/someurl" },
  { "id": "default", "url": "https://example2.com/anotherurl" }
]
```

Reference images as `<id>:<image>`, e.g. `roadsigns:stop_sign`. Images in the sprite with ID `default` need no prefix (`airport`, not `default:airport`).

## Using sprite images

Properties that take a sprite image (type `resolvedImage`):

| Property | Layer type | Kind |
|---|---|---|
| `icon-image` | symbol | layout |
| `fill-pattern` | fill | paint |
| `line-pattern` | line | paint |
| `fill-extrusion-pattern` | fill-extrusion | paint |
| `background-pattern` | background | paint |

Values can be a name, an expression, or (legacy) a `{token}` string:

```json
"icon-image": "airport_11"
"icon-image": ["concat", ["get", "class"], "_11"]
"icon-image": ["coalesce", ["image", ["concat", ["get", "class"], "_11"]], ["image", "circle_11"]]
"icon-image": "{class}_11"
```

Prefer expressions over `{token}` strings. Use `["image", name]` with `coalesce` to fall back when an image is missing. Images can also be embedded in labels: `["format", ["image", "flag_de"], {}, " Berlin", {}]`.

Size icons with `icon-size` (a scale factor; 1 = native size in CSS pixels). Pixel ratio is handled by `pixelRatio` in the index, so a 2x image of 34×34 px with `pixelRatio: 2` displays at 17×17.

## Sprite file format

The PNG holds the images. The JSON index maps each image name to its rectangle in the PNG:

```json
{
  "airport_11": { "x": 0, "y": 21, "width": 17, "height": 17, "pixelRatio": 1 },
  "poi":        { "x": 17, "y": 21, "width": 32, "height": 32, "pixelRatio": 1, "sdf": true }
}
```

Required keys:

| Key | Meaning |
|---|---|
| `x`, `y` | Top-left corner of the image in the PNG, in pixels. |
| `width`, `height` | Image size in PNG pixels. |
| `pixelRatio` | Device pixel ratio the image was drawn for: 1 for `sprite.png`, 2 for `sprite@2x.png`. |

Optional keys:

| Key | Meaning |
|---|---|
| `sdf` | `true` if the image is a signed-distance field; enables `icon-color` and `icon-halo-*`. Default `false`. |
| `content` | `[left, top, right, bottom]` box that text must fit inside when `icon-text-fit` is used. |
| `stretchX` | Array of `[from, to]` pixel ranges that may stretch horizontally. |
| `stretchY` | Array of `[from, to]` pixel ranges that may stretch vertically. |
| `textFitWidth` | `stretchOrShrink` (default), `stretchOnly`, or `proportional`: how the content box scales horizontally with `icon-text-fit: both`. GL JS 4.2.0, Android 11.4.0, iOS 6.6.0. |
| `textFitHeight` | Same, vertically. |

## High-DPI files

On screens with a device pixel ratio above 1, renderers request `@2x` files: `https://example.com/sprite@2x.json` and `https://example.com/sprite@2x.png`. Publish both sets; the 2x index uses the same names with doubled coordinates and `"pixelRatio": 2`. If the `@2x` files are missing, icons fail to load on those screens.

## Stretchable images and icon-text-fit

Highway shields and label backgrounds stretch to fit their text. Mark the stretchable parts and the text area in the index:

```json
{
  "shield": {
    "x": 0, "y": 0, "width": 25, "height": 30, "pixelRatio": 1,
    "stretchX": [[5, 10], [15, 20]],
    "stretchY": [[5, 20]],
    "content": [5, 5, 20, 25]
  }
}
```

Then in the symbol layer:

```json
"layout": {
  "text-field": ["get", "ref"],
  "text-font": ["Noto Sans Bold"],
  "icon-image": "shield",
  "icon-text-fit": "both",
  "icon-text-fit-padding": [2, 4, 2, 4]
}
```

Pixels outside the `stretchX`/`stretchY` ranges keep their size (corners, borders); pixels inside are stretched. `textFitWidth: "stretchOnly"` with `textFitHeight: "proportional"` keeps narrow shields (a single "1") from becoming slimmer than the original shape.

## SDF icons

An SDF icon stores distance to the shape's edge instead of colors, so the renderer can recolor it:

```json
"paint": { "icon-color": ["match", ["get", "kind"], "hospital", "#d32f2f", "#1976d2"], "icon-halo-color": "#fff", "icon-halo-width": 1 }
```

- Set `"sdf": true` on the image in the index (or `{sdf: true}` when adding it at runtime).
- An SDF image renders in a single color (plus halo); multicolor icons cannot be SDF.
- Do not mix SDF and non-SDF icons in one symbol layer; GL JS warns that they cannot share a buffer and some icons render wrongly. Split them into two layers.
- `icon-halo-width` is in pixels only for SDF images generated with a blur radius of 8 (16 for `@2x`).
- spreet generates SDF sprites with `--sdf`.

## Patterns

- `fill-pattern`, `line-pattern`, `fill-extrusion-pattern`, and `background-pattern` repeat a sprite image.
- For seamless tiling, width and height must be powers of two (2, 4, 8, ..., 512); `line-pattern` needs only the width to be a power of two.
- A pattern replaces the color (`fill-color`, `line-color` are ignored for drawing). With an SDF fill pattern, `fill-color` becomes the pattern's foreground color where supported (not yet in released GL JS or Native).
- Patterns are cross-faded between integer zooms when they change by zoom.

## Generating sprites

Build sprites from a folder of SVGs:

- [spreet](https://github.com/flother/spreet) (Rust CLI):

  ```bash
  spreet icons/ sprite              # sprite.png + sprite.json
  spreet --retina icons/ sprite@2x  # sprite@2x.png + sprite@2x.json
  spreet --sdf icons/ sprite-sdf    # SDF sprite
  ```

- [@elastic/spritezero](https://www.npmjs.com/package/@elastic/spritezero) (Node), a maintained fork of spritezero.

File names (without `.svg`) become image names. Keep names stable; styles reference them.

## Runtime images

Platforms can add images that the style references by name without a sprite (GL JS `map.addImage(name, image, {pixelRatio, sdf, stretchX, stretchY, content})`, plus the `styleimagemissing` event to supply images on demand). The style JSON is the same: `"icon-image": "my-runtime-icon"`. `["image", name]` returns `null` until the image is added.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| All icons missing | Sprite URL includes `.json` or `.png`; 404 or CORS failure on the sprite files; `@2x` files missing on a high-DPI screen. |
| Some icons missing | Name typo; missing `id:` prefix with multiple sprites; the expression builds a name that does not exist (log: `Image "x" could not be loaded`). |
| Icons blurry or wrong size | Wrong `pixelRatio`, or a 1x image served as 2x. |
| `icon-color` has no effect | Image is not SDF. |
| Visible seams in patterns | Pattern dimensions are not powers of two. |
| Shield does not stretch | `icon-text-fit` not set, or `stretchX`/`stretchY`/`content` missing from the index. |
