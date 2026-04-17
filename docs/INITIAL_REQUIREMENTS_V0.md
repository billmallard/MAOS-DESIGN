# MAOS-DESIGN Initial Requirements v0

Status: Draft / Working notes. Intentionally early and incomplete.
Purpose: Seed requirements for backlog grooming and interface alignment.

## Aircraft-Level Requirements

- DES-SYS-001: The repository shall define and maintain one controlled baseline aircraft configuration at all times.
- DES-SYS-002: The baseline configuration shall include target MTOW, empty weight estimate, CG envelope, and margin assumptions.
- DES-SYS-003: All subsystem packaging volumes (FCS, ICE, MOTOR, ECS, GEAR, VISION) shall be represented in the baseline geometry.
- DES-SYS-004: Baseline geometry changes that affect subsystem interfaces shall include an impact note.

## Interface Requirements

- DES-IF-001: Aircraft-level interface contracts shall be versioned and published for propulsion, flight controls, ECS, landing gear, and vision.
- DES-IF-002: Interface contracts shall define units, sign conventions, rates, tolerances, and timeout semantics.
- DES-IF-003: Mechanical interface contracts shall define mounting datums and load directions.
- DES-IF-004: Electrical interface contracts shall define nominal voltage ranges and fault annunciation channels.

## Integration Requirements

- DES-INT-001: The design process shall allow at least two viable propulsion architectures until evidence eliminates one.
- DES-INT-002: Major architecture decisions shall document rationale, assumptions, and confidence level.
- DES-INT-003: A subsystem may not be declared integrated unless all required interface fields are mapped and verified.
- DES-INT-004: Integration assumptions shall be traceable to a requirement or explicit open issue.

## Verification Requirements

- DES-VER-001: Each baseline release shall include updated mass and CG calculations.
- DES-VER-002: Each baseline release shall include an interface consistency check across subsystem repos.
- DES-VER-003: Requirement-to-evidence traceability shall be maintained for all aircraft-level requirements.
- DES-VER-004: Unverified assumptions shall be listed in an open assumptions register.
