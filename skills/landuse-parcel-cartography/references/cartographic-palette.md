# Cartographic Land Use Palette

This is a ready-to-use screen palette for parcel land use maps, with 42 subcategories in the conventional hue families. Load it from [../assets/landuse-palette.csv](../assets/landuse-palette.csv) when writing a renderer or style file. This table is the same data in readable form.

**Status:** this palette is *derived*, not an official standard. It turns the traditional pencil schemes ([traditional-schemes.md](traditional-schemes.md)) and the LBCS hue anchors ([lbcs-colors.md](lbcs-colors.md)) into tints that work well as large polygon fills. Where a client has an adopted palette or requires official LBCS values, use theirs.

## How it was built

- **Hue families** follow the APA survey: residential runs yellow → brown, commercial pink → red → crimson, industrial lavender → purple, institutional light → deep blue, open space light → dark green, agriculture olive, transportation and utilities gray.
- **Lightness falls steadily with intensity** within each family. For example, residential steps from CIELAB L* ≈ 99 (rural) to 43 (high-rise), and commercial from 80 (local) to 30 (CBD).
- **Large-area categories are tints.** Single-family, agriculture, and parks avoid the glaring saturation of the LBCS anchors.
- **Colorblind check:** the pairs most likely to be confused were tested by simulating deuteranopia and protanopia (Machado et al. 2009) and measuring CIE76 ΔE: commercial vs. parks, industrial vs. institutional at matching ramp steps, water vs. light institutional blues, and mixed use vs. industrial. Every tested pair stayed at ΔE ≥ 12 under both simulations. The closest pair is mixed use vs. general industrial, which is why mixed use should carry stripes.
- **Vacant** is white. **Road right-of-way** is very light gray, so the two stay distinct. **Unclassified** is a neutral gray with a hatch.

If you change a color, keep its lightness between those of its neighbors in the ramp, and rerun a colorblind preview.

## Palette

### Residential

| ID | Category | Hex | RGB | Pattern | Basis |
|---|---|---|---|---|---|
| `RES-RUR` | Rural / estate residential | `#FFFFBE` | 255, 255, 190 | - | Pale yellow; very light single-family yellow |
| `RES-SF` | Single-family detached | `#FFF05A` | 255, 240, 90 | - | 915 Lemon Yellow; LBCS 1000 tint |
| `RES-2F` | Two-family / duplex / townhouse | `#F5D547` | 245, 213, 71 | - | 942 Yellow Ochre; APA 'dark yellow' |
| `RES-MF-LOW` | Multifamily, low-rise (3-4 units, garden apts) | `#E6A532` | 230, 165, 50 | - | 917 Yellow Orange / 918 Orange; NC multifamily orange |
| `RES-MF-MED` | Multifamily, mid-rise | `#C27A2C` | 194, 122, 44 | - | 943 Burnt Ochre |
| `RES-MF-HIGH` | Multifamily, high-rise | `#8C5A2B` | 140, 90, 43 | - | 946 Dark Brown; APA multifamily 'dark brown' |
| `RES-MH` | Mobile / manufactured home park | `#E8E09A` | 232, 224, 154 | dots | Yellow-family tint + pattern (traditional lavender clashes with industrial) |
| `RES-GQ` | Group quarters / dormitories | `#F0B878` | 240, 184, 120 | - | 918 Orange (De Chiara 2) |

### Commercial

| ID | Category | Hex | RGB | Pattern | Basis |
|---|---|---|---|---|---|
| `COM-LOC` | Neighborhood / local commercial | `#FFB3B3` | 255, 179, 179 | - | 929 Pink; APA 'light red' |
| `COM-OFF` | Office / professional | `#F28A7A` | 242, 138, 122 | - | 921 Vermilion Red / 928 Blush |
| `COM-GEN` | General / community commercial | `#E8493A` | 232, 73, 58 | - | 923 Scarlet Lake / 922 Scarlet Red; LBCS 2000 hue |
| `COM-REG` | Regional commercial / shopping center | `#C21F1F` | 194, 31, 31 | - | 925 Crimson Lake |
| `COM-CBD` | Central business district / intensive | `#8E0F14` | 142, 15, 20 | - | APA: 'the denser the commercial, the redder the color' |
| `COM-HWY` | Highway / auto-oriented commercial | `#F5A06E` | 245, 160, 110 | - | Orange-red; APA 'light red and orange' |
| `COM-TOUR` | Lodging / tourist commercial | `#F49AC1` | 244, 154, 193 | - | Pink for tourist commercial (APA survey) |

### Mixed use

| ID | Category | Hex | RGB | Pattern | Basis |
|---|---|---|---|---|---|
| `MIX-RC` | Mixed use, residential + commercial | `#C98BC2` | 201, 139, 194 | stripes of RES-SF and COM-GEN | Purple shades (Aurora, Denver); or striped |
| `MIX-OI` | Mixed use, office + light industrial / flex | `#B8B4C8` | 184, 180, 200 | - | Gray shades (Aurora) |

