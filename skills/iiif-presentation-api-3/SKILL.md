---
name: iiif-presentation-api-3
description: >
  Use this skill when creating, editing, validating, or debugging IIIF
  Presentation API 3.0 JSON-LD: Manifests, Collections, Canvases, Ranges,
  AnnotationPages, and painting or supplementing Annotations, as consumed by
  viewers such as Mirador, Universal Viewer, TIFY, or Clover. Use it whenever
  JSON has "@context": "http://iiif.io/api/presentation/3/context.json" or
  "type": "Manifest", when publishing images, books, artworks, audio, or video
  as IIIF, when adding region annotations or a table of contents to a
  manifest, or when converting a 2.x manifest to 3.0, even if the user only
  says "make a manifest" or "describe this object for a IIIF viewer". Not for
  the Image API pixel protocol or Presentation 2.x output.
license: MIT
metadata:
  spec-source: https://iiif.io/api/presentation/3.0/
  version: "1.0"
---

# IIIF Presentation API 3.0

Use this skill to build valid IIIF Presentation 3.0 resources. A full working manifest is in [assets/manifest.example.json](assets/manifest.example.json). Start from it whenever you create a manifest.

## Resource hierarchy

```text
Collection ── items → Collection | Manifest
Manifest   ── items → Canvas (at least one)
           ── structures → Range (table of contents) ── items → Range | Canvas | SpecificResource
           ── annotations → AnnotationPage (non-painting)
Canvas     ── items → AnnotationPage ── items → Annotation (motivation: painting)
           ── annotations → AnnotationPage (supplementing, commenting, …)
```

## Required and recommended properties

| Resource | Required | Recommended |
|---|---|---|
| Collection | `id`, `type`, `label`, `items` | `metadata`, `summary`, `provider`, `thumbnail` |
| Manifest | `id`, `type`, `label`, `items` | `metadata`, `summary`, `provider`, `thumbnail` |
| Canvas | `id`, `type`, and `height`+`width` and/or `duration` | `label`, `items` |
| Range | `id`, `type`, `items` | `label` |
| AnnotationPage | `id`, `type` | `items` |
| Annotation | `id`, `type`, `motivation`, `target` (plus `body` for content) | |
| Content resource | `id`, `type` | `format`, plus `height`/`width`/`duration` and `language` where relevant |

**Every** resource, including content resources and services, needs `id` and `type`. For resources this spec defines, `id` is an HTTP(S) URI.

## Property shapes

- **Language maps** are used for `label`, `summary`, and the `label`/`value` of metadata entries. Each key is a BCP 47 language code or `none`, and each value is **an array** of strings: `{ "en": ["Title"], "none": ["Whistler (1871)"] }`.
- **`metadata`** is an array of `{ "label": {…}, "value": {…} }` objects. **`requiredStatement`** is a single object of the same shape. Clients must display it.
- **`rights`** is one string: a Creative Commons or RightsStatements.org URI, such as `http://creativecommons.org/licenses/by/4.0/`.
- **These are always arrays:** `items`, `structures`, `annotations`, `thumbnail`, `provider`, `homepage`, `rendering`, `seeAlso`, `partOf`, `service`, `services`, `behavior`, `language`.
- **These are single objects or strings:** `requiredStatement`, `start`, `supplementary`, `placeholderCanvas`, `accompanyingCanvas`, `rights`, `navDate`, `viewingDirection`, `timeMode`, `format`, `height`, `width`, `duration`.
- **`provider`** entries are `{ "id", "type": "Agent", "label", "homepage": [...], "logo": [...] }`.
- **`viewingDirection`** is one of `left-to-right` (the default), `right-to-left`, `top-to-bottom`, or `bottom-to-top`. It is allowed on Collection, Manifest, and Range.
- **`behavior`** values:
  - Layout: `individuals`, `paged`, `continuous`, `unordered`, plus `facing-pages` and `non-paged` on Canvas.
  - Time: `auto-advance`, `no-auto-advance`, `repeat`, `no-repeat`.
  - Collection only: `multi-part`, `together`.
  - Range only: `sequence`, `thumbnail-nav`, `no-nav`.
  - Annotations and pages: `hidden`.
- Minimal HTML (`a`, `b`, `br`, `i`, `img`, `p`, `small`, `span`, `sub`, `sup`) is allowed **only** in `summary` and metadata `value`. The string must start with `<` and end with `>`. It is never allowed in `label`.

## Painting an image onto a Canvas

