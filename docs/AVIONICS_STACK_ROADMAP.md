# MAOS Avionics Stack Roadmap

**Status:** Draft v0 (2026-06-19). Concept/integration phase.
**Subsystem:** AVIONICS (AVI) — not yet represented in `REQUIREMENTS_INDEX_V0.md`; this roadmap seeds it.
**Owner:** display + sensing + navigation integration across the MakerPlane software repos and COTS boxes.

## Safety Notice

Experimental Amateur-Built category. Everything here is **advisory / supplemental** and is not a certified navigation, attitude, or traffic system. Where a certified function is required (IFR navigation, ADS-B Out position source, Mode S transponder), it is provided by **certified COTS equipment**; the open-source stack displays and augments — it does not replace. No claim of airworthiness, certification, or compliance.

---

## 1. Purpose & scope

Define how the open-source avionics components and the COTS equipment combine into a coherent panel, and sequence the remaining work. In scope: flight/engine display, sensor sourcing, traffic/weather, navigation integration, and the interfaces between them. Out of scope: the certified boxes' internals, and aircraft-level structures/propulsion (owned by the other MAOS-* repos).

## 2. Architecture — the FIX bus is the interface contract

The stack is **source-pluggable around a single normalized data bus**. `fix-gateway` owns the bus (the **FIX database**): every datum is a named key with units, range, timeout, and quality flags. Sensors/sources are plugins that publish keys; the display subscribes. This is the avionics-side analogue of the design repo's ICD philosophy — **freeze the FIX key contract (units, sign conventions, rates, timeout semantics), then implement sources and display independently.**

```
   SOURCES (plugins)                 fix-gateway (FIX bus)            CONSUMERS
   ────────────────────              ────────────────────            ──────────────
   Stratux  ─ GPS, ADS-B in, AHRS ┐
   OnSpeed  ─ AOA, pitot-static,  │   normalized keys:               pyEfis  ─ PFD / EMS /
              AHRS, (ONSPEED ser.) ├─▶ LAT LONG GS TRACK ALT IAS  ─▶          SVS / MFD (display)
   GNX-375  ─ IFR GPS nav, ADS-B  │   AOA PITCH ROLL ALAT VS HEAD          + audio (OnSpeed tones)
              Out, transponder    │   COURSE CDI GSI  TFC*… (traffic)
   Magnetometer ─ mag heading  ───┤   + quality flags + source select
   CAN-FIX  ─ engine/EMS        ───┘   + compute plugins (e.g. wind)
```

**Consequences that drive the roadmap:**
- **Redundancy & selection are first-class.** Attitude can come from Stratux *and* OnSpeed; position from Stratux GPS *and* the GNX. The gateway is where sources are selected/fused — design for it.
- **The bus is great for scalars, weak for dynamic lists.** Traffic targets and route legs are variable-length; their FIX representation is an explicit open interface decision (see §7, §8).
- **Experimental display, certified nav.** pyEfis owns presentation + experimental cues (SVS, AOA, OnSpeed tones); certified nav/position/transponder stays in the GNX. The boundary is deliberate.

## 3. Component inventory

