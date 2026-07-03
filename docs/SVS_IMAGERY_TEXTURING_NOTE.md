# Note: photo-realistic (satellite) SVS texturing — feasibility & utility

2026-07-03, Bill + Claude. Prompted by Nighthawk Guardian's photo-realistic
SVS rendering (satellite-overlay look). Companion to
[EFIS_GREENFIELD_DESIGN.md](EFIS_GREENFIELD_DESIGN.md); see also pyEfis
issues #86 (FLTA) / #87 (attitude smoothing) from the same comparison.

## Feasibility on Pi 5: YES — Track 1b already built the machinery

Not a rendering-performance problem: the clipmap already draws textured,
fogged, world-anchored triangles; imagery is one more RGB fetch in the
fragment shader (V3D fill rate at 1080p is fine). The work is data
logistics, and the pattern now exists:

- **Per-ring imagery textures mirroring the Track 1b per-level heightmaps**:
  ring-matched resolution, same anchor-grid rebuilds, same async worker,
  same world-anchoring/phase-stability/drift-bound discipline (all the bugs
  are pre-fought). ~512 px RGB x 7 rings = a few MB GPU; region decodes are
  ms-scale on the worker.
- **Data tier**: Sentinel-2-class 10 m (free, global) ~25-40 MB/deg^2
  compressed — a flying region in single-digit GB. NAIP 1 m is ~100x and a
  non-starter. Distribution = one more signed makerplane-data pack kind
  with currency, like terrain/water.
- **Compositing**: imagery as base layer only; the clearance/safety
  colouring stays authoritative (caution/warning/conflict tint or override
  in the fragment shader).

## Utility: contested — and certified avionics mostly votes no (for the PFD)

Garmin/Collins/Honeywell could trivially drape imagery and deliberately
ship stylised terrain on certified SVS. The human-factors case:

- SVS earns its keep in IMC, where photorealism adds visual noise: low-
  contrast browns/greens bury symbology and the clearance colouring that
  actually prevents CFIT.
- **Baked lighting lies**: capture-day sun/shadows/season frozen in the
  imagery conflict with live slope shading (can invert perceived relief)
  and misrepresent current conditions — a misleading-terrain-information
  hazard in the AC 25-11B Table 4-6 sense.
- At SVS viewing geometry, 10 m imagery smears a few miles out; the crisp
  look is a high-zoom static demo artifact.
- Where imagery IS useful: VFR orientation and the airport environment —
  an MFD/moving-map virtue. **Chart draping (queued as P10) shares 100% of
  this infrastructure and has clearer pilot utility** (pilots brief from
  sectionals, not from orbit).

## Recommendation

If/when built: `svs_texture_source: hypsometric | imagery | hybrid`
(hybrid = imagery base under an authoritative clearance tint), sequenced
BEHIND P10 chart draping, and settled by flight A/B — one knob, both
modes, Bill's eyes over real mountains.
