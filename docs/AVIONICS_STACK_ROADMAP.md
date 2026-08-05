# MAOS Avionics Stack Roadmap

**Status:** Draft v0.1 (2026-06-27). Concept/integration phase. *(v0.1 adds §7 Connected aircraft / ground systems and §8 Hardware-in-the-loop / digital twin.)*
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
   X-Plane (HIL) ─ all of the above   + compute plugins (e.g. wind)
   CAN-FIX  ─ engine/EMS        ───┘
```

**Consequences that drive the roadmap:**
- **Redundancy & selection are first-class.** Attitude can come from Stratux *and* OnSpeed; position from Stratux GPS *and* the GNX. The gateway is where sources are selected/fused — design for it. **The same selection axis admits a *simulated* source** (X-Plane in place of real sensors), which is the basis of the hardware-in-the-loop capability (§8).
- **The bus is great for scalars, weak for dynamic lists.** Traffic targets and route legs are variable-length; their FIX representation is an explicit open interface decision (see §9, §10).
- **The key DB is the single normalized model of aircraft state.** Beyond the display, that one model is what the maintenance port reads, the recorder logs, and the ground link uploads (§7).
- **Experimental display, certified nav.** pyEfis owns presentation + experimental cues (SVS, AOA, OnSpeed tones); certified nav/position/transponder stays in the GNX. The boundary is deliberate.

## 3. Component inventory

| Component | Role | Repo / source | Lang | Status |
|-----------|------|---------------|------|--------|
| **pyEfis** | Display: PFD, EMS, SVS; MFD to come | `makerplane/pyEfis` (fork `billmallard`) | PyQt6 | Active; SVS upstream PR #274 (CI green); MFD milestone open (#48–#56) |
| **fix-gateway** | The FIX bus + source plugins + recorder | `makerplane/fix-gateway` | Python | Active; plugins: `stratux`, `garmin_gnx375`, `xplane`, `compute`, `data_recorder`/`data_playback` |
| **OnSpeed-Gen2** | AOA + pitot-static + AHRS computer | `onspeed/OnSpeed-Gen2` | C++/Arduino | Flying; emits 10 Hz `ONSPEED` serial; FIX bridge TODO |
| **Stratux** | GPS, ADS-B In (traffic/weather), AHRS | companion HW + `stratux` plugin | — | GPS+AHRS decoded; runs over wired Ethernet to the hub; traffic/weather decode TODO |
| **GNX-375** | Certified IFR GPS nav, ADS-B Out, Mode S transponder | COTS (Garmin) | — | Wired NMEA nav works; Connext (attitude/traffic) blocked (heartbeat only) |
| **MGL V16** | Com radio | COTS + pyEfis Radio screen | — | Control UI in progress (WIP) |
| **Magnetometer** | Magnetic heading | MAOS project ("Smart Puck", CAN-FIX) | — | **Not started / the gap** |
| **X-Plane** | Flight sim / digital twin (HIL source + control sink) | Laminar + `xplane` plugin / PR #203 | — | Plugin exists; iron-bird HIL setup TODO (§8) |
| **pyAvMap** | Raster moving-map engine | `makerplane/pyAvMap` | PyQt5 | To be folded into pyEfis MFD map instrument (#50) |
| **makerplane-data** | Navdata/terrain currency + (future) telemetry backend | `navdata.aerocommons.org` / Cloudflare | — | Live (signed packs, R2, on-device updater); push/fleet layer TODO (§7) |

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
| *(any of the above, SIM)* | X-Plane via `xplane` plugin | HIL — source-selected (§8) |

GPS provides ground **track**, never magnetic **heading** — hence the magnetometer is the last *primary-instrument* gap.

## 5. Roadmap (phased)

Sequencing favors completing primary sensing before situational-awareness features, and integrating certified boxes before attempting any nav augmentation.

**Phase 0 — Display foundation (in hand, closing out)**
- pyEfis PFD/EMS/SVS; land SVS upstream (PR #274); fix-gateway X-Plane feed (PR #203).

**Phase 1 — Complete primary sensing**
- **Magnetometer → `HEAD`.** Build the sensor; compute **tilt-compensated heading** using the pitch/roll already on the bus (Stratux/OnSpeed AHRS) + hard/soft-iron calibration ("compass swing") + magnetic variation. Implementation as the **CAN-FIX "Smart Puck"** (ESP32/Teensy + MCP2515 + `CAN-FIX-ArduinoLib`, publishing param 389) with the gateway orchestrating calibration over the CAN-FIX Node-Config channel. *Pre-check:* whether the Stratux/OnSpeed IMU is 9-DOF (mag present) before committing to a separate unit.
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
- Complete the GNX bridge: wired **NMEA "Aviation Output"** (position/track/GS/CDI/course) ✓; **ARINC-429 GPSS** for autopilot steering; **Connext/Bluetooth** for attitude/traffic *if* it can be unblocked (currently heartbeat-only — see §10). Mitigation if Connext stays closed: attitude from Stratux/OnSpeed, traffic from Stratux.

**Phase 5 — Vehicle systems integration** *(as those subsystems mature)*
- Environmental control (cabin thermal + pressurization), electric propulsion, ICE/generator, and HV battery management onto the same bus — full treatment in §6.

**Phase 6 — Connected aircraft / ground systems** *(ground-side; gated by the flight stack being self-contained first)*
- Dedicated **hub Pi** as the aircraft data concentrator; **maintenance Ethernet port**; **unified flight-data logging** over the FIX key DB; **shore-link telemetry** (push) on the makerplane-data backend; fleet analytics / trend & exceedance alerts / audit; signed ground-only OTA. Full treatment + safety guardrails in §7.

**Cross-cutting — Hardware-in-the-loop / digital twin (iron bird)**
- Not a phase but a **verification capability used throughout**: source-select X-Plane in place of real sensors so the real cockpit hardware "flies" the sim. Stand it up early (the `xplane` plugin exists) and use it to verify each phase on the bench before flight. Full treatment + safety interlocks in §8.

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

**Practical flags:** motor + multi-cell battery telemetry is higher channel-count and rate than piston EMS — budget the CAN-FIX/FIX bus load; and these extend the same FIX key contract (§9) that should be frozen first.

## 7. Connected aircraft & ground systems

The FIX key database is the single normalized model of aircraft state. Three ground-side capabilities ride on that one model without new flight architecture: a **maintenance port**, a **unified logger**, and a **shore link** to the cloud.

**Transport split (validated).** The stack mixes transports natively at the gateway: the Stratux now feeds the hub over **wired Ethernet/UDP** (rock-solid: ~31 pkt/s, 0 dropouts vs. the WiFi link's multi-second blackouts), while distributed sensors/actuators stay on **CAN-FIX** (power-on-connector, deterministic priority, cheap nodes, linear bus). Ethernet for computer-class + high-bandwidth nodes (displays, ADS-B/traffic/weather, maps, updates); CAN-FIX for distributed sensing/control. The gateway bridges the two domains. Use ruggedized Ethernet (e.g. M12 X-coded) for the panel/backbone runs.

**Hub Pi (dedicated).** In production, split roles: a headless **hub Pi** = data concentrator (CAN-FIX↔Ethernet bridge + recorder + ground link); pyEfis **display(s)** are clients of it. A display fault then cannot take down the data hub, and the hub stays minimal/hardenable.

**Unified logging.** `fix-gateway` already ships `data_recorder`/`data_playback`. Every key — from CAN-FIX *or* IP sources — is recordable and replayable, giving one log of the entire aircraft (not per-device logs). Replay doubles as a ground-test and incident-reconstruction tool, and as the regression input for HIL (§8). Manage storage off the boot SD (USB SSD or RAM ring buffer; rotate; prune after upload).

**Maintenance port.** A panel Ethernet jack places a laptop on the aircraft LAN to read the live key DB / status dashboards. Read-only by default; writes auth-gated (below).

**Shore link / fleet backend.** Extends the existing makerplane-data platform (Cloudflare Workers/D1/R2; signed packs; on-device updater with snapshot rollback and graceful offline give-up). Today the aircraft **pulls** signed config/navdata. Add the **push** direction — upload logs/health when a ground link appears — and a cloud analytics layer for trends, exceedance alerts, audit, and fleet view. Same infrastructure, inverted data flow ("shore power + shore internet"). Concept = open-source aircraft health monitoring (airline ACARS/AHM) scaled to E-AB.

**Guardrails (requirement intent — this is avionics):**
- **AVI-CON-1** — The flight-critical path (sensors → gateway → display) *shall* function fully with no laptop, no internet, and no cloud, in all flight phases. The connected layer is opportunistic and ground-side only.
- **AVI-CON-2** — The ground link *shall* be outbound-initiated only (device pulls/pushes); the aircraft *shall not* run an internet-reachable listening service. The cloud "sees" the aircraft because the aircraft sent data up.
- **AVI-CON-3** — The cloud *shall not* command flight behavior. Updates *shall* be signed, verified on-device, applied on the ground only, and rollback-capable (reuse the config-pull pattern).
- **AVI-CON-4** — Every write/config path — including the CAN-FIX Node-Config-Set channel — *shall* be authenticated and interlocked to a ground/maintenance mode.
- **AVI-CON-5** — Hub availability *shall* degrade gracefully (displays remain sane if the hub drops); evaluate `rais`/`quorum` redundancy for the hub role.
- **AVI-CON-6** — Define flight-data ownership and consent for fleet telemetry (the Config-Manager account model is the identity basis).

## 8. Hardware-in-the-loop & the digital twin (iron bird)

Because every source normalizes into the FIX key DB, the gateway can **source-select real sensors vs. X-Plane on a per-key basis**, and nothing downstream (displays, and eventually actuator nodes) can tell the difference. This turns the real cockpit into an iron-bird simulator: the actual avionics software, displays, gateway, wiring, and CAN-FIX nodes run **unmodified** — only the physics and the outside world are synthetic.

**Setup.**
- **Digital twin co-evolution:** build the X-Plane aircraft model alongside the physical design (Onshape CAD → mass/geometry → flight model), refining fidelity as the airframe matures.
- **The one-window cockpit:** the aircraft has a single physical window; everything else is already screens (SVS). Put a monitor in the physical window showing X-Plane's out-the-window view; the EFIS SVS renders the same world (both real-terrain), so the synthetic and the sim views agree.
- **Feed:** the `xplane` plugin streams X-Plane state into the FIX bus (PR #203 already wires the RREF nav path).

**Fidelity levels.**
- **Open-loop:** X-Plane → FIX → displays; the pilot hand-flies the panel/sim. Verifies display, sensing presentation, procedures, failure annunciation.
- **Closed-loop (full iron bird):** real controls / CAN-FIX actuator nodes → gateway → **X-Plane** (the PR #203 FIX→X-Plane `send:` path) → sim physics → sensors back → gateway → displays. This is exactly the output that block was built for — and the reason it is *disabled for real flight* (it once reached the real engine port), which §8's interlocks make safe to use in HIL.

**Value.** Fly every approach, failure, and emergency hundreds of times in the real hardware before first flight; catch integration/UI/procedure bugs on the ground; regression-test new software builds by replaying recorded flights (§7 `data_playback`).

**Guardrails (requirement intent):**
- **AVI-HIL-1** — SIM vs. FLIGHT *shall* be an explicit, unmistakable, interlocked mode, with continuous annunciation; SIM mode *shall* be enterable only on the ground (e.g. weight-on-wheels / maintenance key). Never silent, never automatic in the air.
- **AVI-HIL-2** — In SIM mode, simulated data *shall* be unmistakably marked and *shall never* be presented as valid for-flight data; in FLIGHT mode, simulated sources *shall* be inert.
- **AVI-HIL-3** — Control outputs and real actuators *shall* be isolated by mode: sim control outputs route to X-Plane only; real actuator commands route to hardware only; no cross-path. Real engine/actuators *shall* be physically safed during HIL. (Directly addresses the PR #203 hazard where a sim send reached the real engine port.)
- **AVI-HIL-4** — Real-vs-sim selection *shall* be a gateway responsibility, an explicit axis of the source-selection decision (§9).

## 9. Key interface decisions (freeze before deep implementation)

1. **FIX key contract** for new data: `AOA`/`%lift`, `HEAD` (true vs magnetic + magvar handling), traffic, route — define units, sign conventions, rates, timeout semantics, quality flags.
2. **Dynamic-list representation over FIX** (traffic targets, route legs): indexed key blocks vs. side channel vs. structured value. The single most important open interface item (pyEfis #52).
3. **Source-selection / fusion** for redundant inputs (two AHRS, two position sources): priority, blending, and degraded-mode behavior, owned by the gateway. **This selection axis explicitly includes real-vs-simulated (HIL, §8)** — selection, annunciation, and interlock are one mechanism.
4. **Experimental/certified boundary:** which functions must originate in certified equipment (IFR nav, ADS-B Out, transponder) vs. may be advisory/open-source.
5. **Connectivity boundary:** what the ground link / maintenance port may read vs. write, and the authentication/interlock model for all write paths (§7).

### 9.1 Ratified keys

`fix-gateway`'s `database/*.yaml` remains the **runtime source of truth** for a
key's exact values (type, min/max, initial, timeout). This subsection is the
**design record**: the sign conventions, source-selection semantics, and
experimental/certified rationale behind each ratified block — the "why" §9
freezes, not a mirror of the DB. Add a block here when new keys are minted.

**Bearing pointers (HSI RMI)** — ratified 2026-08-04 (P5b.1/.2; fix-gateway
PR #19, pyEfis #118, makerplane-data #29). Two selectable RMI needles, each a
**bearing-TO**, degrees **magnetic**, so they register on the magnetic compass
rose (`HEAD`). All eight keys are `float`, `deg`, range `0.0–359.9` except the
two selectors.

| Key | Meaning | Writer / derivation |
|---|---|---|
| `GPSBRGT` | GPS bearing to active waypoint, **true** | `bearing` compute fn — great-circle initial bearing from `LAT`/`LONG` to `WPLAT`/`WPLON` |
| `GPSBRG` | GPS bearing to waypoint, **magnetic** | `wrap360(GPSBRGT + MAGVAR)`, mirroring the established `TRACK → TRACKM` idiom |
| `VOR1BRG` / `VOR2BRG` | VOR bearing to station, magnetic | X-Plane `nav#_bearing_deg_mag` (sim bench). On-aircraft CAN-FiX radial (canid 1228/1232) is bearing-FROM, so **radial + 180** via a `wrap360` — documented follow-on, not yet wired |
| `BRG1SRC` / `BRG2SRC` | Per-pointer source selector `0=VOR1, 1=VOR2, 2=GPS` | Pilot-set; `tol` 2000 ms; defaults pointer 1 = GPS (2), pointer 2 = VOR1 (0) |
| `BRG1` / `BRG2` | Bearing the HSI needle reads (selected source, magnetic) | A `select` per pointer routing `{VOR1BRG, VOR2BRG, GPSBRG}` per `BRGnSRC` — **exactly one writer each**, one-key-one-writer preserved (NAVSRC/COURSE precedent) |

**Frozen decisions:** (a) every bearing is magnetic and bearing-TO; GPS is made
magnetic via `MAGVAR` exactly as ground track is, so all three sources are
consistent on the rose. (b) Each pointer is independently pilot-selectable
across all three sources; `BRGn` is written only by its `select` (no source
plugin writes it directly). (c) Quality/annunciation: the display hides a needle
on heading-invalid (HSI-ANN-001) or on the pointer's own fail/old/bad quality
(HSI-FAIL-001). (d) Experimental/certified: these needles are **advisory** — VOR
bearing originates in the tuned nav radio; GPS bearing is open-stack
great-circle to the active waypoint, not an IFR navigation source (§9 item 4).

## 10. Open problems & risk register

| Item | Risk | Mitigation / note |
|------|------|-------------------|
| Magnetometer calibration | Heading error from hard/soft-iron + installation | Compass-swing cal; tilt-comp using existing AHRS; site away from ferrous/current |
| GNX Connext (BT) | Encrypted/handshake-gated; currently heartbeat-only over RFCOMM ch5 | Don't depend on it: attitude ← Stratux/OnSpeed, traffic ← Stratux; revisit only if handshake is found |
| Dynamic lists over FIX | Bus is scalar-oriented | Resolve #52 before traffic/route implementation |
| IFR legality | DIY navigator can't be legal-IFR | GNX is the certified source; open stack stays advisory |
| Chart / nav-data currency | FAA 56-day cycle | makerplane-data signed packs + on-device updater |
| pyAvMap Qt mismatch | PyQt5 vs pyEfis PyQt6 | Port engine into the PyQt6 `map` instrument (#50), don't run standalone |
| Connectivity attack surface | Internet/shore link exposes flight systems | Outbound-only, signed + verified ground-only OTA, auth-gated writes, no listening service (AVI-CON-2/3/4) |
| SIM/FLIGHT cross-contamination | Sim data treated as real, or sim output reaching real hardware | Explicit interlocked mode + by-mode control/actuator isolation + physically safed actuators (AVI-HIL-1/2/3) |
| Hub single point of failure | One Pi concentrates the aircraft data | Graceful display degradation; evaluate `rais`/`quorum` redundancy (AVI-CON-5) |
| WiFi vs wired link reliability | 2.4 GHz AP links drop (observed multi-second blackouts) | Prefer wired Ethernet for the backbone; treat WiFi as opportunistic only |

## 11. Next step for this document

Formalize the AVIONICS (AVI) subsystem into the requirements process: add `AVI-*` requirements (functional/interface/fault/verification) to an `INITIAL_REQUIREMENTS` file and the cross-repo `REQUIREMENTS_INDEX`, with the FIX key contract (§9) as the interface backbone. The `AVI-CON-*` (§7) and `AVI-HIL-*` (§8) guardrails are first-draft requirement statements to carry into that exercise. This roadmap is the input to that exercise, not a substitute for it.

## 12. References

- pyEfis MFD milestone & issues: `billmallard/pyEfis` milestone "Multi Function Display" (#48–#56).
- pyAvMap MFD candidacy: `pyAvMap/docs/MFD-Assessment.md`, `pyAvMap/docs/ARCHITECTURE.md`.
- OnSpeed orientation + integration: `onspeed/OnSpeed-Gen2/CLAUDE.md`.
- pyEfis orientation + widget/screen manual: `pyEfis/CLAUDE.md`; GitHub wiki.
- fix-gateway X-Plane / nav feed: `makerplane/FIX-Gateway` PR #203 (also the HIL control-out path, §8).
- CAN-FIX protocol + Arduino lib: `makerplane/canfix-spec`, `makerplane/CAN-FIX-ArduinoLib` (Smart Puck, §5 Phase 1).
- Ground/data backend: `navdata.aerocommons.org`, makerplane-data (Cloudflare Workers/D1/R2; §7).