| Component | Role | Repo / source | Lang | Status |
|-----------|------|---------------|------|--------|
| **pyEfis** | Display: PFD, EMS, SVS; MFD to come | `makerplane/pyEfis` (fork `billmallard`) | PyQt6 | Active; SVS upstream PR #274 (CI green); MFD milestone open (#48–#56) |
| **fix-gateway** | The FIX bus + source plugins | `makerplane/fix-gateway` | Python | Active; plugins: `stratux`, `garmin_gnx375`, `xplane`, `compute` |
| **OnSpeed-Gen2** | AOA + pitot-static + AHRS computer | `onspeed/OnSpeed-Gen2` | C++/Arduino | Flying; emits 10 Hz `ONSPEED` serial; FIX bridge TODO |
| **Stratux** | GPS, ADS-B In (traffic/weather), AHRS | companion HW + `stratux` plugin | — | GPS+AHRS decoded; **traffic/weather decode TODO** |
| **GNX-375** | Certified IFR GPS nav, ADS-B Out, Mode S transponder | COTS (Garmin) | — | Wired NMEA nav works; Connext (attitude/traffic) blocked (heartbeat only) |
| **MGL V16** | Com radio | COTS + pyEfis Radio screen | — | Control UI in progress (WIP) |
| **Magnetometer** | Magnetic heading | MAOS project (to build) | — | **Not started / the gap** |
| **pyAvMap** | Raster moving-map engine | `makerplane/pyAvMap` | PyQt5 | To be folded into pyEfis MFD map instrument (#50) |
| **makerplane-data** | Navdata/terrain currency | `navdata.aerocommons.org` | — | Live (terrain/airports/obstacles/water packs) |

## 4. Data-source map

Where each primary datum comes from in the current stack:

| Flight datum (FIX keys) | Source(s) | Status |
|-------------------------|-----------|--------|
| Position — `LAT` `LONG` `GS` `TRACK`, GPS alt | Stratux GPS (+ GNX SBAS, IFR-grade) | ✓ available |
| Pitot-static — `IAS`, pressure `ALT` | OnSpeed | ✓ computed; FIX bridge TODO |
| Angle of attack — `AOA`, %lift | OnSpeed | ✓ computed; FIX bridge TODO |
| Attitude — `PITCH` `ROLL` `ALAT` `ROT`, `VS` | Stratux AHRS **and** OnSpeed AHRS | ✓ (redundant) |
| **Magnetic heading — `HEAD`** | — | ✗ **gap** (magnetometer project) |
| ADS-B traffic — `TFC*` | Stratux | ✓ at HW; decode + FIX model + render TODO |
| Weather (FIS-B) | Stratux | ✓ at HW; decode + render TODO |
| Nav — `COURSE` `CDI` `GSI`, approaches | GNX-375 (certified) | Wired NMEA ✓; GPSS/Connext in progress |
| Engine — EMS keys | CAN-FIX sensors | ✓ (pyEfis EMS screens) |
| Baro setting, dimmer, units | pyEfis control head | ✓ |

GPS provides ground **track**, never magnetic **heading** — hence the magnetometer is the last *primary-instrument* gap.

## 5. Roadmap (phased)

Sequencing favors completing primary sensing before situational-awareness features, and integrating certified boxes before attempting any nav augmentation.

**Phase 0 — Display foundation (in hand, closing out)**
- pyEfis PFD/EMS/SVS; land SVS upstream (PR #274); fix-gateway X-Plane feed (PR #203).

**Phase 1 — Complete primary sensing**
- **Magnetometer → `HEAD`.** Build the sensor; compute **tilt-compensated heading** using the pitch/roll already on the bus (Stratux/OnSpeed AHRS) + hard/soft-iron calibration ("compass swing") + magnetic variation. Natural home: a **fix-gateway `compute` plugin** (same pattern as `wind_components`). *Pre-check:* whether the Stratux/OnSpeed IMU is 9-DOF (mag present) before committing to a separate unit.
- **OnSpeed → FIX bridge.** A `fix-gateway` plugin parsing the `ONSPEED` serial sentence (10 Hz, fixed-width ASCII + checksum) → publish `AOA`/`IAS`/`ALT`/`PITCH`/`ROLL`/`ALAT`/`VS`. Pairs with a new pyEfis **AOA / %-lift instrument** (none today).
- Establish **air-data / AHRS source-selection** policy in the gateway (which source wins; degraded behavior when one drops).

**Phase 2 — Situational-awareness MFD** (GitHub milestone "Multi Function Display", pyEfis #48–#56)
- `map` instrument (2D top-down surface, #49); vector layers reusing SVS data (#51); raster chart layer porting pyAvMap (#50); airspace (#55).
- **Dynamic-list FIX model** (#52) — the interface decision that gates traffic + route.
- **Traffic** (#53): extend `stratux` GDL90 decode (0x14 Traffic Reports; also the GNX ADS-B source) → FIX model → render. **Weather** (FIS-B) decode + render.
- Chart/nav-data currency via makerplane-data (#56).

**Phase 3 — COTS radio & transponder integration**
- **Radio:** finish MGL V16 control head in the pyEfis Radio screen (freq/active-standby/volume).
- **Transponder:** the GNX-375 *is* the Mode S / ADS-B-Out transponder — add a control-head path (squawk / IDENT / mode) over its serial/control interface. New hardware not required.

**Phase 4 — Certified GPS-navigator integration ("the official navigator problem")**
- Treat the **GNX-375 as the authoritative IFR nav/position source**; pyEfis is the display + experimental layer. Do **not** build a DIY IFR navigator (legality + RNAV/approach engine scope).
- Complete the GNX bridge: wired **NMEA "Aviation Output"** (position/track/GS/CDI/course) ✓; **ARINC-429 GPSS** for autopilot steering; **Connext/Bluetooth** for attitude/traffic *if* it can be unblocked (currently heartbeat-only — see §8). Mitigation if Connext stays closed: attitude from Stratux/OnSpeed, traffic from Stratux.

**Phase 5 — Vehicle systems integration** *(as those subsystems mature)*
- Environmental control (cabin thermal + pressurization), electric propulsion, ICE/generator, and HV battery management onto the same bus — full treatment in §6.

**Longer horizon**
- GI-275-style autopilot commander (heading bug / alt preselect / GPSS) on experimental-category aircraft only.
- Vector MFD on SVS data (alternative to raster); data-manager maturation.

## 6. Vehicle systems integration (ECS, propulsion, battery)

The same source-pluggable model extends to the aircraft's non-flight systems — environmental control (cabin thermal + pressurization), electric propulsion (motor/inverter), the ICE/turbine generator, and high-voltage battery management (BMS). These are later phases (gated by subsystem maturity), but they require **no new avionics architecture**: each is another CAN-FIX source publishing normalized FIX keys, displayed by the existing widget set.

**Integration path — identical to engine EMS today:**

```
ECS / Motor+Inverter / BMS / ICE controllers ─CAN-FIX─▶ fix-gateway plugin ─▶ FIX keys ─▶ pyEfis (gauges/screens + mode commands)
```

The MAOS-ECS, MAOS-MOTOR, and MAOS-ICE requirements already specify **normalized telemetry interfaces** (units, update rates, fault flags, commanded-vs-measured mode state — e.g. ECS-IF-004, MOT-IF-004, ICE-IF-005). Those map directly onto FIX keys, so the interface contract is partly pre-specified.

**Reuses (no new framework):**
- Arc / bar / numeric / ganged gauges with green/yellow/red bands + annunciation, pointed at new keys: cabin ΔP and temps, HV bus voltage/current, battery SOC/temp/cell health, motor & inverter temperatures, generator output. pyEfis already displays `VOLT`/`CURRNT`.
- New **screen layouts** via the screenbuilder — an **ECS page** and a **power/energy page** for the series hybrid.
- **Mode commanding** via the existing button/condition/action model + FIX command keys (ECS off/thermal/pressurization; motor generator-vs-propulsor; etc.), the same way the autopilot buttons command modes.

**New display work this actually drives:**
- A **power/energy synoptic** (ICE→generator→battery→motor power-flow, SOC, derate state) — richer than a traditional EMS strip.
- A formal **Crew Alerting System (CAS)** — a prioritized warning/caution/advisory area. A pressurized HV-hybrid has far more abnormal conditions (HV/bus faults, battery thermal limits, pressurization loss) than per-gauge coloring serves well. This is the principal new display feature these systems push pyEfis toward.

**Hard boundary (safety):** pyEfis is **advisory display + HMI, not a controller.** Control loops, state machines, and *protection* remain in the subsystem controllers — consistent with the MAOS-DESIGN governance that subsystem repos own firmware, deterministic transitions, and fault handling. For the two life-/energy-safety items — **cabin pressurization** and the **HV battery** — primary protection must never depend on the advisory display; pyEfis annunciates and commands modes only.

**Practical flags:** motor + multi-cell battery telemetry is higher channel-count and rate than piston EMS — budget the CAN-FIX/FIX bus load; and these extend the same FIX key contract (§7) that should be frozen first.

## 7. Key interface decisions (freeze before deep implementation)

1. **FIX key contract** for new data: `AOA`/`%lift`, `HEAD` (true vs magnetic + magvar handling), traffic, route — define units, sign conventions, rates, timeout semantics, quality flags.
2. **Dynamic-list representation over FIX** (traffic targets, route legs): indexed key blocks vs. side channel vs. structured value. The single most important open interface item (pyEfis #52).
3. **Source-selection / fusion** for redundant inputs (two AHRS, two position sources): priority, blending, and degraded-mode behavior, owned by the gateway.
4. **Experimental/certified boundary:** which functions must originate in certified equipment (IFR nav, ADS-B Out, transponder) vs. may be advisory/open-source.

## 8. Open problems & risk register

| Item | Risk | Mitigation / note |
|------|------|-------------------|
| Magnetometer calibration | Heading error from hard/soft-iron + installation | Compass-swing cal; tilt-comp using existing AHRS; site away from ferrous/current |
| GNX Connext (BT) | Encrypted/handshake-gated; currently heartbeat-only over RFCOMM ch5 | Don't depend on it: attitude ← Stratux/OnSpeed, traffic ← Stratux; revisit only if handshake is found |
| Dynamic lists over FIX | Bus is scalar-oriented | Resolve #52 before traffic/route implementation |
| IFR legality | DIY navigator can't be legal-IFR | GNX is the certified source; open stack stays advisory |
| Chart / nav-data currency | FAA 56-day cycle | makerplane-data signed packs + on-device updater |
| pyAvMap Qt mismatch | PyQt5 vs pyEfis PyQt6 | Port engine into the PyQt6 `map` instrument (#50), don't run standalone |

## 9. Next step for this document

Formalize the AVIONICS (AVI) subsystem into the requirements process: add `AVI-*` requirements (functional/interface/fault/verification) to an `INITIAL_REQUIREMENTS` file and the cross-repo `REQUIREMENTS_INDEX`, with the FIX key contract (§7) as the interface backbone. This roadmap is the input to that exercise, not a substitute for it.

## 10. References

- pyEfis MFD milestone & issues: `billmallard/pyEfis` milestone "Multi Function Display" (#48–#56).
- pyAvMap MFD candidacy: `pyAvMap/docs/MFD-Assessment.md`, `pyAvMap/docs/ARCHITECTURE.md`.
- OnSpeed orientation + integration: `onspeed/OnSpeed-Gen2/CLAUDE.md`.
- pyEfis orientation + widget/screen manual: `pyEfis/CLAUDE.md`; GitHub wiki.
- fix-gateway X-Plane / nav feed: `makerplane/FIX-Gateway` PR #203.
