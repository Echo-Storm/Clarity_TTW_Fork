# Clarity TTW - Fix Handoff Notes

## Summary

The TTW version had 22 FO3 image space records using raw vanilla game data instead of
your Clarity-processed values. Everything on the New Vegas side was ported correctly.
All weather, region, IMAD, and lighting template records appear intact and untouched.
The fixed ESP is a drop-in replacement at the same file size - only DNAM bytes were
patched in-place.

---

## What We Found

### The broken records

22 IMGS (image space) records contained vanilla FO3 data instead of Clarity values.
These covered the most commonly visited FO3 environments:

- DefaultImageSpaceExterior (all FO3 outdoor areas)
- WastelandImageSpace01
- UrbanImageSpace01 / UrbanImageSpace01intro
- gUrbanImageSpace01 / gWastelandImageSpace01 / gVaultImageSpace01
- MetroImageSpace01 / MetroDefaultImageSpace01
- ShackInterior01 / ShackInteriorWaste01
- CaveInteriorIM01 / CaveTestBrightImageSpace
- OfficeDefaultImageSpace / NeoClassDefaultImageSpace
- RCDefaultImageSpace
- IndDefaultImageSpace
- GNRRoofImageSpace01
- LamplightInterior
- GrantTest
- BasementDefaultImageSpace (1 field off - emissive 2.0 vs your 1.6)
- StatesmanHotelSpace (1 field off - brightness 1.0 vs your 0.7)

All 22 were verified against both Clarity.esp (FO3) and Clarity.esp (NV) - the values
in both originals were identical for every shared record.

### Why this caused the reported problems

Vanilla FO3 image spaces carry a permanent gray tint overlay (tintA around 0.5) that
your mod disables by setting tintA = 0.0. Those 22 records still had tintA active,
so FO3 areas in TTW were playing through a constant gray wash the entire time.

Additionally the vanilla cinematic values differ substantially from your processed ones:

  DefaultImageSpaceExterior example -
    saturation:  vanilla 0.125   vs  Clarity 1.200
    brightness:  vanilla 1.200   vs  Clarity 0.900
    contrast:    vanilla 0.900   vs  Clarity 0.604
    tintA:       vanilla 0.510   vs  Clarity 0.000
    bloom.blurR: vanilla 1.000   vs  Clarity 0.030
    bloom.alpha: vanilla 0.030   vs  Clarity 0.800

The Citadel / time-of-day-goes-black bug was almost certainly DefaultImageSpaceExterior
specifically - the most critical record in the entire mod, used for all FO3 outdoor
areas, was running vanilla values the whole time.

### What was NOT broken

- All 65 NV Clarity records: correctly ported, values match your originals exactly
- All 128 IMAD records: untouched and appear correctly ported
- All 108 WTHR records: intact, including the Hoover Dam / Fort / Colorado River
  custom weathers and the Searchlight entries
- All 3 REGN records: intact
- All 51 LGTM records: intact
- HEDR version: already 1.34 (correct for TTW)

---

## What Was Changed

Single operation: for each of the 22 broken IMGS records, the full 152-byte DNAM
block was replaced with your Clarity-processed values (sourced from whichever original
had the record - all 22 matched both originals identically).

No other record types were modified. No records were added or removed.
File size is unchanged at 331,363 bytes.

### Verification

Three checks passed on the output file:
1. All records with Clarity originals now match those originals exactly
2. No colored tints remain (all cinematic tintR/G/B are balanced)
3. HEDR version confirmed 1.34

---

## HDR Audit Notes (no action taken)

Three records flagged during the HDR audit - all confirmed intentional from your
originals, left unchanged:

- UltraLuxeKitchen: eyeAdaptSpeed = 8.0 (NV Clarity only - deliberate)
- NVDLC04NukeIS: contrast = 5.0 (NV Clarity only - nuke flash effect)
- DLC05MZImageSpaceTheFinalFrontier: brightClamp = 0.01 (FO3 Clarity only - space)

---

## Record Type Inventory (TTW final)

  IMGS: 110   (22 fixed, 88 were already correct)
  IMAD: 128
  WTHR: 108
  LGTM:  51
  REGN:   3
  LIGH:   1
  TES4:   1   (HEDR 1.34)
