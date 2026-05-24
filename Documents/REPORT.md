# Clarity TTW - Fork Report

This is a report of the changes made in Echo's fork of Clarity TTW. Clarity is originally
by phoenix0113 - your TTW port is what this fork is built on. The fork was built against
a 2020 build sourced from the original TTW forums (now defunct), which was newer than
your 2018 Nexus release (Nexus ID 62481). All differences listed here have been verified
with a binary-level ESP parser comparing the two files directly by record.

---

## The Core Bug

The TTW version had 22 FO3 image space (IMGS) records carrying raw vanilla FO3 data
instead of your Clarity-processed values. Everything on the New Vegas side was ported
correctly. The bug only affected FO3 content.

Vanilla FO3 image spaces carry a permanent gray tint overlay (tintA around 0.5). Your
mod disables this by setting tintA to 0.0. Those 22 records still had tintA active, so
every FO3 location in TTW was rendering through a constant gray wash the whole time.

The cinematic values were also wrong across all 22. Using DefaultImageSpaceExterior as
an example - this is the most critical record in the mod, covering all FO3 outdoor areas:

| Field        | Vanilla FO3 | Clarity |
|-------------|-------------|---------|
| saturation   | 0.125       | 1.200   |
| brightness   | 1.200       | 0.900   |
| contrast     | 0.900       | 0.604   |
| tintA        | 0.510       | 0.000   |
| bloom.blurR  | 1.000       | 0.030   |
| bloom.alpha  | 0.030       | 0.800   |

The Citadel / time-of-day-goes-black bug that users reported was almost certainly this
record specifically. DefaultImageSpaceExterior was running vanilla values the entire time.

All 22 broken records were verified against both your FO3 Clarity.esp and NV Clarity.esp.
The values in both originals were identical for every shared record, so the fix source
was unambiguous.

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
(132 or 148 bytes depending on the record) and were padded out to the full 152-byte
layout used by the rest of the file.

---

## What Was Confirmed Correct

Before making any changes, a full audit was run on the TTW file. These record types
were all intact and untouched relative to your originals:

- All 65 NV Clarity IMGS records: values match your originals exactly
- All 128 IMAD records: correctly ported
- All 108 WTHR records: intact, including the Hoover Dam / Fort / Colorado River
  custom weathers and the Searchlight entries
- All 3 REGN records: intact
- All 51 LGTM records: intact
- HEDR version: 1.34 (correct for TTW)

Three records were flagged during an HDR audit but confirmed intentional and left
unchanged:

- UltraLuxeKitchen: eyeAdaptSpeed = 8.0 (NV Clarity only)
- NVDLC04NukeIS: contrast = 5.0 (NV Clarity only - nuke flash effect)
- DLC05MZImageSpaceTheFinalFrontier: brightClamp = 0.01 (FO3 Clarity only - space)

---

## Additional Changes in the Fork

Beyond the bug fix, the fork includes the following changes relative to your current
Nexus release. These were present in the 2020 TTW forum build and are carried forward.

### Master list reordering

The master file load order was reorganized to place NV DLCs before FO3 DLCs:

| Slot | This fork        | Your Nexus release |
|------|------------------|--------------------|
| 00   | FalloutNV.esm    | FalloutNV.esm      |
| 01   | DeadMoney.esm    | Fallout3.esm       |
| 02   | HonestHearts.esm | Anchorage.esm      |
| 03   | OldWorldBlues.esm| ThePitt.esm        |
| 04   | LonesomeRoad.esm | BrokenSteel.esm    |
| 05   | Fallout3.esm     | PointLookout.esm   |
| 06   | Anchorage.esm    | DeadMoney.esm      |
| 07   | ThePitt.esm      | HonestHearts.esm   |
| 08   | BrokenSteel.esm  | OldWorldBlues.esm  |
| 09   | PointLookout.esm | LonesomeRoad.esm   |
| 0A   | Zeta.esm         | Zeta.esm           |

Because FormID references store the mod-index as a raw byte, this reordering caused
cascading changes to IAD, SNAM, and WNAM subrecords throughout the file. Those are all
the same referenced records with updated byte encoding - not actual content changes.

