# Clarity TTW - Echo's Fork

A fork of Clarity TTW, the image space and weather overhaul for Tale of Two Wastelands.

Clarity is originally by phoenix0113: https://www.nexusmods.com/newvegas/mods/62481
The TTW port was made by david88m and is hosted in the old files section of that page.

This fork fixes a significant bug present in the TTW version where 22 FO3 image space
records were left with vanilla FO3 data instead of Clarity's processed values, causing
a permanent gray wash over all FO3 areas in TTW. It also carries forward additions from
a 2020 build sourced from the original TTW forums, which included feature additions that
were never merged back to the Nexus page.

---

## Requirements

- Tale of Two Wastelands 3.3+
- Fallout: New Vegas
- Fallout 3
- All DLC for both games

---

## Installation

Drop `Clarity_TTW_2k26.esp` into your TTW Data folder and enable it. No other files
required. Place it after any weather mods in your load order.

This is a standalone ESP. It does not require the original Clarity TTW to be installed.

---

## What This Fixes

### 22 FO3 image spaces had vanilla data instead of Clarity values

The most significant issue. Twenty-two FO3 image space records were carrying unmodified
vanilla FO3 data. Vanilla FO3 image spaces carry a permanent gray tint overlay (tintA
around 0.5). Clarity disables this by setting tintA to 0.0. Those 22 records still had
tintA active, so every FO3 location in TTW was rendering through a constant gray wash.

The cinematic values were also wrong across all 22. Using DefaultImageSpaceExterior as
an example - this record covers all FO3 outdoor areas:

| Field        | Vanilla FO3 | Clarity |
|-------------|-------------|---------|
| saturation   | 0.125       | 1.200   |
| brightness   | 1.200       | 0.900   |
| contrast     | 0.900       | 0.604   |
| tintA        | 0.510       | 0.000   |
| bloom.blurR  | 1.000       | 0.030   |
| bloom.alpha  | 0.030       | 0.800   |

The Citadel / time-of-day-goes-black bug that users reported was almost certainly
DefaultImageSpaceExterior specifically - the most critical record in the mod, covering
all FO3 outdoor areas, was running vanilla values the whole time.

All 22 values were verified against both your FO3 Clarity.esp and NV Clarity.esp. The
values in both originals were identical for every shared record.

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
- BasementDefaultImageSpace (emissive corrected: 2.0 -> 1.6)
- StatesmanHotelSpace (brightness corrected: 1.0 -> 0.7)

### DLCPittSteelMillInterior - compressed flag removed

The compressed record flag was cleared on this image space record.

---

## Additional Changes vs the Nexus Release

The following were present in the 2020 TTW forums build and are carried forward here.

### Master list reordering

NV DLCs are placed before FO3 DLCs in the master list. This is the standard ordering
for TTW plugins built against a modern TTW install.

### 21 night image space color corrections

Twenty-one night-time IMAD records had their color keyframe values adjusted. Most
changes are exactly +/- 1/255 per channel (floating-point rounding corrections). Two
records carry larger corrections of roughly 33/255 and 37/255 per channel respectively.

### DefaultWeather lighting correction

Two values in the DefaultWeather INAM lighting data corrected from 0.0 to 1.0.

### New weather records

Eight weather records covering areas that previously fell back to unmanaged vanilla
conditions:

- NVHooverDamWeather, NVHooverBattleWeather, NVHooverFinalBattle (Hoover Dam / endgame)
- NVWastelandClearEast
- InvertedDaylightWeather, InvertedDaylightWeatherWarm, InvertedDaylightWeatherWarmNV
- NVDLC03crystalEmWeather (Big MT crystal emitter)

### New Lonesome Road night image spaces

Two missing night-time image adapters added: NVDLC04LRNightISFX, NVDLC04HRNightISFX

### FO3 / DC-specific record variants

16 dedicated Washington DC variants of lighting templates and image spaces, given a
"DC" suffix to prevent FO3 and NV lighting setups from sharing records that have
different requirements in each region.

---

## Record Count

| Type | This fork | Nexus release (david88m) |
|------|-----------|--------------------------|
| IMGS | 110       | 110                      |
| IMAD | 128       | 126                      |
| WTHR | 108       | 100                      |
| LGTM | 51        | 51                       |
| REGN | 3         | 3                        |
| LIGH | 1         | 1                        |

---

## Compatibility

Works with any mod that does not also override the same image space or weather records.
A compatibility patch for Desert Natural Weathers TTW is not included in this release.

---

## Credits

- **phoenix0113** - original Clarity mod (Fallout New Vegas / Fallout 3)
- **david88m** - TTW port hosted in old files on the Clarity Nexus page
- **EchoStorm** - bug fixes, additional records, this fork

Full technical documentation of every change is in the `Documents/` folder.