```json
{
  "id": "https://example.org/iiif/book1/annotation/p1-image",
  "type": "Annotation",
  "motivation": "painting",
  "body": {
    "id": "https://example.org/iiif2/page1/full/full/0/default.jpg",
    "type": "Image",
    "format": "image/jpeg",
    "width": 3000,
    "height": 4000,
    "service": [{
      "@id": "https://example.org/iiif2/page1",
      "@type": "ImageService2",
      "profile": "http://iiif.io/api/image/2/level1.json"
    }]
  },
  "target": "https://example.org/iiif/book1/canvas/p1"
}
```

- The body `id` is a real image URL. It **must not** be just the image-service base URI.
- **Image API 2 service**: `@id`, `@type: "ImageService2"`, and `profile` as the full level URI.
- **Image API 3 service**: `id`, `type: "ImageService3"`, and `profile: "level1"`.
- The Canvas usually has the same `width` and `height` as the image. If it does not, the image is scaled to fill the Canvas.

## Targeting a region

For annotations about part of a canvas, such as transcriptions, labels, or comments, put an AnnotationPage in the Canvas's `annotations` (not in `items`) and use a SpecificResource target:

```json
{
  "id": "https://example.org/iiif/book1/annotation/p1-note1",
  "type": "Annotation",
  "motivation": "supplementing",
  "body": { "type": "TextualBody", "value": "Signature", "language": "en", "format": "text/plain" },
  "target": {
    "type": "SpecificResource",
    "source": { "id": "https://example.org/iiif/book1/canvas/p1", "type": "Canvas" },
    "selector": {
      "type": "FragmentSelector",
      "conformsTo": "http://www.w3.org/TR/media-frags/",
      "value": "xywh=100,200,300,400"
    }
  }
}
```

The shorthand `"target": "https://…/canvas/p1#xywh=100,200,300,400"` is also valid. For time-based media, use `t=start,end`. When a fragment has both parts, put `t=` first and join them with `&`.

These annotations follow the W3C Web Annotation model for selectors, bodies, and other motivations such as `commenting` and `tagging`. Use the `w3c-web-annotation-model` skill, if installed, for selector details.

## Procedure: create a manifest

1. Copy [assets/manifest.example.json](assets/manifest.example.json).
2. Set the manifest `id` to the URL where the manifest will be served, then set `label`, `summary`, `metadata`, `rights`, `requiredStatement`, and `provider`.
3. For each image, view, or page, add one Canvas:
   - Give it a unique `id` with no fragment, plus `label`, `width`, and `height`. Use the image's `width` and `height` from the image server's `info.json`.
   - Add one AnnotationPage with one `painting` Annotation that targets the Canvas `id`.
4. Add a `thumbnail` to the Manifest, and optionally to each Canvas.
5. Optionally add `structures` (Ranges) for chapters, and `annotations` for supplementary content.
6. Validate:
   - The JSON parses and `@context` is the first key, appearing only at the top level.
   - Every resource has `id` and `type`, and every `id` is unique.
   - Every painting target matches an existing Canvas `id`.
   - Multi-valued properties are arrays.
   - For a full check, run the official validator at <https://presentation-validator.iiif.io/>.

## Gotchas

- **The context is `http://iiif.io/api/presentation/3/context.json`.** It has `/3/`, not `/3.0/`, and starts with `http`. If you use extension contexts, list them before it so that it comes last.
- **Every Annotation needs its own `id`**, and a SpecificResource needs `"type": "SpecificResource"`. Older examples often leave these out.
- **Canvas `height` and `width` go together**: if one is present, both are required. Canvas IDs must not contain `#`.
- **`painting` annotations go only in Canvas `items`.** Anything listed under `annotations` must not use `painting`.
- **Keep the 2.x names out of 3.0.** Use `@id`→`id`, `@type`→`type`, `description`→`summary`, `attribution`→`requiredStatement`, `license`→`rights`, `viewingHint`→`behavior`, `related`→`homepage`, `within`→`partOf`, `startCanvas`→`start`, and `otherContent`→`annotations`. `sequences` and Canvas `images` no longer exist; use `items`.
- **A bare string is not a valid language-map value**, so `"label": "Title"` is invalid. Write `"label": { "en": ["Title"] }`.
- **IIIF `label` is a language map, but a W3C TextualBody uses `value` + `language`.** Do not wrap the `value` in a language map.
- **`rights` accepts only CC, RightsStatements.org, or registered extension URIs.** Put free-text rights statements in `requiredStatement`.
- **Clients must ignore motivations they do not recognize**, so a misspelled `painting` makes the image silently disappear.
