---
name: w3c-web-annotation-model
description: >
  Use this skill when creating, parsing, validating, or debugging W3C Web
  Annotations, the JSON-LD data model with @context
  http://www.w3.org/ns/anno.jsonld that links a body to a target. Covers
  bodies (TextualBody, external resources, Choice), targets and
  SpecificResource, every selector type (FragmentSelector with media
  fragments xywh=, SvgSelector, TextQuoteSelector, TextPositionSelector,
  CssSelector, XPathSelector, DataPositionSelector, RangeSelector, refinedBy),
  states, motivations, agents, and AnnotationCollection/AnnotationPage. Use
  it for comments, highlights, tags, or region annotations on images, web
  pages, PDFs, or IIIF canvases, even when the user just says "annotate a
  region" or "store a highlight". Not for the Annotation Protocol's HTTP API.
license: MIT
metadata:
  spec-source: https://www.w3.org/TR/annotation-model/
  version: "1.0"
---

# W3C Web Annotation Data Model

This skill covers the W3C Web Annotation Data Model (W3C Recommendation, 2017). An annotation connects a **body**, the content of the annotation, to one or more **targets**, the things being annotated.

## Minimal annotation

```json
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "id": "https://example.org/anno/1",
  "type": "Annotation",
  "motivation": "commenting",
  "creator": { "id": "https://example.org/user/42", "type": "Person", "name": "A. Person" },
  "created": "2026-10-31T12:00:00Z",
  "body": { "type": "TextualBody", "value": "Nice brushwork", "format": "text/plain", "language": "en" },
  "target": "https://example.org/page1"
}
```

| Property | Rule |
|---|---|
| `@context` | **MUST** include `http://www.w3.org/ns/anno.jsonld`. If it is the only context, give it as a string. |
| `id` | **MUST** be exactly one IRI |
| `type` | **MUST** include `Annotation` |
| `target` | **MUST** have one or more |
| `body` | **SHOULD** have one or more, **MAY** have zero |
| `bodyValue` | A plain-string body, read as `text/plain`. **MUST NOT** appear together with `body`. |
| `motivation` | SHOULD have exactly one (see the list below) |
| `creator`, `created`, `modified`, `generator`, `generated` | Provenance. Timestamps are `xsd:dateTime` in UTC with a `Z` suffix. |
| `rights`, `audience`, `accessibility`, `canonical`, `via` | Optional. `rights` is an IRI. |

With multiple bodies and targets, **every body relates to every target individually**.

## Bodies

- **TextualBody** (embedded): `value` is required, and `type` should be `TextualBody`. Optional fields are `format` (a media type, such as `text/html`), `language` (BCP 47), `textDirection` (`ltr`, `rtl`, or `auto`), and `purpose` (a motivation such as `tagging`).
- **External resource**: `{ "id": "https://…/image.png", "type": "Image", "format": "image/png" }`. The `type` should be one of `Dataset`, `Image`, `Video`, `Sound`, `Text`. A plain IRI string is also allowed.
- **Choice**: `{ "type": "Choice", "items": [ … ] }`. The first item is the default.

`Composite`, `List`, and `Independents` are in a non-normative appendix of the spec. Avoid them unless a consumer explicitly supports them.

## SpecificResource: targeting part of a resource

Use a SpecificResource whenever the annotation applies to a segment, state, or style of a resource rather than the whole resource:

```json
"target": {
  "type": "SpecificResource",
  "source": "https://example.org/image1",
  "selector": {
    "type": "FragmentSelector",
    "conformsTo": "http://www.w3.org/TR/media-frags/",
    "value": "xywh=100,200,300,400"
  }
}
```

- `source` **MUST** be exactly one IRI or resource.
- Optional properties: `selector`, `state`, `styleClass`, `purpose`, `renderedVia`, `scope`.
- If there are several selectors, they should select the same content, and the consumer picks one.
- **States are processed before selectors.**

## Selectors

| Type | Required properties | Use for |
|---|---|---|
| `FragmentSelector` | `value`; `conformsTo` SHOULD be present | Fragment syntax for a media type. Media fragments (`xywh=x,y,w,h`, `xywh=percent:…`, `t=start,end`) suit images, video, and IIIF canvases. |
| `SvgSelector` | `value` (well-formed SVG), or an `id` pointing to an SVG file | Non-rectangular shapes. Coordinates are relative to the source's dimensions. |
| `TextQuoteSelector` | `exact`; `prefix` and `suffix` SHOULD be present | Text found by quoting it. If the quote matches several places, it matches all of them. |
| `TextPositionSelector` | `start` (included), `end` (excluded) | Positions in the normalized text, counted in Unicode code points from 0. Breaks easily when the text changes. |
| `DataPositionSelector` | `start`, `end` | Byte ranges, counted from 0 |
| `CssSelector` | `value` | DOM elements matched by a CSS selector |
| `XPathSelector` | `value` | DOM elements matched by an XPath |
| `RangeSelector` | `startSelector` (included), `endSelector` (excluded) | A span between two selections. Both should be the same selector type. |

