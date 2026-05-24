# Clarity TTW - Fork Report

Hi phoenix0113. I'm a TTW player and modder who found a significant bug in the TTW port
of Clarity that's been hosted on your mod page (uploaded by david88m). I've fixed it,
documented everything and forked the file. I wanted to give you a full technical
account of what was wrong and what changed, and let you know the fork exists in case
you want to fold any of this back in, take it down, host it, 
or if you'd rather handle it yourself.

The fork is here: https://github.com/Echo-Storm/Clarity_TTW_Fork

All comparisons in this report were done against your original Clarity.esp files (both
the FO3 and NV versions) as the source of truth. Those originals are intact and correct.
The bug was introduced in the TTW port.

---

## The Bug

The TTW version had 22 FO3 image space (IMGS) records carrying raw vanilla FO3 data
instead of your Clarity-processed values. Everything on the New Vegas side was ported
correctly. The bug only affected FO3 content.

Vanilla FO3 image spaces carry a permanent gray tint overlay (tintA around 0.5). Your
mod disables this by setting tintA to 0.0. Those 22 records still had tintA active in
the TTW version, so every FO3 location in TTW was rendering through a constant gray wash.

The cinematic values were also wrong across all 22. Using DefaultImageSpaceExterior as
an example - this is the most critical record in the mod, covering all FO3 outdoor areas:

| Field        | Vanilla FO3 | Your Clarity values |
|-------------|-------------|---------------------|
| saturation   | 0.125       | 1.200               |
| brightness   | 1.200       | 0.900               |
| contrast     | 0.900       | 0.604               |
| tintA        | 0.510       | 0.000               |
| bloom.blurR  | 1.000       | 0.030               |
| bloom.alpha  | 0.030       | 0.800               |

The Citadel / time-of-day-goes-black bug that TTW users have reported was almost
certainly DefaultImageSpaceExterior specifically. The most critical record in the mod,
covering all FO3 outdoor areas, was running vanilla values the whole time.

All 22 broken records were verified against both your FO3 Clarity.esp and NV Clarity.esp.
The values in both originals were identical for every shared record, so there was no
ambiguity about what the correct values should be.

**Affected records:**
- DefaultImageSpaceExterior (all FO3 outdoor areas - most critical)
- WastelandImageSpace01
- UrbanImageSpace01 / UrbanImageSpace01intro
- gUrbanImageSpace01 / gWastelandImageSpace01 / gVaultImageSpace01
- MetroImageSpace01 / MetroDefaultImageSpace01
- ShackInterior01 / ShackInteriorWaste01
- CaveInteriorIM01 / CaveTestBrightImageSpace
- OfficeDefaultImageSpace / NeoClassDefaultImageSpace
- RCDefaultImageSpace / IndDefaultImageSpace
- GNRRoofImageSpace01 / LamplightInterior / GrantTest
- BasementDefaultImageSpace (one field off - emissive 2.0 vs your 1.6)
- StatesmanHotelSpace (one field off - brightness 1.0 vs your 0.7)

For each of the 22 records, the full 152-byte DNAM block was replaced with your
Clarity-processed values. The DNAM blocks in the broken records were also undersized
(132 or 148 bytes depending on the record) and were padded to the full 152-byte layout
used by the rest of the file.

### DLCPittSteelMillInterior - compressed flag removed

The compressed record flag was cleared on this image space record. It is now stored
uncompressed inline with the rest of the file.

---

## What Was Confirmed Correct in the TTW Port

Before making any changes a full audit was run. These record types were all intact and
correct relative to your originals:

- All 65 NV Clarity IMGS records: values match your originals exactly
- All 128 IMAD records: correctly ported
- All 108 WTHR records: intact, including the Hoover Dam / Fort / Colorado River
  custom weathers and the Searchlight entries
- All 3 REGN records: intact
- All 51 LGTM records: intact
- HEDR version: 1.34 (correct for TTW)

Three records were flagged during an HDR audit but confirmed intentional from your
originals and left unchanged:

- UltraLuxeKitchen: eyeAdaptSpeed = 8.0 (NV Clarity only)
- NVDLC04NukeIS: contrast = 5.0 (NV Clarity only - nuke flash effect)
- DLC05MZImageSpaceTheFinalFrontier: brightClamp = 0.01 (FO3 Clarity only - space)

---

## Other Changes in the Fork

Beyond the bug fix, the fork carries forward several additions that were present in a
2020 build found on the original TTW forums (now defunct). These were not in david88m's
file on your Nexus page.

### Master list reordering

The master file load order was reorganized to place NV DLCs before FO3 DLCs. This is
the standard ordering for TTW plugins built against a modern TTW install:

