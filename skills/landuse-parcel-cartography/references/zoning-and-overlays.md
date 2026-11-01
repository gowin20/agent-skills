# Zoning, Future Land Use, and Overlay Symbology

Read this file when making a zoning map or a future land use map (FLUM), or when symbolizing overlay districts. The overlay table and the map-type distinctions come from GatherGov, ["How to Read and Interpret Zoning and Land Use Map"](https://gathergov.com/articles/zoning-map-guide) (April 2026). The color practice is consistent with the APA survey in [traditional-schemes.md](traditional-schemes.md).

## The three map types

| | Existing land use | Zoning | Future land use (FLUM) |
|---|---|---|---|
| Nature | Descriptive: "what *is*, not what's *allowed*" | Regulatory: a legal document, adopted in the land development code | Policy: adopted in the comprehensive plan, covering the next 10-20 years |
| Unit | Parcel | Zoning district, usually drawn along parcel lines | Generalized area, sometimes parcel based |
| Categories | Detailed actual uses | District codes (R-1, C-2, M-2, PUD) | Broad character and density ranges |
| Color | Category hue plus ramp, solid fills | Hue family by district type, a ramp within the family, a code label on every district | The same hue families, with density shown by ramp or by a dot or hatch overlay |

Keep the hue families the same on all three maps in a plan, so a reader can lay them side by side. A FLUM designation does not change what may be built today, so on maps that show both, make the FLUM visibly different: for example, a generalized fill with no parcel lines, or a pattern.

## Coloring zoning districts

There is no national standard for zoning colors. District codes and their meanings differ from place to place ("a 'C-2' in Phoenix means something entirely different from a 'C-2' in Philadelphia"). Apply the land use hue families by district *type*, and ramp by intensity within each type:

| District family (typical prefixes) | Hue family | Ramp, light to dark |
|---|---|---|
| Agricultural / rural (A, AG, RA, RU) | Olive or yellow-green | Agricultural → rural residential |
| Residential (R, RS, RM, RE) | Yellow to brown | R-E / R-1 (largest lot) → R-2 → R-3 → RM / R-4+ (highest density) |
| Office (O, OP, OI) | Light red, salmon, or pink | Office → office-institutional |
| Commercial / business (C, B, CB, CG) | Orange to red | C-1 neighborhood → C-2 community → C-3 general/highway → CBD / downtown |
| Mixed use (MU, MX, TC, DT) | Purple or red-violet, or stripes of the component colors | Neighborhood mixed use → urban core |
| Industrial / manufacturing (I, M, LI, HI) | Purple or gray | I-1 / M-1 light → I-2 / M-2 heavy |
| Public / institutional (P, PI, CF, GOV) | Blue | |
| Parks, open space, conservation (OS, P, CON, REC) | Green | Recreation → conservation |
| Planned development (PUD, PD, PDD) | Its own color with a heavy outline and the "PUD" label, or the color of its main use with a PUD outline | |

On every district, label the code. When two districts in the same family sit next to each other, the label is often the only thing that tells them apart.

## Overlay districts

Overlays add rules on top of the base zoning and often cross several base districts. **Draw them on top of the base fill, never instead of it.** Use hatching, outlines, or a transparent tint, so the base district still shows through.

| Overlay type | Typical map indicator |
|---|---|
| Historic preservation | Hatching, or a distinct border around the historic area |
| Environmental / conservation | Green or blue-green hatching, often following natural features |
| Flood zone (FEMA) | Blue shading or cross-hatching along waterways |
| Transit-oriented development (TOD) | A circle or radius boundary around transit stations |
| Affordable housing | Varies: a distinct color or a text label |
| Airport / runway clear zone | Concentric zones radiating from the runway |
| Corridor / highway | A linear overlay following the road |
| Community design | A distinct border around a defined neighborhood or commercial area |
| Planned unit development (PUD) | A distinct boundary with a PUD label |

How to draw them:

- Give each overlay **one technique**, chosen so that no two overlays look alike: a distinct hatch angle, a distinct line style (dashed, dotted, double), or a distinct outline color.
- Use **outline-only** symbols for large overlays (TOD radius, airport zones, corridors), so the base colors stay readable.
- Use **hatching** for overlays that fill areas (historic, flood, conservation). Draw the hatch in a hue that does not belong to any base category, or in dark gray.
- When there are more than 3-4 overlays, put them on separate inset maps or toggleable layers rather than stacking them all on one map.
- Some jurisdictions list overlays in a separate table rather than drawing them all. Either way, every overlay drawn must appear in the legend.

## Legend practice

- Group the legend by district family, in convention order (agricultural, residential, office, commercial, mixed use, industrial, public, open space, PUD), then a separate "Overlays" section.
- Show each district's code and its full name.
- Remind readers that colors are local: "Never assume you know what a color means without checking the legend."
