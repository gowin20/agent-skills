---
name: iiif-image-api-2
description: >
  Use this skill when building, parsing, validating, or debugging IIIF Image
  API 2.0 requests: image URLs of the form
  /{identifier}/{region}/{size}/{rotation}/{quality}.{format}, info.json
  documents, tile requests for deep-zoom viewers such as OpenSeadragon or
  Leaflet-IIIF, and compliance levels 0, 1, and 2. Use it whenever a URL
  contains a path like /full/full/0/default.jpg or a context of
  http://iiif.io/api/image/2/context.json, when cropping, scaling, or
  thumbnailing images from an IIIF server, or when an "ImageService2"
  appears in a manifest. Not for IIIF Image API 3.0 (which adds max, square,
  and different info.json keys) or for Presentation API manifests.
license: MIT
metadata:
  spec-source: https://iiif.io/api/image/2.0/
  version: "1.0"
---

# IIIF Image API 2.0

Use this skill to build and interpret image requests and `info.json` documents for IIIF Image API **2.0** servers.

## URI templates

```text
Image:  {scheme}://{server}{/prefix}/{identifier}/{region}/{size}/{rotation}/{quality}.{format}
Info:   {scheme}://{server}{/prefix}/{identifier}/info.json
Base:   {scheme}://{server}{/prefix}/{identifier}
```

Example: `https://example.org/iiif/abcd1234/full/full/0/default.jpg`

- **Percent-encode the identifier**, including `/ ? # [ ] @ %` and non-ASCII characters. Do not encode `:`. The `/` separators between parameters must **not** be encoded. Example: `ark:/12025/654xz321` becomes `ark:%2F12025%2F654xz321`.
- Requesting the base URI should redirect (303) to `info.json`.

## Request parameters

The server applies the parameters in this order: **region → size → rotation → quality → format**.

| Parameter | Value | Meaning |
|---|---|---|
| region | `full` | Whole image |
| | `x,y,w,h` | Pixels (integers) from the top-left corner |
| | `pct:x,y,w,h` | Percentages of the full image (floats allowed) |
| size | `full` | No scaling |
| | `w,` | Width is `w`, height keeps the aspect ratio |
| | `,h` | Height is `h`, width keeps the aspect ratio |
| | `pct:n` | `n`% of the extracted region |
| | `w,h` | Exactly `w`×`h` (may distort) |
| | `!w,h` | Best fit inside `w`×`h`, keeping the aspect ratio |
| rotation | `n` | Degrees clockwise, from 0 to 360 (floats allowed) |
| | `!n` | Mirror (reflect across the **vertical** axis, a left-right flip), **then** rotate `n` |
| quality | `default`, `color`, `gray`, `bitonal` | `default` is whatever the server's natural quality is |
| format | `jpg`, `png`, `tif`, `gif`, `jp2`, `pdf`, `webp` | |

The region is cropped at the image edge, not padded. A region with zero size, or one entirely outside the image, returns 400.

## info.json

```json
{
  "@context": "http://iiif.io/api/image/2/context.json",
  "@id": "https://example.org/iiif/abcd1234",
  "protocol": "http://iiif.io/api/image",
  "width": 6000,
  "height": 4000,
  "sizes": [{ "width": 150, "height": 100 }, { "width": 600, "height": 400 }],
  "tiles": [{ "width": 512, "scaleFactors": [1, 2, 4, 8, 16] }],
  "profile": [
    "http://iiif.io/api/image/2/level1.json",
    { "formats": ["png"], "qualities": ["gray"], "supports": ["regionByPct", "sizeByForcedWh"] }
  ]
}
```

