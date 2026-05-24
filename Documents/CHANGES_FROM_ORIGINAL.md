# Echo's Clarity TTW — Changes:

This document details every difference between this ESP and the version
this fork was built against — a 2020 build of Clarity TTW sourced from the
original TTW forums (now defunct), which was newer than david88m's 2018
Nexus release (Nexus ID 62481). These are the differences between this
version and david88m's current Nexus release:

---

## Master List Reordering

The master file load order was reorganized to place NV DLCs before FO3 DLCs.

**This version:**
```
00 FalloutNV.esm
01 DeadMoney.esm
02 HonestHearts.esm
03 OldWorldBlues.esm
04 LonesomeRoad.esm
05 Fallout3.esm
06 Anchorage.esm
07 ThePitt.esm
08 BrokenSteel.esm
09 PointLookout.esm
0A Zeta.esm
```

**Original TTW forum version:**
```
00 FalloutNV.esm
01 Fallout3.esm
02 Anchorage.esm
03 ThePitt.esm
04 BrokenSteel.esm
05 PointLookout.esm
06 DeadMoney.esm
07 HonestHearts.esm
08 OldWorldBlues.esm
09 LonesomeRoad.esm
0A Zeta.esm
```

Because all FormID references (image adapters, ambient sounds, image spaces)
are stored as raw bytes including the mod-index high byte, this reordering
caused a cascade of byte-level changes across IAD, SNAM, and WNAM subrecords
throughout the file. Those changes are not listed individually below since
they represent the same referenced records — only the FormID encoding changed.

---

## Bug Fixes

### 22 IMGS records — FO3 image spaces had vanilla data instead of Clarity values

The most significant fix. Twenty-two FO3 image space (IMGS) records contained
unmodified vanilla FO3 data instead of Clarity-processed values. All 22 had
their full DNAM block replaced with the correct Clarity values.

Root cause: the persistent gray tint overlay (tintA ~ 0.5) present in vanilla
FO3 image spaces was never cleared in these records. Clarity sets tintA = 0.0
across the board. These records left it active, causing every FO3 location in
TTW to render through a constant gray wash.

Additionally, the cinematic values (saturation, brightness, contrast, bloom)
in all 22 records were at vanilla FO3 defaults rather than Clarity values.
Example for DefaultImageSpaceExterior:

| Field        | Vanilla | Clarity |
|-------------|---------|---------|
| saturation   | 0.125   | 1.200   |
| brightness   | 1.200   | 0.900   |
| contrast     | 0.900   | 0.604   |
| tintA        | 0.510   | 0.000   |
| bloom.blurR  | 1.000   | 0.030   |
| bloom.alpha  | 0.030   | 0.800   |

The DNAM block was also padded from its original 132 or 148 bytes to the full
152-byte layout used by the rest of the file.

**Affected records:**
- DefaultImageSpaceExterior (covers all FO3 outdoor areas - most critical)
- WastelandImageSpace01
- UrbanImageSpace01
- UrbanImageSpace01intro
- gUrbanImageSpace01
- gWastelandImageSpace01
- gVaultImageSpace01
- MetroImageSpace01
- MetroDefaultImageSpace01
- ShackInterior01
- ShackInteriorWaste01
- CaveInteriorIM01
- CaveTestBrightImageSpace
- OfficeDefaultImageSpace
- NeoClassDefaultImageSpace
- RCDefaultImageSpace
- IndDefaultImageSpace
- GNRRoofImageSpace01
- LamplightInterior
- GrantTest
- BasementDefaultImageSpace (emissive: 2.0 -> 1.6)
- StatesmanHotelSpace (brightness: 1.0 -> 0.7)

### DLCPittSteelMillInterior — compressed flag removed

The compressed record flag (0x400) was cleared on this IMGS record.
The record is now stored uncompressed inline with the rest of the file.

---

## Night Image Space Modifier Corrections (21 IMAD records)

Twenty-one night-time IMAD (image adapter) records had color correction
keyframe values (TNAM) adjusted. The changes affect the RGB tint components
of the night-time color grade animation.

