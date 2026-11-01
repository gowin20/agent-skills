---
name: landuse-parcel-cartography
description: >
  Use this skill when choosing colors, fills, patterns, or a legend for a map of
  land use parcels, zoning districts, or future land use (FLUM, comprehensive
  plan) categories. It covers the traditional planning color convention
  (yellow residential, red commercial, purple or gray industrial, blue
  institutional, green open space), APA LBCS RGB color codes, light-to-dark
  ramps for density and intensity, hatching for mixed use, overlays, and
  black-and-white output, colorblind-safe adjustments, and lookup tables from
  land use categories to hex colors. Use it when styling a parcel layer in
  QGIS, ArcGIS, Mapbox, Leaflet, or D3, when writing a renderer or style file
  for assessor land use codes, or when reviewing a planning map's symbology,
  even if the user only says "color these parcels" or "make a zoning map".
  Not for land cover rasters (NLCD, Anderson) or for general charts.
license: MIT
metadata:
  title: Land Use Parcel Cartography - Symbology and Color Conventions
  sources: APA "Traditional Color Coding for Land Uses" (Jeer and Bain, 1997); APA LBCS color standards; Cyburbia forum thread on land use code colors (2002); GatherGov zoning map guide (2026)
  version: "1.0"
---

# Land Use Parcel Cartography: Symbology and Color

Planners have colored land use maps with one convention since the 1950s. Readers expect it, so a parcel map that follows it can be read at a glance and compared with maps from other places. This skill tells you how to apply that convention to parcel, zoning, and future land use data. It also covers when to depart from it.

## The core convention

The convention gives each major land use category a **hue family**. Within a family, you vary **lightness** to show subtypes, density, or intensity.

| Category | Hue family | Light end of the ramp | Dark end of the ramp | LBCS anchor (Function) |
|---|---|---|---|---|
| Residential | Yellow to brown | Rural or single-family (pale yellow) | High-rise multifamily (dark brown) | `#FFFF00` yellow |
| Commercial / retail / office | Orange to red | Local or neighborhood (light red, orange, pink) | Regional, CBD, intensive (deep red) | `#FF0000` red |
| Industrial | Purple or gray | Light industrial, flex (lavender, light gray) | Heavy industry (dark purple, dark gray, black) | `#A020F0` purple |
| Institutional / public / civic | Blue | Quasi-public, churches (light blue) | Government, major institutions (deep blue) | `#0000FF` blue |
| Parks, recreation, open space | Green | Recreation (light green) | Conservation, forest (dark green) | `#90EE90` light green |
| Agriculture | Yellow-green or olive | Crops (apple green) | Livestock, forestry (olive) | `#228B22` forest green |
| Transportation, utilities, ROW | Gray or black | Rights-of-way (white or light gray) | Utilities, rail (dark gray) | `#BEBEBE` gray |
| Mixed use | Purple or stripes of the component colors | | | none |
| Public lands, military | Pink | | | `#FFC0CB` (LBCS Structure 7000) |
| Vacant / unclassified | White or no fill | | | `#FFFFFF` |
| Water | Light blue or cyan | | | none |

The sources disagree in places. Industrial is purple in APA's summary and LBCS, gray in De Chiara, and blue in some Denver and North Carolina schemes. Multifamily is dark brown in De Chiara and orange in the North Carolina scheme. Pick one choice and use it throughout the map.

## Default procedure