- **Required:** `@context`, `@id` (the base URI, with no trailing slash), `protocol`, `width`, `height`, `profile`.
- **Optional:** `sizes` (each entry needs `width` and `height`), `tiles` (`width` and `scaleFactors` are required; `height` defaults to `width`), `service`.
- `profile[0]` **must** be the compliance level URI. An optional object after it lists extra `formats`, `qualities`, and `supports` beyond that level. If a feature is not declared, assume it is not supported.
- `supports` feature names: `baseUriRedirect`, `canonicalLinkHeader`, `cors`, `jsonldMediaType`, `mirroring`, `profileLinkHeader`, `regionByPct`, `regionByPx`, `rotationArbitrary`, `rotationBy90s`, `sizeAboveFull`, `sizeByWhListed`, `sizeByForcedWh` (`!w,h`), `sizeByH`, `sizeByPct`, `sizeByW`, `sizeByWh` (`w,h`).

## Compliance levels

R = required. Any feature not marked R is optional at that level.

| Feature | L0 | L1 | L2 |
|---|---|---|---|
| region `full` | R | R | R |
| region `x,y,w,h` | | R | R |
| region `pct:` | | | R |
| size `full`, and `w,h` for sizes listed in `sizes` | R | R | R |
| size `w,` `,h` `pct:n` | | R | R |
| size `w,h` (any), `!w,h` | | | R |
| rotation `0` | R | R | R |
| rotation 90/180/270 | | | R |
| quality `default` | R | R | R |
| quality `bitonal` | | | R |
| quality `color`, `gray` | | | R (if applicable) |
| format `jpg` | R | R | R |
| format `png` | | | R |
| base URI redirect, CORS, JSON-LD media type | | R | R |

Arbitrary rotation, mirroring, and upscaling beyond full size (`sizeAboveFull`) are optional at every level.

## Procedure: build a request

1. Fetch `info.json`. Read `width`, `height`, `profile[0]` (the level), any extras in `profile[1]`, `sizes`, and `tiles`.
2. Choose the region and size using **only the features the server supports**. Requests that work at every level from 1 upward:
   - Thumbnail: `/full/{w},/0/default.jpg`
   - Crop: `/{x},{y},{w},{h}/{w2},/0/default.jpg`
3. At **level 0**, request only `full/full`, sizes listed in `sizes` (as `w,h`), or tiles that match the declared `tiles`.
4. Use **canonical form** where possible: region is `full` or `x,y,w,h`; size is `full`, `w,`, or `w,h` (only when the aspect ratio changes); rotation is an integer where possible; quality is `default` unless you need another.
5. For tiles at scale factor `s` with tile width `t` (and `th`, which equals `t` unless `height` is given):
   - Region: `x = col·t·s`, `y = row·th·s`, `w = min(t·s, width − x)`, `h = min(th·s, height − y)`.
   - Size: `ceil(w / s),`
   - Edge tiles are partial.

## HTTP behaviour

- An `info.json` response is `application/json`, or `application/ld+json` if the client sends that `Accept` header. Servers at level 1 and above send `Access-Control-Allow-Origin: *`.
- Errors:
  - 400: bad syntax, a zero-size result, or an unsupported parameter value.
  - 401 / 403: authentication or authorisation.
  - 404: unknown identifier, or a parameter value not supported for this image.
  - 501: a valid request the server does not implement.
  - 500 / 503: server problems.
- Servers may redirect a non-canonical request to its canonical URI, and may send `Link: <...>;rel="canonical"`.

## Gotchas

- **2.0 has no `max` size and no `square` region.** Those were added in 2.1 and 3.0. Use `full`.
- **There is no `native` quality.** That was Image API 1.x. Use `default`.
- **`@context` uses `http://`, not `https://`.** It is matched as an exact string.
- **v2 keys start with `@`** (`@id`, `@context`). Image API 3 uses `id`, `type`, and `profile: "level1"`, so check which version you are reading.
- **Level 1 does not guarantee `pct:` regions, `!w,h`, or arbitrary `w,h`.** Check `profile[1].supports` before using them.
- **`pct:n` in size is relative to the extracted region**, not to the full image.
- **Pixel dimensions are integers.** Round consistently yourself. The spec leaves rounding to the implementation.
- **Rotations that are not multiples of 90 enlarge the bounding box.** Request `png` to get transparent corners.
