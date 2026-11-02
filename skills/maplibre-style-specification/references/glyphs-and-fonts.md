# Glyphs and fonts

MapLibre does not draw text from font files directly (except via `font-faces` on Native). It draws **glyphs**: pre-rendered signed-distance-field (SDF) bitmaps packed in Protocol Buffer (`.pbf`) files, 256 Unicode code points per file. One SDF glyph renders sharply at any size and supports halos and color changes on the GPU.

## Contents

- [The glyphs URL](#the-glyphs-url)
- [Font stacks and text-font](#font-stacks-and-text-font)
- [Hosting glyphs](#hosting-glyphs)
- [Generating glyph PBFs](#generating-glyph-pbfs)
- [Non-Latin scripts](#non-latin-scripts)
- [font-faces](#font-faces)
- [Local fonts without glyphs](#local-fonts-without-glyphs)
- [Troubleshooting labels](#troubleshooting-labels)

## The glyphs URL

```json
"glyphs": "https://demotiles.maplibre.org/font/{fontstack}/{range}.pbf"
```

- Must be an **absolute** URL (scheme, host, path). Relative URLs are invalid; renderers may also accept custom registered protocols.
- Must contain both tokens. The validator rejects a URL missing either one.
  - `{fontstack}`: the layer's `text-font` array joined by commas, e.g. `Open Sans Semibold` or `Noto Sans Regular,Noto Sans Bold`. Spaces arrive percent-encoded (`Open%20Sans%20Semibold`).
  - `{range}`: a block of 256 code points: `0-255`, `256-511`, ..., `65280-65535`. The renderer works out which blocks the visible text needs and fetches only those.
- Required whenever a `symbol` layer has `text-field` (except GL JS 5.11.0+, see [Local fonts](#local-fonts-without-glyphs)). Icons alone do not need glyphs.

## Font stacks and text-font

`text-font` is a layout property of `symbol` layers. Default: `["Open Sans Regular", "Arial Unicode MS Regular"]`.

```json
"layout": {
  "text-field": ["get", "name"],
  "text-font": ["Noto Sans Regular"]
}
```

- Names must match the server's font names **exactly**, including case and weight words (`Regular`, `Semibold`, `Bold`, `Italic`).
- The whole array is one font stack. A **dynamic** glyph server (e.g. TileServer GL, Martin, MapTiler) combines the fonts, taking each glyph from the first font that has it. A **static** host only has one directory per font, so a multi-font stack like `Open Sans Semibold,Arial Unicode MS Regular` returns 404. On static hosting, use one font per `text-font` array, or pre-build a directory for the combined stack.
- The default `text-font` is not available on many servers (the MapLibre demo server has `Open Sans Semibold`, `Noto Sans Regular`, and `Noto Sans Bold`, but not `Open Sans Regular` or `Arial Unicode MS Regular`). **Always set `text-font` explicitly.**
- `text-font` accepts expressions, e.g. a bold font for important features: `["case", [">", ["get", "rank"], 5], ["literal", ["Noto Sans Bold"]], ["literal", ["Noto Sans Regular"]]]`. Each distinct stack is fetched separately.
- Inside a `format` expression, override the font per section with `{"text-font": ["literal", ["Noto Sans Bold"]]}`.

## Hosting glyphs

A static glyph set is a directory per font with 256 files:

```text
fonts/
├── Noto Sans Regular/
│   ├── 0-255.pbf
│   ├── 256-511.pbf
│   └── ... 65280-65535.pbf
└── Noto Sans Bold/
    └── ...
```

Serve with CORS headers (`Access-Control-Allow-Origin`) when the map runs on another origin. Serve `.pbf` as `application/x-protobuf` or `application/octet-stream`. If files are gzip-compressed on disk, send `Content-Encoding: gzip`. A missing range (for scripts the font lacks) can be an empty file or a 404; a 404 logs an error but other ranges keep working.

Ready-made glyph sets:

- MapLibre demo server: `https://demotiles.maplibre.org/font/{fontstack}/{range}.pbf` (for demos and testing).
- [openmaptiles/fonts](https://github.com/openmaptiles/fonts): Noto Sans, Open Sans, PT Sans, Roboto, Metropolis.
- [protomaps/basemaps-assets](https://github.com/protomaps/basemaps-assets): Noto Sans glyphs used by Protomaps basemaps.

## Generating glyph PBFs

Convert TTF/OTF fonts to SDF PBF ranges with one of:

- [maplibre/font-maker](https://github.com/maplibre/font-maker): browser tool and CLI; can merge several font files into one named stack.
- [stadiamaps/sdf_font_tools](https://github.com/stadiamaps/sdf_font_tools) (`build_pbf_glyphs`): Rust CLI, fast for whole directories.
- [openmaptiles/fonts](https://github.com/openmaptiles/fonts): Node script built on node-fontnik; edit its config to add fonts.

The font name in the PBF (and the directory name) is what `text-font` must match. Check the license of every font you convert.

## Non-Latin scripts

- **CJK** (Chinese, Japanese, Korean): ranges for ideographs are large. GL JS by default renders CJK ideographs and syllables from a local font (map option `localIdeographFontFamily`, default `"sans-serif"`) instead of downloading glyphs; set it to `false` to use server glyphs.
- **Right-to-left** (Arabic, Hebrew): GL JS needs the RTL text plugin (`maplibregl.setRTLTextPlugin(...)` with `@mapbox/mapbox-gl-rtl-text`). Without it, text renders in the wrong order and `["is-supported-script", s]` returns `false`. MapLibre Native handles bidirectional text itself.
- **Complex shaping** (Devanagari, Bengali, Khmer, Myanmar, ...): SDF glyph PBFs cannot shape these scripts. Use `font-faces` on MapLibre Native. In GL JS, either accept imperfect rendering or filter such labels with `is-supported-script`.
- Fallback for scripts your main font lacks: include a broad-coverage font in a dynamic server's stack, or merge fonts into one stack with font-maker.

## font-faces

The root `font-faces` object maps a `text-font` name to real font files, so the renderer can shape complex scripts:

```json
"font-faces": {
  "Noto Sans Regular": [
    { "url": "https://cdn.jsdelivr.net/gh/notofonts/notofonts.github.io/fonts/NotoSansKhmer/hinted/ttf/NotoSansKhmer-Regular.ttf", "unicode-range": ["U+1780-17FF"] },
    { "url": "https://cdn.jsdelivr.net/gh/notofonts/notofonts.github.io/fonts/NotoSansDevanagari/hinted/ttf/NotoSansDevanagari-Regular.ttf", "unicode-range": ["U+0900-097F"] }
  ],
  "Unifont": "https://ftp.gnu.org/gnu/unifont/unifont-15.0.01/unifont-15.0.01.ttf"
}
```

- A value is a URL string or an array of `{url, unicode-range}` objects.
- `unicode-range` is optional (default `U+0-10FFFF`), an array of CSS-style ranges.
- Resolution: for each name in `text-font`, characters covered by that entry's font files use them; remaining characters fall back to the `glyphs` URL.
- Which font formats work is up to the implementation; unsupported files are ignored.
- Support: MapLibre Native Android 11.13.0, iOS 6.18.0. Not yet in GL JS.

## Local fonts without glyphs

In GL JS 5.11.0+, if `glyphs` is omitted, text renders with fonts installed on the user's system (glyphs are rasterized locally). Choosing a specific local font through `text-font` is not yet supported, and Native still requires `glyphs`. Use this for prototypes, not production styles.

## Troubleshooting labels

Work through these in order:

1. Is there a root `glyphs` URL with `{fontstack}` and `{range}`?
2. Does `https://.../<your text-font stack>/0-255.pbf` return 200 in a browser? A 404 means the font name or stack is wrong for that server.
3. Is `text-font` set explicitly? The default stack is missing on most servers.
4. Does `text-field` resolve to a non-empty string for these features? Test with a constant: `"text-field": "TEST"`.
5. Is the layer visible at this zoom (`minzoom`, `maxzoom`, `filter`) and does the `source-layer` exist?
6. Are labels hidden by collisions? Temporarily set `"text-allow-overlap": true` and `"text-ignore-placement": true`.
7. Is the text the same color as the background, or `text-opacity` 0?
8. Check the browser console and network tab for glyph or CORS errors.