### Industrial

| ID | Category | Hex | RGB | Pattern | Basis |
|---|---|---|---|---|---|
| `IND-LT` | Light industrial | `#EEDDF2` | 238, 221, 242 | - | Light purple / 964 Light Gray |
| `IND-WH` | Warehouse / distribution / wholesale | `#D2B2E0` | 210, 178, 224 | - | 936 Slate Gray / 930 Magenta (NC wholesale) |
| `IND-GEN` | General industrial / manufacturing | `#B07CC6` | 176, 124, 198 | - | 931 Purple; LBCS 3000 hue |
| `IND-HVY` | Heavy industrial | `#5B3A8A` | 91, 58, 138 | - | Darker = heavier; 935 Black (De Chiara) |
| `IND-EXT` | Extractive / mining | `#7A6A72` | 122, 106, 114 | - | Dark gray-purple; LBCS Function 8000 purple4 |

### Institutional

| ID | Category | Hex | RGB | Pattern | Basis |
|---|---|---|---|---|---|
| `INS-GOV` | Government / civic | `#2A55B0` | 42, 85, 176 | - | 902 Ultramarine; LBCS 6000 hue |
| `INS-EDU` | Schools / education | `#4A7BD4` | 74, 123, 212 | - | 919 Sky Blue / 903 True Blue |
| `INS-MED` | Health care / hospitals | `#7EA2E0` | 126, 162, 224 | - | Blue (some schemes use greens) |
| `INS-REL` | Religious / quasi-public | `#A9C1EC` | 169, 193, 236 | - | 901 Indigo Blue (De Chiara churches), lightened |
| `INS-CEM` | Cemeteries | `#7FC2B8` | 127, 194, 184 | - | 905 Aquamarine |
| `INS-MIL` | Military / federal reservations | `#FFC0CB` | 255, 192, 203 | - | Pink for public lands; LBCS Structure 7000 |

### Parks and open space

| ID | Category | Hex | RGB | Pattern | Basis |
|---|---|---|---|---|---|
| `OS-REC` | Active recreation / sports | `#B8E6A0` | 184, 230, 160 | - | Light green; LBCS 7000/5000 hue |
| `OS-PARK` | Parks | `#7CC66B` | 124, 198, 107 | - | 910 True Green |
| `OS-PRIV` | Private recreation / golf | `#A9D08E` | 169, 208, 142 | - | Green tint |
| `OS-CON` | Conservation / preserve / forest | `#2E7D32` | 46, 125, 50 | - | Darker greens for environmental areas |

### Agriculture

| ID | Category | Hex | RGB | Pattern | Basis |
|---|---|---|---|---|---|
| `AG-CROP` | Agriculture, cropland | `#D2DE94` | 210, 222, 148 | - | 912 Apple Green, lightened |
| `AG-PAST` | Agriculture, pasture / livestock | `#A8B964` | 168, 185, 100 | - | 911 Olive Green |
| `AG-FOR` | Agriculture, timber / silviculture | `#6B8E3A` | 107, 142, 58 | - | Olive drab |

### Transportation and utilities

| ID | Category | Hex | RGB | Pattern | Basis |
|---|---|---|---|---|---|
| `TRN-ROW` | Road right-of-way | `#F2F2F2` | 242, 242, 242 | - | White or very light gray |
| `TRN-RAIL` | Railroad | `#B4B4B4` | 180, 180, 180 | - | 962 / 964 Grey |
| `TRN-AIR` | Airport / transportation facility | `#9E9E9E` | 158, 158, 158 | - | LBCS 4000 gray |
| `UTL` | Utilities / communications / waste | `#6E6E6E` | 110, 110, 110 | - | Darker gray; black preferred in traditional schemes |

### Other

| ID | Category | Hex | RGB | Pattern | Basis |
|---|---|---|---|---|---|
| `VAC` | Vacant / undeveloped | `#FFFFFF` | 255, 255, 255 | optional tint of intended category | 'No color' (De Chiara) |
| `WATER` | Water | `#A6E6EE` | 166, 230, 238 | - | Light cyan; keep distinct from institutional blue |
| `UNK` | Unclassified / no data | `#E0E0E0` | 224, 224, 224 | diagonal hatch | Neutral gray, distinct from vacant |

## Short palette (9 classes)

Use this for small-scale maps, general audiences, or a top-level legend.

| Category | Hex | From |
|---|---|---|
| Residential | `#FFF05A` | `RES-SF` |
| Commercial | `#E8493A` | `COM-GEN` |
| Mixed use | `#C98BC2` + stripes | `MIX-RC` |
| Industrial | `#B07CC6` | `IND-GEN` |
| Institutional / public | `#4A7BD4` | `INS-EDU` |
| Parks and open space | `#7CC66B` | `OS-PARK` |
| Agriculture | `#A8B964` | `AG-PAST` |
| Transportation and utilities | `#9E9E9E` | `TRN-AIR` |
| Vacant | `#FFFFFF` | `VAC` |
