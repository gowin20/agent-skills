# APA LBCS Color Codes

The American Planning Association's **Land-Based Classification Standards (LBCS)** classify land use in five independent dimensions. Each dimension has nine top-level categories (codes 1000-9000), and each top-level category has one assigned color.

Source: APA, [LBCS standards](https://www.planning.org/lbcs/standards/), and the APA color spreadsheet `LBCSColorForAllDimensions.xls`, which is linked from that page. Every value below was checked against that spreadsheet. The same data is in [../assets/lbcs-colors.csv](../assets/lbcs-colors.csv).

## Rules from the standard

- "Within each dimension, the color value is constant for the top-level category and all its subcategories."
- Where one color for a whole category is not enough, apply "ramping": shade the category's color lighter or darker for its subcategories. APA's example: if residential activities is yellow, make all the residential subcategories shades of yellow.
- Specify RGB values, not color names. Use the hex value where the software accepts only hex.
- The color names are X11 color names (`forest green`, `bisque3`, `tan4`, and so on), so they also work as named colors in R, Tcl/Tk, and many plotting tools. CSS has no `bisque3`, `tan4`, `wheat4`, `brown4`, `blue4`, `purple4`, or `gray52`, and CSS `gray` is `#808080`, not `#BEBEBE`. Always use the hex.

## Choosing a dimension

| Dimension | Question it answers | Use it for |
|---|---|---|
| **Activity** | What people are actually doing on the land | Existing land use from field surveys |
| **Function** | The economic purpose of the establishment | Most parcel land use maps, and assessor and business data |
| **Structure** | The type of building | Building inventories |
| **Site development character** | The physical state of the site | Development status maps |
| **Ownership** | The legal constraints on use | Public and private ownership maps |

For a general parcel land use map, use **Function**, or **Activity** if the data comes from a field survey.

## Activity

| Code | Category | Color name | RGB | Hex |
|---|---|---|---|---|
| 1000 | Residential activities | yellow | 255, 255, 0 | `#FFFF00` |
| 2000 | Shopping, business, or trade activities | red | 255, 0, 0 | `#FF0000` |
| 3000 | Industrial, manufacturing, and waste-related activities | purple | 160, 32, 240 | `#A020F0` |
| 4000 | Social, institutional, or infrastructure-related activities | blue | 0, 0, 255 | `#0000FF` |
| 5000 | Travel or movement activities | gray | 190, 190, 190 | `#BEBEBE` |
| 6000 | Mass assembly of people | dark slate gray | 47, 79, 79 | `#2F4F4F` |
| 7000 | Leisure activities | light green | 144, 238, 144 | `#90EE90` |
| 8000 | Natural resources-related activities | forest green | 34, 139, 34 | `#228B22` |
| 9000 | No human activity or unclassifiable activity | white | 255, 255, 255 | `#FFFFFF` |

## Function

| Code | Category | Color name | RGB | Hex |
|---|---|---|---|---|
| 1000 | Residence or accommodation functions | yellow | 255, 255, 0 | `#FFFF00` |
| 2000 | General sales or services | red | 255, 0, 0 | `#FF0000` |
| 3000 | Manufacturing and wholesale trade | purple | 160, 32, 240 | `#A020F0` |
| 4000 | Transportation, communication, information, and utilities | gray | 190, 190, 190 | `#BEBEBE` |
| 5000 | Arts, entertainment, and recreation | light green | 144, 238, 144 | `#90EE90` |
| 6000 | Education, public admin., health care, and other inst. | blue | 0, 0, 255 | `#0000FF` |
| 7000 | Construction-related businesses | dark cyan | 0, 139, 139 | `#008B8B` |
| 8000 | Mining and extraction establishments | purple4 | 85, 26, 139 | `#551A8B` |
| 9000 | Agriculture, forestry, fishing and hunting | forest green | 34, 139, 34 | `#228B22` |

## Structure

| Code | Category | Color name | RGB | Hex |
|---|---|---|---|---|
| 1000 | Residential buildings | yellow | 255, 255, 0 | `#FFFF00` |
| 2000 | Commercial buildings and other specialized structures | red | 255, 0, 0 | `#FF0000` |
| 3000 | Public assembly structures | purple | 160, 32, 240 | `#A020F0` |
| 4000 | Institutional or community facilities | blue | 0, 0, 255 | `#0000FF` |
| 5000 | Transportation-related facilities | gray | 190, 190, 190 | `#BEBEBE` |
| 6000 | Utility and other nonbuilding structures | gray52 | 133, 133, 133 | `#858585` |
| 7000 | Specialized military structures | pink | 255, 192, 203 | `#FFC0CB` |
| 8000 | Sheds, farm buildings, or agricultural facilities | forest green | 34, 139, 34 | `#228B22` |
| 9000 | No structure | white | 255, 255, 255 | `#FFFFFF` |

Structure 3000 (public assembly) is purple, the same hex as industrial in Activity and Function. This is one reason you must not mix dimensions on one map.

## Site development character

| Code | Category | Color name | RGB | Hex |
|---|---|---|---|---|
| 1000 | Site in natural state | light green | 144, 238, 144 | `#90EE90` |
| 2000 | Developing site | beige | 245, 245, 220 | `#F5F5DC` |
| 3000 | Developed site: crops, grazing, forestry, etc. | bisque3 | 205, 183, 158 | `#CDB79E` |
| 4000 | Developed site: no buildings and no structures | wheat4 | 139, 126, 102 | `#8B7E66` |
| 5000 | Developed site: nonbuilding structures | tan4 | 139, 90, 43 | `#8B5A2B` |
| 6000 | Developed site: with buildings | brown4 | 139, 35, 35 | `#8B2323` |
| 7000 | Developed site: with parks | forest green | 34, 139, 34 | `#228B22` |
| 8000 | Not applicable to this dimension | light gray | 211, 211, 211 | `#D3D3D3` |
| 9000 | Unclassifiable site development character | white | 255, 255, 255 | `#FFFFFF` |

## Ownership

| Code | Category | Color name | RGB | Hex |
|---|---|---|---|---|
| 1000 | No constraints: private ownership | beige | 245, 245, 220 | `#F5F5DC` |
| 2000 | Some constraints: easements or other use restrictions | blue | 0, 0, 255 | `#0000FF` |
| 3000 | Limited restrictions: leased and other tenancy restrictions | blue4 | 0, 0, 139 | `#00008B` |
| 4000 | Public restrictions: local, state, and federal ownership | light green | 144, 238, 144 | `#90EE90` |
| 5000 | Other public use restrictions: regional, special districts, etc. | dark green | 0, 100, 0 | `#006400` |
| 6000 | Nonprofit ownership restrictions | olive drab | 107, 142, 35 | `#6B8E23` |
| 7000 | Joint ownership character: public entities | gray | 190, 190, 190 | `#BEBEBE` |
| 8000 | Joint ownership character: public, private, nonprofit, etc. | black | 0, 0, 0 | `#000000` |
| 9000 | Not applicable to this dimension | white | 255, 255, 255 | `#FFFFFF` |

## Using LBCS colors on a finished map

The LBCS colors are fully saturated. They work well as codes and for small features. Filled across large parcels, they glare. To keep LBCS compliance and still make a readable map:

1. Store the official hex in the data or style as the category's color of record.
2. Draw large fills as a tint of that hex. Mixing about 40-60% toward white keeps the hue, for example `#FFFF00` → `#FFFF99`. Or draw the official color at reduced fill opacity over a white background.
3. Ramp subcategories within the hue, as the standard allows, and record the ramp in the legend.
4. Say in the legend or metadata that the colors are LBCS tints, so nobody mistakes them for the official values.