`refinedBy` nests a selector or state that is applied to the result of the outer one. For example, an XPath to a paragraph refined by a TextQuoteSelector inside it.

Common `conformsTo` values:

| Media | `conformsTo` | Example value |
|---|---|---|
| Image, video, audio | `http://www.w3.org/TR/media-frags/` | `xywh=50,50,640,480` |
| HTML | `http://tools.ietf.org/rfc/rfc3236` | `namedSection` |
| PDF | `http://tools.ietf.org/rfc/rfc3778` | `page=10&viewrect=50,50,640,480` |
| Plain text | `http://tools.ietf.org/rfc/rfc5147` | `char=0,10` |
| SVG | `http://www.w3.org/TR/SVG/` | `svgView(viewBox(50,50,640,480))` |

```json
{ "type": "TextQuoteSelector", "exact": "anotation", "prefix": "this is an ", "suffix": " that has some" }
{ "type": "SvgSelector", "value": "<svg xmlns=\"http://www.w3.org/2000/svg\"><polygon points=\"100,100 200,100 150,200\"/></svg>" }
```

## States and styles

- `TimeState`: use `sourceDate`, **or** both `sourceDateStart` and `sourceDateEnd`. Optionally add `cached` (the IRI of an archived copy). Use it to annotate the version of a page as it was at a given time.
- `HttpRequestState`: `value` holds the full set of HTTP request headers to send, as one string.
- Styling: the annotation's `stylesheet` (type `CssStylesheet`, inline `value` or an IRI), plus `styleClass` on a SpecificResource. Styles are hints that consumers may ignore.

## Motivations

The 13 W3C motivations are `assessing`, `bookmarking`, `classifying`, `commenting`, `describing`, `editing`, `highlighting`, `identifying`, `linking`, `moderating`, `questioning`, `replying`, and `tagging`. A body can carry its own `purpose`, using the same values. For example, a tag body inside a commenting annotation has `"purpose": "tagging"`.

## Collections

- **AnnotationCollection**: `id`, `type: "AnnotationCollection"`, `label` (a string), `total`, `first` (required when `total` > 0), and `last`.
- **AnnotationPage**: `id`, `type: "AnnotationPage"`, `items` (one or more annotations), `partOf`, `next` (required unless it is the last page), `prev`, and `startIndex` (counted from 0).
- Embedded pages **should not** repeat `@context`.

## Procedure: create an annotation

1. Choose the `motivation` from the list above.
2. Build the body: a TextualBody for text you write, or an IRI or object for an external resource.
3. Build the target: a plain IRI for a whole resource, or a SpecificResource with a selector for part of one. Prefer `FragmentSelector` with `xywh=` for rectangles on images, and `TextQuoteSelector` with `prefix` and `suffix` for text.
4. Add `id`, `@context`, `type`, `creator`, and `created`.
5. Validate:
   - There is at least one target.
   - `body` and `bodyValue` are not both present.
   - Every selector has its required properties.
   - Timestamps end in `Z`.
   - The JSON parses.

Serve annotations as `application/ld+json;profile="http://www.w3.org/ns/anno.jsonld"`.

## Gotchas

- **`painting` and `supplementing` are not W3C motivations.** They are defined by IIIF Presentation 3.0. Use them only inside IIIF manifests.
- **The context is `http://www.w3.org/ns/anno.jsonld`**, not the vocabulary namespace `http://www.w3.org/ns/oa#`. Use the `id` and `type` keys, not `@id` and `@type`.
- **`label` is a plain string here.** IIIF uses language maps for `label`, so the same key has a different shape in each spec.
- **`xywh=` values are pixels unless prefixed with `percent:`.**
- **Prefer a FragmentSelector over appending `#xywh=…` to the target IRI.** The spec recommends the explicit selector.
- **TextPositionSelector breaks when the document changes.** Pair it with a TextQuoteSelector as a second selector, or add a TimeState.
- **Do not put style, script, or animation inside SvgSelector SVG.** Consumers should ignore it.