| Slot | This fork         | david88m's file   |
|------|-------------------|-------------------|
| 00   | FalloutNV.esm     | FalloutNV.esm     |
| 01   | DeadMoney.esm     | Fallout3.esm      |
| 02   | HonestHearts.esm  | Anchorage.esm     |
| 03   | OldWorldBlues.esm | ThePitt.esm       |
| 04   | LonesomeRoad.esm  | BrokenSteel.esm   |
| 05   | Fallout3.esm      | PointLookout.esm  |
| 06   | Anchorage.esm     | DeadMoney.esm     |
| 07   | ThePitt.esm       | HonestHearts.esm  |
| 08   | BrokenSteel.esm   | OldWorldBlues.esm |
| 09   | PointLookout.esm  | LonesomeRoad.esm  |
| 0A   | Zeta.esm          | Zeta.esm          |

Because FormID references store the mod-index as a raw byte, this reordering caused
cascading byte changes to IAD, SNAM, and WNAM subrecords throughout the file. Those are
the same referenced records with updated byte encoding - not actual content changes.

### 21 night image space color corrections

Twenty-one night-time IMAD records had their TNAM color keyframe values adjusted. Most
changes are exactly +/- 1/255 per channel (floating-point rounding corrections from
re-exporting color data at byte precision). Two records carry larger corrections of
roughly 33/255 and 37/255 per channel respectively.

Affected: WastelandNorthDayISFX, WastelandNorthNightISFX, WastelandEastNightISFX,
SuburbanNightISFX, UrbanDeepNightISFX, UrbanDeepInnerNightISFX, WastelandNightISFX,
NVWastelandNightIS, DLC03CitPitNightISFX, NVSewerNightLightISFX, NVUrbanNightIS,
NVLegateCampNightIS, NVLegateBattleNightIS, MegatonFalloutDecayNightISFX,
DLC02AnchNightISFX, DLC04MarshInnerNightISFX, NVDLC02TheNarrowsNightIS,
NVDLC02ZionValleyNightIS, NVDLC04DivideNightIS, NVDLC04NukeTransNightISFX.

### DefaultWeather lighting correction

Two values in the DefaultWeather INAM lighting data corrected from 0.0 to 1.0.

### 11 LGTM reference updates

Eleven FO3 lighting template records had their internal FormID references updated to
match the new master order. No lighting values were changed.

Affected: UtilityDefaultTemplate, NTLGuardDepotTemplate, StatesmanHotelTemplate,
OfficeDefaultTemplate, CavesDefaultTemplate, CavesDefaultTemplate02,
MetroDefaultTemplate, ShackDefaultTemplate, NeoclassicalDefaultTemplate,
UnderworldTemplate, Vault92Template.

### New records

The following records were added and are not present in david88m's file:

**Custom weather records for Hoover Dam and the endgame sequence (4 WTHR):**
- NVHooverDamWeather, NVHooverBattleWeather, NVHooverFinalBattle, NVWastelandClearEast

**Inverted daylight weather variants (3 WTHR):**
- InvertedDaylightWeather, InvertedDaylightWeatherWarm, InvertedDaylightWeatherWarmNV

**Big MT crystal emitter weather (1 WTHR):**
- NVDLC03crystalEmWeather

**Missing Lonesome Road night image spaces (2 IMAD):**
- NVDLC04LRNightISFX, NVDLC04HRNightISFX

**FO3 / DC-specific record variants (16 records):**
Dedicated Washington DC variants of lighting templates and image spaces, given a "DC"
suffix to separate them from their NV counterparts. Prevents FO3 and NV lighting setups
from sharing records that have different requirements in each region.

LGTM (11): UtilityDefaultTemplateDC, NTLGuardDepotTemplateDC, OfficeDefaultTemplateDC,
CavesDefaultTemplateDC, CavesDefaultTemplate02DC, MetroDefaultTemplateDC,
ShackDefaultTemplateDC, NeoclassicalDefaultTemplateDC, UnderworldTemplateDC,
Vault92TemplateDC, StatesmanHotelTemplateDC

IMGS (2): BasementDefaultImageSpaceDC, StatesmanHotelSpaceDC

IMAD (3): WastelandDayISFXDC, WastelandSunriseISFXDC, MegatonFalloutDecayNightISFXDC

---

## Record Count Summary

| Type | This fork | david88m's file |
|------|-----------|-----------------|
| IMGS | 110       | 110             |
| IMAD | 128       | 126             |
| WTHR | 108       | 100             |
| LGTM | 51        | 51              |
| REGN | 3         | 3               |
| LIGH | 1         | 1               |

Net additions: +8 WTHR, +2 IMAD

---

Thanks for making Clarity. They are the best visual mods for these games and your
work on the originals was amazing. If I can assist further let me know, 
I enjoyed getting this fixed.

- EchoStorm