1. **Identify the map type.** It sets the categories and how literal the colors should be. See [Map types](#existing-land-use-zoning-and-future-land-use).
2. **Crosswalk the source codes to the major categories.** Parcel data usually carries local assessor or zoning codes, not standard categories. Build an explicit lookup from each code to a top-level category, then to a subcategory. Collect codes you cannot classify in an "Other / unclassified" bucket, and never let them fall to the software's default color.
3. **Choose a palette depth.** Use 6-9 categories for a general audience or a small scale. Use 15-30 subcategories for a planning-staff map at parcel scale. Most readers cannot tell apart more than about 12 solid fills, so beyond that, use ramps within families and patterns.
4. **Assign colors from the tables.** Start from [assets/landuse-palette.csv](assets/landuse-palette.csv) for a ready-to-use cartographic palette. Use [assets/lbcs-colors.csv](assets/lbcs-colors.csv) when the data is coded in LBCS or the client requires APA codes.
5. **Ramp within each family.** Order subcategories from least to most intense and step the lightness evenly. See [Ramping](#ramping-for-density-and-intensity).
6. **Add patterns** for mixed use, overlays, proposed or future status, and black-and-white output. See [Patterns](#patterns-and-hatching).
7. **Style parcel outlines and labels.** See [Outlines and labels](#outlines-labels-and-legend).
8. **Check the result.** Run the [validation checklist](#validation-checklist), including a colorblind simulation and a grayscale print preview.

## Coloring techniques

### Qualitative hue by category

Use one hue family per major category. This is the basic technique, and it is the reason the convention exists: a reader who has seen one planning map can read the next one. Do not use a GIS program's default "random" or "unique values" palette. APA traced a whole family of nonstandard land use maps back to a sample template shipped with Arc/Info, which non-planners copied.

### Ramping for density and intensity

LBCS calls this "ramping": shading a hue from light to dark to show increasing density or intensity within one category.

- **Residential:** pale yellow (rural, estate) → yellow (single-family) → dark yellow or ochre (duplex, townhouse) → orange or burnt ochre (low-rise multifamily) → dark brown (high-rise multifamily).
- **Commercial:** pink or light orange (local, neighborhood) → light red (office, community) → red (general) → crimson or dark red (regional, CBD, intensive). "The denser the commercial, the redder the color."
- **Industrial:** lavender or light gray (light, flex) → purple or medium gray (general) → dark purple, dark gray, or black (heavy). "The heavier the industry, the darker the gray."
- **Open space:** light green (active recreation) → green (parks) → dark green (conservation, forest, preserves).

Keep each step visibly different in lightness, not only in hue. Most land use maps have 3-5 steps per family. Darker should always mean more intense, never the reverse.

### Tints for large areas

The LBCS anchors are pure, fully saturated colors (`#FFFF00`, `#FF0000`, `#0000FF`). They work as data codes, but filling large parcels with them overwhelms the map and hides the base layers. When you fill polygons:

- Use **tints** (lighter, less saturated versions) for categories that cover large areas: single-family residential, agriculture, and open space. Some single-family yellows are so pale they look almost white.
- Save **saturated, dark colors** for categories that cover small areas: commercial, industrial, and institutional. One Melbourne scheme drew residential in pink rather than red for this reason.
- Keep the **hue** of the anchor so the convention still reads.

### Patterns and hatching

Patterns carry a second variable on top of color, or replace color when the map is printed in black and white.

| Situation | Technique |
|---|---|
| Mixed use (residential + commercial) | Diagonal stripes in the two component colors, or a solid purple or red-violet. Office plus light industrial mixes are traditionally gray. |
| Overlay districts | Hatching or a distinct outline drawn over the base zoning color. Never replace the base color. See [references/zoning-and-overlays.md](references/zoning-and-overlays.md). |
| Future or proposed use | The color of the intended use, with a dot or hatch overlay, or alternating stripes with white |
| Density on future land use maps | A dot or hatch overlay of increasing weight on the category color |
| Hazardous or nuisance subtypes | The parent color with a dot pattern, for example brown with red dots for explosives |
| Black-and-white output | Traditional hatching: cross-hatch for residential, slanted lines for commercial, points or symbols for industrial, vertical lines for public, stipple for parks, curved lines for agriculture. Heavier pattern = denser use. |

Draw hatch lines in a darker shade of the base hue, or in a neutral dark gray, at 30-60% opacity, so the base color still shows. Keep the line spacing coarse enough to survive the output scale: about 1.5-3 mm on paper.

### Vacant, unknown, and no-data parcels

- **Vacant land:** no color, or white. On assessor-based maps, where vacant parcels carry an intended category (vacant residential, vacant commercial), use a very light tint of that category's hue, or white with a hatch in the category color.
- **Unclassified or no data:** a neutral light gray with a distinct pattern, labeled in the legend. Keep it different from vacant.

## Existing land use, zoning, and future land use

| Map type | What it shows | Color approach |
|---|---|---|
| **Existing land use** | What is on each parcel now. Descriptive, not regulatory. | Solid fills by actual use, at full subcategory detail. Parcel-level data, often from assessor codes. |
| **Zoning** | Which uses the law allows. Regulatory. | One color per district. Districts follow the hue families by prefix: R → yellow and brown, C or B → red, O → light red or pink, I or M → purple or gray, A or AG → olive or light green, P or OS → green, PUD → outline plus label, MU → stripes or purple. Label each district with its code. |
| **Future land use (FLUM)** | The planned character and density over 10-20 years | Broader categories in the same hue families. Show density ranges with ramps or dot or hatch overlays. Often drawn as generalized areas rather than parcels. |

Use the same hue families on all three maps in a set, so readers can compare them. Do not reuse the same colors to mean different things on maps in one plan. District codes are not portable: "C-2" in one city is not "C-2" in another, so always give the legend.

## Outlines, labels, and legend

- **Parcel boundaries:** use a thin neutral line (0.1-0.3 pt, mid to light gray) or a slightly darker shade of each fill. Heavy black parcel lines hide the fills at small scales. Below about 1:10,000, dissolve parcels by category, or drop the boundaries.
- **District and category boundaries:** draw a heavier line than parcel lines wherever adjacent parcels change category or district.
- **Labels:** put zoning or land use codes on polygons that are large enough to hold them. Codes give a second channel that does not depend on color.
- **Legend:** list the categories in convention order (residential, commercial, mixed use, industrial, institutional, parks and open space, agriculture, transportation and utilities, vacant, water). Ramp each family from light to dark. Show patterns as swatches. Never rely on the reader knowing the convention without a legend.

## Accessibility

- **Red vs. green** (commercial vs. parks) and **purple vs. blue** (industrial vs. institutional) are the pairs that blur most under red-green color vision deficiency. Separate them by lightness as well as hue. For example, use a light green for parks against a mid or dark red for commercial, and a light lavender for industrial against a mid blue for institutional.
- Keep **water** a light cyan that is clearly different from institutional blue.
- Add codes or labels, and use patterns for any pair that still blurs after you adjust lightness.
- Preview with a colorblind simulator (QGIS map view preview modes, ArcGIS Pro's color vision deficiency simulation, or Color Oracle). Preview in grayscale too.

## Gotchas

- **Specify RGB or hex, never color names.** LBCS recommends RGB values because names render differently in different software. Old pencil schemes use Prismacolor numbers, and the pencil names have changed over the years (922 "Scarlet Red" is now sold as Poppy Red). The number is the stable key. See [references/traditional-schemes.md](references/traditional-schemes.md).
- **LBCS has five dimensions, each with its own colors.** The same hex value means different things in different dimensions (`#0000FF` is institutional in Activity and Function, but "some constraints" in Ownership). Confirm which dimension a parcel's code belongs to before coloring it.
- **LBCS fixes colors only at the top level.** All subcategories inherit the top-level color unless you ramp them. Do not invent subcategory hex values and call them LBCS.
- **Classify by function, not ownership,** unless the map is about ownership. A city maintenance garage is functionally industrial. Ownership-based schemes would color it institutional.
- **Mobile and manufactured home parks** are traditionally lavender or purple (Denver, De Chiara). That clashes with industrial purple on modern maps. Prefer a distinct yellow-family tint with a pattern, and say so in the legend.
- **Hotels and trailer courts** are purple or magenta in De Chiara's scheme. Most modern maps treat lodging as commercial (red family).
- **Do not borrow land cover palettes** such as NLCD or Anderson for parcel land use. They use different categories and different color logic.
- **Convention is the default, not a law.** Practitioners disagree, and agencies often adapt it. If the client already has an adopted palette, match it exactly. Departures should serve the reader: legibility, print, color vision, a local standard. Do not depart from the convention for decoration.

## Validation checklist

- [ ] Every source code maps to a category. Nothing is drawn in the software's default color.
- [ ] Each major category uses its conventional hue family, or the departure is deliberate and documented.
- [ ] Within each family, lightness increases steadily with intensity. Darker means more intense.
- [ ] Large-area categories use tints, and saturated colors are saved for small areas.
- [ ] Water, institutional blue, and parks green are clearly different from each other.
- [ ] The map passes a colorblind preview. Any pair that still blurs has labels or patterns.
- [ ] The map reads in a grayscale preview, or a black-and-white hatching version exists.
- [ ] The legend lists every category in convention order, with pattern swatches.
- [ ] Colors are stored as hex or RGB in the style file, not as names.

## Resources

- [assets/landuse-palette.csv](assets/landuse-palette.csv): a ready-to-use cartographic palette, 42 parcel land use subcategories with hex and RGB, ramped within the conventional hue families and checked for colorblind legibility. Load it when building a renderer or style file.
- [references/cartographic-palette.md](references/cartographic-palette.md): the same palette as readable tables, plus a 9-class short palette and notes on how it was built. Read it when you need to choose, explain, or adjust individual colors.
- [assets/lbcs-colors.csv](assets/lbcs-colors.csv): all 45 official APA LBCS top-level colors across five dimensions, machine readable.
- [references/lbcs-colors.md](references/lbcs-colors.md): read when the data uses LBCS codes, or when the user asks for "the APA standard colors".
- [references/traditional-schemes.md](references/traditional-schemes.md): read when matching a historical or textbook scheme, reproducing a pencil-colored map, or when you need the subcategory assignments of De Chiara, the North Carolina 1958 standard, or Denver 1965, or the standard black-and-white hatching patterns.
- [references/zoning-and-overlays.md](references/zoning-and-overlays.md): read when making a zoning or future land use map, coloring district codes, or symbolizing overlay districts (historic, flood, TOD, airport, and others).