Most adjustments are exactly +/- 1/255 per channel — floating-point rounding
corrections from re-exporting color data at byte precision. Two records carry
larger corrections:

- One keyframe was darkened by approximately 33/255 per channel
- One keyframe was brightened by approximately 37/255 per channel

**Affected records:**
WastelandNorthDayISFX, WastelandNorthNightISFX, WastelandEastNightISFX,
SuburbanNightISFX, UrbanDeepNightISFX, UrbanDeepInnerNightISFX,
WastelandNightISFX, NVWastelandNightIS, DLC03CitPitNightISFX,
NVSewerNightLightISFX, NVUrbanNightIS, NVLegateCampNightIS,
NVLegateBattleNightIS, MegatonFalloutDecayNightISFX (also has FIAD/KIAD
data updated), DLC02AnchNightISFX, DLC04MarshInnerNightISFX,
NVDLC02TheNarrowsNightIS, NVDLC02ZionValleyNightIS, NVDLC04DivideNightIS,
NVDLC04NukeTransNightISFX

Additionally, WastelandDayISFX and WastelandSunriseISFX have IAD reference
updates consistent with the master reordering.

---

## DefaultWeather Lighting Correction

The DefaultWeather WTHR record (covers the main FO3 DC wasteland) had two
values in its INAM lighting data changed from 0.0 to 1.0.

---

## New Records

### Custom weather records for Hoover Dam and endgame (4 WTHR)

Four new weather records covering the Hoover Dam / endgame sequence, providing
Clarity-consistent lighting for areas that previously fell back to unmanaged
vanilla conditions:

- NVHooverDamWeather
- NVHooverBattleWeather
- NVHooverFinalBattle
- NVWastelandClearEast

### Inverted daylight weather variants (3 WTHR)

Three interior/inverted-lighting condition weather records:

- InvertedDaylightWeather
- InvertedDaylightWeatherWarm
- InvertedDaylightWeatherWarmNV

### Big MT crystal emitter weather (1 WTHR)

- NVDLC03crystalEmWeather

### Lonesome Road night image spaces (2 IMAD)

Two missing night-time image adapters for Lonesome Road zones:

- NVDLC04LRNightISFX
- NVDLC04HRNightISFX

### FO3 / DC-specific record variants (16 records)

Dedicated FO3/Washington DC variants of lighting templates and image spaces,
distinguished from their NV counterparts by a "DC" suffix. This separation
prevents FO3 and NV lighting setups from sharing records that have different
requirements in each game region.

LGTM (10):
- UtilityDefaultTemplateDC
- NTLGuardDepotTemplateDC
- OfficeDefaultTemplateDC
- CavesDefaultTemplateDC
- CavesDefaultTemplate02DC
- MetroDefaultTemplateDC
- ShackDefaultTemplateDC
- NeoclassicalDefaultTemplateDC
- UnderworldTemplateDC
- Vault92TemplateDC

IMGS (2):
- BasementDefaultImageSpaceDC
- StatesmanHotelSpaceDC

LGTM (2 more):
- StatesmanHotelTemplateDC

IMAD (2):
- WastelandDayISFXDC
- WastelandSunriseISFXDC
- MegatonFalloutDecayNightISFXDC

---

## LGTM Reference Updates (11 records)

Eleven FO3 lighting template records had internal FormID references updated to
point to the correct records under the new master order. No lighting values
were changed — only the referenced image space and related FormIDs were
remapped.

Affected: UtilityDefaultTemplate, NTLGuardDepotTemplate, StatesmanHotelTemplate,
OfficeDefaultTemplate, CavesDefaultTemplate, CavesDefaultTemplate02,
MetroDefaultTemplate, ShackDefaultTemplate, NeoclassicalDefaultTemplate,
UnderworldTemplate, Vault92Template

---

## Record Count Summary

| Type | This version | Original TTW version |
|------|-------------|---------------------|
| IMGS | 110         | 110                 |
| IMAD | 128         | 126                 |
| WTHR | 108         | 100                 |
| LGTM | 51          | 51                  |
| REGN | 3           | 3                   |
| LIGH | 1           | 1                   |

Net additions: +8 WTHR, +2 IMAD