### DLCPittSteelMillInterior - compressed flag removed

The compressed record flag (0x400) was cleared on this IMGS record. It is now stored
uncompressed.

### 21 night IMAD color corrections

Twenty-one night-time image adapter records had their TNAM color keyframe values
adjusted. Most changes are exactly +/- 1/255 per channel, which are floating-point
rounding corrections from re-exporting color data at byte precision. Two records carry
larger corrections: one keyframe darkened by roughly 33/255 per channel, one brightened
by roughly 37/255 per channel.

Affected: WastelandNorthDayISFX, WastelandNorthNightISFX, WastelandEastNightISFX,
SuburbanNightISFX, UrbanDeepNightISFX, UrbanDeepInnerNightISFX, WastelandNightISFX,
NVWastelandNightIS, DLC03CitPitNightISFX, NVSewerNightLightISFX, NVUrbanNightIS,
NVLegateCampNightIS, NVLegateBattleNightIS, MegatonFalloutDecayNightISFX,
DLC02AnchNightISFX, DLC04MarshInnerNightISFX, NVDLC02TheNarrowsNightIS,
NVDLC02ZionValleyNightIS, NVDLC04DivideNightIS, NVDLC04NukeTransNightISFX.

WastelandDayISFX and WastelandSunriseISFX also have IAD reference updates consistent
with the master reordering.

### DefaultWeather lighting correction

The DefaultWeather WTHR record (the main FO3 DC wasteland weather) had two values in
its INAM lighting data corrected from 0.0 to 1.0.

### 11 LGTM reference updates

Eleven FO3 lighting template records had their internal FormID references updated to
match the new master order. No lighting values were changed - only the referenced image
space FormIDs were remapped.

Affected: UtilityDefaultTemplate, NTLGuardDepotTemplate, StatesmanHotelTemplate,
OfficeDefaultTemplate, CavesDefaultTemplate, CavesDefaultTemplate02,
MetroDefaultTemplate, ShackDefaultTemplate, NeoclassicalDefaultTemplate,
UnderworldTemplate, Vault92Template.

### New records

The following records were added and are not present in your Nexus release:

**Custom weather records for Hoover Dam and the endgame sequence (4 WTHR):**
- NVHooverDamWeather
- NVHooverBattleWeather
- NVHooverFinalBattle
- NVWastelandClearEast

**Inverted daylight weather variants (3 WTHR):**
- InvertedDaylightWeather
- InvertedDaylightWeatherWarm
- InvertedDaylightWeatherWarmNV

**Big MT crystal emitter weather (1 WTHR):**
- NVDLC03crystalEmWeather

**Missing Lonesome Road night image spaces (2 IMAD):**
- NVDLC04LRNightISFX
- NVDLC04HRNightISFX

**FO3 / DC-specific record variants (16 records):**

Dedicated Washington DC variants of lighting templates and image spaces, given a "DC"
suffix to separate them from their NV counterparts. This prevents FO3 and NV lighting
setups from sharing records that have different requirements in each region.

LGTM (11): UtilityDefaultTemplateDC, NTLGuardDepotTemplateDC, OfficeDefaultTemplateDC,
CavesDefaultTemplateDC, CavesDefaultTemplate02DC, MetroDefaultTemplateDC,
ShackDefaultTemplateDC, NeoclassicalDefaultTemplateDC, UnderworldTemplateDC,
Vault92TemplateDC, StatesmanHotelTemplateDC

IMGS (2): BasementDefaultImageSpaceDC, StatesmanHotelSpaceDC

IMAD (3): WastelandDayISFXDC, WastelandSunriseISFXDC, MegatonFalloutDecayNightISFXDC

---

## Record Count Summary

| Type | This fork | Your Nexus release |
|------|-----------|--------------------|
| IMGS | 110       | 110                |
| IMAD | 128       | 126                |
| WTHR | 108       | 100                |
| LGTM | 51        | 51                 |
| REGN | 3         | 3                  |
| LIGH | 1         | 1                  |

Net additions over your Nexus release: +8 WTHR, +2 IMAD
