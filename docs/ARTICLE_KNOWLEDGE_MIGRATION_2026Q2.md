# MAOS-DESIGN Article Knowledge Migration (2026 Q2)

Purpose: Consolidate cross-domain configuration decisions from aerocommons articles into MAOS-DESIGN as program-level geometry and architecture guidance.

Scope note: This is R&D guidance for Experimental Amateur-Built development. It is not a certification claim.

## Source Articles

- 2026-04-06-maos-geometry-design-decisions.md
- 2026-03-24-voltage-decision-400vdc.md
- 2026-04-05-maos-1g1b2m-architecture-decision.md
- 2026-04-05-maos-generator-selection.md
- 2026-04-05-maos-propulsion-redundancy-battery.md

## Imported Decisions and Working Baseline

- Pod-and-boom geometry remains valid as first-pass but is not frozen.
- Key unresolved configuration trades remain active: boom vertical position, wing direct-attach versus short pylon, and landing-gear architecture.
- Electrical baseline is 400 VDC for current integration phase with documented review triggers.
- System architecture decisions must preserve graceful degradation across major failure nodes.

## Configuration Guidance to Carry Into This Repo

- Keep geometry decisions coupled to thermal, propulsion, and handling implications.
- Maintain explicit design-gate criteria for configuration lock and rollback triggers.
- Document all cross-subsystem assumptions with traceable handoff artifacts.

## Open Decisions Assigned to MAOS-DESIGN

- Resolve top-boom vs mid-boom with stall/deep-stall risk evidence.
- Resolve wing mounting approach with structural and interference-drag trade evidence.
- Resolve landing-gear concept with stroke and clearance constraints.
- Publish updated design-point assumptions for altitude and mission profile.

## Immediate Work Items

1. Create docs/CONFIGURATION_TRADE_STUDY_BOOM_WING_GEAR.md.
2. Create docs/CONFIGURATION_LOCK_CRITERIA_AND_ROLLBACK_TRIGGERS.md.
3. Create docs/SUBSYSTEM_ASSUMPTIONS_REGISTER_V0.md.
4. Create docs/UPDATED_MISSION_DESIGN_POINT_V1.md.

## Suggested Deliverables to Add Next

- docs/CONFIGURATION_TRADE_STUDY_BOOM_WING_GEAR.md
- docs/CONFIGURATION_LOCK_CRITERIA_AND_ROLLBACK_TRIGGERS.md
- docs/SUBSYSTEM_ASSUMPTIONS_REGISTER_V0.md
- docs/UPDATED_MISSION_DESIGN_POINT_V1.md
