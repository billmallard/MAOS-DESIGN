> **Status addendum — 2026-08-22 (STRUCTURES).** Publishing now because MAOS-DESIGN write access, blocked at the time this analysis was written, is confirmed live as of today. The analysis body below is unchanged from its 2026-08-10 original, and its own §0/§7/§8/§9 already flag the load-bearing dependency: it is sized against the Combo-3 Hyper9 AC-X144/AC-X1 package that AER-68 found under-powered (38 kW continuous vs. a 96 kW generator floor). Since 2026-08-10, PROPULSION has published a 3-package price-vs-capability frontier on issue AER-73 (MAOS-ICE PR #18, 2026-08-11) instead of a single replacement selection, and the parent disposition issue (AER-56) is still open — Bill's most recent comment there (2026-08-11) asked PROPULSION to also survey the APU/starter-generator market. No package has been chosen yet.
>
> **Read this document for the method and the load-case shapes** (boom-root mount placement, torque-reaction and gyroscopic-fault-case derivation, crash-case treatment of the battery pack) — all of that carries over to whichever package is eventually selected. **Do not read the specific mass/torque numbers in §3–§6 as current** — they belong to a disqualified machine selection and will be re-sized against real numbers once PROPULSION's frontier resolves to one point (tracked on issue AER-69).

---

# Combo-3 Powertrain — Structural Mounting & Load-Path Requirements (AER-69)

**Status:** First-cut hand analysis (Niu/Bruhn/Timoshenko method, FAR23-analog load-case derivation). No FEM this pass — mount bracket geometry is not drawn yet, so there is nothing to mesh. Experimental Amateur-Built. Not a certification claim.
**Date:** 2026-08-10
**Owner:** STRUCTURES
**Origin:** [AER-69](/AER/issues/AER-69), downstream of [AER-68](/AER/issues/AER-68) (PROPULSION detailed spec, MAOS-ICE PR #15, `docs/COMBO3_DETAILED_ENGINEERING_V1.md`), child of [AER-56](/AER/issues/AER-56).

**Read this first if you're short on time:** §7 (adversarial findings) and §8 (verdict). The headline is not new hardware — it's that PROPULSION's own §7.1/§7.2 electric-machine disqualifier, once you add the mounting-structure mass this document derives, gets measurably worse, not better, and the battery crash case alone is a real structural driver that needs a named load path, not a bracket.

---

## 0. Dependency on AER-68's own verdict — read before trusting any number below

AER-68's verdict (§8 of the Combo-3 doc) is **a named disqualifier, not validated-with-open-gates**: the generator (38 kW continuous, rated) and the two propulsion motors (76 kW combined continuous, rated) cannot deliver the 96 kW cruise / 155 kW climb mission envelope on the manufacturer's own published numbers, independent of the Hayabusa engine's own bench-test gate. That verdict is escalated to AER-56 for Bill/Elon's disposition call: iterate Combo 3 with a corrected generator/motor selection, or fall back to Combo 1.

**I am proceeding with this issue's mount/load-path deliverable anyway**, per AER-68 §8's own statement that "the BOM, packaging, and cooling work... stand on their own regardless of the electric-machine finding and are directly reusable once a corrected generator/motor selection is made." The masses, envelopes, and mount locations below are the current sourced BOM; the load-case *method* (thrust, torque reaction, gyroscopic, crash, vibration) and the boom-root mount *location* both carry over to a corrected machine — but the specific torque/mass numbers in §3–§6 will need re-sizing the moment a different generator or motor is selected. **Every number below inherits this dependency.** I flag it once here rather than re-flagging every table.

---

## 1. What's real geometry vs. what's still an assumption

Unlike AER-68's own pass (which had no CAD connection), I pulled live geometry from the MAOS-ClaudeCAD Onshape model this pass. Real, sourced numbers below are marked **[CAD]**; everything else is carried from AER-68's BOM **[BOM]** or is my own first-cut engineering assumption **[ASSUM]**, flagged as such.

| Item | Value | Source |
|---|---|---|
| Boom OD | 305 mm (12 in) | **[CAD]** bounding box, `boom` part studio |
| Boom length | 5.03 m (16.5 ft), global station X ≈ 3.048–8.077 m | **[CAD]** |
| Boom root station | X ≈ 3.048 m (global), inside the pod's own aft envelope (pod spans X = 0–4.267 m) | **[CAD]** |
| Nacelle (motor/prop fairing) envelope | ⌀686 mm × 914 mm (27 in × 36 in), global station X ≈ 3.12–4.04 m | **[CAD]** — confirms the nacelle sits *at* the pod/boom junction, not out on the free boom span |
| Pod envelope | 4.267 m (14 ft) long × 1.53 m (60 in) wide × 1.49 m (58.6 in) tall | **[CAD]** |
| Boom wall thickness, material, temper | **Not yet a frozen Structures number** — I assumed 3 mm 6061-T6 for the sanity check in §5.3 only; this is not a spec | **[ASSUM]**, open item |
| Engine + turbo mass/envelope | 105 kg, ~650×550×500 mm | **[BOM]** AER-68 §1 rows 1–2 |
| Belt reduction mass/envelope | 12 kg, ~300×250×150 mm | **[BOM]** AER-68 §1 row 3 |
| Generator (AC-X144) mass/envelope | 63 kg incl. controller, ⌀229×355 mm | **[BOM]** AER-68 §1 row 4 |
| 2× propulsion motors (AC-X1) mass/envelope | 125 kg total (62.5 kg ea) incl. controllers, ⌀229×355 mm ea | **[BOM]** AER-68 §1 row 5 |
| Battery pack mass/envelope | 320 kg finished, ~1,200×600×250 mm module footprint (pre-layout) | **[BOM]** AER-68 §1 row 6 |
| Coaxial contra-rotating prop assembly | 40 kg, ⌀1.1–1.3 m stack, 0.6 m axial | **[BOM]** AER-68 §1 row 13 |

**A finding worth stating plainly:** the issue text conditions the boom-root mount on "the tail-boom-root ring arrangement if carried." **It is not carried.** The live CAD and AER-68's own spec both show the current architecture as two conventional cylindrical radial-flux motors (⌀229×355 mm each, NetGain HyPer9 AC-X1) driving a coaxial contra-rotating prop directly on a shaft line — not a rim/ring motor wrapping the boom. That candidate architecture (referenced in my own standing instructions as "a leading candidate... not settled") has not been carried into the Combo-3 detailed spec. I'm sizing the mount that is actually in the CAD and the BOM.

**Favorable finding, consistent with my own standing operating principle on rotating masses:** the twin-motor/prop-hub assembly sits at the boom *root* (nacelle at X ≈ 3.12–4.04 m), essentially coincident with the pod's own aft structure, not cantilevered on the free ~5 m boom span. A rotating mass mounted at or very near a stiff structural node is inherently more whirl-flutter-tolerant than the same mass hung on an unsupported extension — this placement is the good case, not the bad one. It does **not** clear whirl flutter by itself (§7.4) — that is still a separate required analysis once real mount stiffness is known.

---

## 2. Certification basis (unconfirmed — not board-frozen)

Per standing STRUCTURES practice, until the CEO/board freezes a cert basis:

- **Flight/maneuver envelope:** Part 23-derived / ASTM F44 utility class. Limit maneuvering **+3.8g / −1.52g**; ultimate = 1.5 × limit.
- **Emergency landing (crash) design factors:** I'm adopting the 14 CFR §23.561-style static ultimate inertia factor set as the working case, since no MAOS-specific crash spec exists in the design-state repos I have access to (I searched MAOS-DESIGN, MAOS-GEAR, MAOS-ICE, MAOS-MOTOR — nothing found): **forward 9.0g, downward 6.0g, upward 3.0g, sideward 1.5g, rearward 1.5g.** These factors are conventionally treated as *ultimate* design loads directly (not limit×1.5) — that's the regulation's own convention, and I've kept it rather than double-applying an ultimate factor.
- **Engine mount torque factor:** 14 CFR §23.361-style table, 4-cylinder reciprocating engine → limit torque = **2× mean/steady torque**, ultimate = 1.5× that.
- **Electric machine fault/locked-rotor torque:** no NetGain-published fault-torque figure exists for the AC-X144/AC-X1 (open item, §9). I've adopted a **3× rated-torque** multiplier as a first-cut, conservative-but-unsourced placeholder consistent with general PM/induction-machine short-circuit torque literature — **flagged, not vendor-confirmed.**
- **Gyroscopic load case:** 14 CFR §23.371-style method — pitch rate at pull-up to limit load factor, ω = (n−1)·g / V. **I do not have AERO's Vₐ figure.** I substituted V = 155 KTAS (cruise TAS, the only speed this whole documentation chain has used so far) — this is a **non-conservative stand-in**: real Vₐ is normally lower than cruise speed, and a lower V in this formula gives a *higher* pitch rate and a *larger* gyroscopic moment. Treat every gyroscopic number below as a lower bound until AERO publishes Vₐ.

**Everything above is unconfirmed and will re-size the moment the board picks different numbers.** This is stated once here; every load case in §3–§6 inherits it.

---

## 3. Load cases (method, then applied per mount in §4–§6)

### 3.1 Steady thrust
Reacted axially through the mount into the boom-root structure (motors/prop) or through the pod's forward structure (engine, if firewall-forward — siting is an open item, §9). Cruise thrust = 826 N (AER-68 §5.4, itself carried from the trade study, AERO-unconfirmed installed-efficiency figure). This is a small load relative to the torque/gyroscopic/crash cases below and is not the governing case for any mount in this package.

### 3.2 Torque reaction (engine, generator, prop) — mission-target sizing, not rated-hardware sizing
**I am sizing every torque-reaction fitting to the mission power target (96 kW cruise / 155 kW climb, traced to the board's own power budget, `maos-1g1b2m-architecture-decision`), not to the current Hyper9 hardware's rated-continuous ceiling.** Rationale: the mount geometry (bolt pattern, bracket, shaft-line alignment) is a mechanical interface that a corrected generator/motor selection will still need to transmit mission torque through. Sizing mounts to the disqualified hardware's lower rated torque would just mean re-sizing every fitting again the moment PROPULSION corrects the machine selection — wasted work in exactly the place Elon's order says to avoid it (don't optimize a part that's about to change). The *mission* torque target traces to a real, board-level source; it is not invented.

| Reaction point | Mean/steady torque | Method | Limit | Ultimate |
|---|---:|---|---:|---:|
| Engine crank (mount reacts equal/opposite) | 155 N·m (114 lbf·ft) at ~6,500 RPM, 105.3 kW crank power required to deliver 96 kW electrical through gen(94%)+belt(97%) losses | §23.361-style, 4-cyl factor = 2× | 309 N·m (228 lbf·ft) | **464 N·m (342 lbf·ft)** |
| Generator input shaft (belt-reduction output) | 296 N·m (218 lbf·ft) at 3,300 RPM input | 3× fault-torque placeholder | 887 N·m (654 lbf·ft) | **1,330 N·m (981 lbf·ft)** |
| Each propulsion motor, at climb target (155 kW combined ÷ 2, at 3,300 RPM) | 224 N·m (165 lbf·ft) | 3× fault-torque placeholder | 673 N·m (496 lbf·ft) | **1,009 N·m (744 lbf·ft)** |

The generator mount fitting is the single largest torque-reaction requirement in the package — nearly 3× the engine mount's ultimate torque — because the 3× electrical-fault multiplier is applied on top of an already-higher input torque (belt-reduced RPM is lower than crank RPM, so torque is higher for the same power). **This should be checked against a real NetGain-published fault-torque figure before it's load-bearing** (open item, §9) — 3× is my own placeholder, not a vendor number.

### 3.3 Gyroscopic (prop)
Per-disk polar MOI estimated at **1.88 kg·m²** (k≈0.4 × 15 kg blade-mass-per-disk × 0.56 m² — EXTRAP, no real prop MOI exists, same evidence category as AER-68's own prop-assembly mass estimate). At rated-continuous spin (3,300 RPM = 345.6 rad/s) and the placeholder pitch rate (§2, V=155 KTAS, n=3.8g): **limit gyroscopic moment ≈ 224 N·m (165 lbf·ft) per disk, ultimate ≈ 336 N·m (248 lbf·ft).**

**Contra-rotation cancellation — real, but not universal.** Because the two disks counter-rotate at (nominally) equal RPM, net gyroscopic moment on the *shared* structure should approximately cancel in the symmetric, both-motors-running case. **It does not cancel in the single-motor-inoperative case** — an engine-out-equivalent electrical fault leaves one disk spinning with its full, uncancelled gyroscopic moment reacted through that motor's own mount into the boom root. This is a real, board-traceable load case (an emergency/failure condition, not an invented one) and it is the governing combined case for the boom-root fitting:

**Combined boom-root fitting, single-motor-out case (simple sum of torque-reaction ultimate + gyroscopic ultimate, not a rigorous multi-axis interaction — flagged as a conservative bound pending real fitting geometry): ≈ 1,345 N·m (992 lbf·ft).**

### 3.4 Crash / emergency landing (14 CFR §23.561-style factors, §2)
Applied as static ultimate inertia loads at each mass's own CG, reacted through its mount into primary structure. See §6 for the battery-specific numbers — it is the only mass in this package large enough for the crash case to be the structural driver.

### 3.5 Vibration / isolation (Hayabusa reciprocating engine)
Four-stroke inline-4 firing frequency = (cylinders/2) × RPM/60:
- Idle (~1,200 RPM): **40 Hz**
- Cruise (~6,500 RPM): **217 Hz**

Standard elastomeric engine-mount practice (Lord Corp aerospace mount catalogs; validated on Corvair/Subaru/Rotax auto-engine homebuilt conversions) targets mount natural frequency **fn ≤ ⅓–½ of the lowest sustained excitation frequency** to sit inside the isolation region (f/fn > √2). Using idle as the lowest steady excitation: **fn target ≤ 13–20 Hz.** The engine necessarily passes through the mount's resonance during start/stop transients — accepted practice is a brief transient passage, not continuous dwell; this needs a ground-run/bench confirmation, not a static claim (§8).

**This is a real added-mass item, not free**: soft engine mounts (elastomeric bushings sized for this frequency band, plus the local stiffening the softer mount demands at the airframe attachment to avoid a secondary flexible-support problem) are heavier than a rigid bolted mount of the same load capacity. Folded into the 5% fraction in §5, flagged as possibly optimistic for a soft-mount design (§9).

---

## 4. Per-mount requirement set

### 4.1 Engine + turbo (105 kg)
- **Location: not yet sited.** AER-68 §2 flags "firewall-forward or pod-forward, per standing 1G architecture... none of which is sited yet." I cannot draw a load path to primary structure without a station. **Open item, owner: whoever holds the pod internal layout (Structures/PROPULSION joint, or CAD Operator once geometry is directed).**
- Load cases: steady thrust (reaction into mount only, not transmitted structurally beyond the mount — engine does not itself produce net aircraft thrust in this series-hybrid architecture), torque reaction (§3.2, governing), crash 9g/6g/3g/1.5g/1.5g (§3.4), vibration isolation (§3.5, governing for mount type — soft/isolated, not rigid).
- **Vibration isolation drives the mount topology choice** (soft elastomeric mount vs. rigid), which in turn drives the local airframe stiffness requirement at the attachment points. This is a real design decision, not a detail — call it out to whoever sites the engine.

### 4.2 Belt reduction + generator (75 kg combined)
- Shared shaft-line assembly — belt reduction output couples directly to generator input, so these two items share a mount base/plate treated as one structural subassembly with a single load path to primary structure, not two independent mounts.
- **Alignment tolerance for the 2:1–2.3:1 belt drive at ~6,500 RPM engine-side input is a real machining/mounting-stiffness requirement** (AER-68 §2's own flag), not a bolt-together afterthought — belt-drive misalignment at this RPM class drives both fatigue life and noise/vibration, and the mount base needs enough torsional/bending stiffness to hold pulley alignment under the full torque-reaction case (§3.2), not just static weight.
- Load cases: torque reaction (§3.2, governing — this is the single highest torque-reaction requirement in the package), crash 9g/6g/3g/1.5g/1.5g, vibration (secondary — this subassembly does not carry the Hayabusa's reciprocating imbalance directly, but does see whatever isn't isolated by the engine's own soft mount, transmitted through the belt).

### 4.3 2× propulsion motors + prop hub interface (125 kg motors, at boom root — real CAD location, §1)
- **Location: real, confirmed.** Nacelle envelope ⌀686×914 mm at global station X≈3.12–4.04 m, essentially coincident with the pod's own aft structure (pod ends at X=4.267 m). This is the favorable whirl-flutter placement noted in §1 — mount design should preserve that by tying the motor mount ring directly into the pod's aft frame/bulkhead, not into the free boom tube alone.
- **Quick closed-form sanity check on the boom tube itself** (Timoshenko/Bredt thin-wall torsion, assumed 3 mm 6061-T6 wall — my own placeholder, §1): reacting the single-motor ultimate torque (1,009 N·m) through the boom's gross torsional section gives τ ≈ 2.35 MPa, against a 6061-T6 ultimate shear allowable of ~207 MPa — **about 1% utilization.** The boom tube's gross section is not the constraint. **The constraint is the local fitting** — the bracket-to-tube attachment, fastener pattern, and any doubler/reinforcement at the motor-ring interface — which cannot be sized without real bracket geometry (open item, §9). Don't read the tube check as "the mount is easy" — it only clears the tube; the fitting is still unsized.
- Load cases: steady thrust (both motors), torque reaction per motor (§3.2), **gyroscopic — governing, single-motor-out case (§3.3, 1,345 N·m ultimate combined)**, crash 9g/6g/3g/1.5g/1.5g, vibration (secondary — electric motors do not carry the Hayabusa's reciprocating signature, but the mount should still be checked against whatever residual imbalance the belt-driven generator side transmits structurally, and against the prop's own once-per-rev aerodynamic excitation, which is an AERO input I don't have — open item).
- **Whirl flutter is not cleared by this analysis.** Per my own standing operating principle: a clean static margin here does not clear whirl flutter, which is a separate rotor-speed-dependent aeroelastic stability boundary (Bisplinghoff/Ashley/Halfman; Reed). The favorable root-mounted location lowers the risk relative to a boom-tip arrangement, but does not substitute for the analysis. **Flagged as a required follow-on once real mount stiffness (Kθ, Kψ) is known from the fitting design** — this is exactly the structural-dynamics gate item I own for the tail boom.

### 4.4 Battery pack (320 kg)
- **SAFETY (AER-62 §4) requires physical/thermal zoning away from HV bus wiring, contactors, and motor-controller electronics** (AER-68 §2) — a shared bay is a common-cause hazard against the aircraft's sole emergency reserve. This constrains *where* the pack can go; it is not resolved yet (open item, §9, owner: pod layout).
- Load cases: crash 9g/6g/3g/1.5g/1.5g (**governing — see §6, this is the structural driver for this mount, not a secondary check**), vibration (low — battery packs are not RPM-excited, but the mount should be checked against the generator/motor mounts' transmitted vibration if co-located in the pod).
- **This is a single 320 kg (706 lb) item — heavier than most GA singles' entire useful load — and it needs a crash load path into primary pod structure comparable to how occupant seats are treated, not a bracket into skin.** See §6 for the number.

---

## 5. First-cut mounting-structure mass estimate

Fraction-based EXTRAP against GA engine-mount practice (typical steel-tube truss mounts for a Lycoming/Continental-class installation run ~3–5% of the carried mass; I've applied a higher fraction where the load cases derived above are unusually demanding — the belt-alignment stiffness requirement and the crash-critical battery mount):

| Mount group | Carried mass | Fraction | Structure mass |
|---|---:|---:|---:|
| Engine + turbo | 105 kg | 5% (soft-mount penalty, §3.5) | 5.3 kg |
| Belt reduction + generator | 75 kg | 6% (alignment-stiffness penalty, §4.2) | 4.5 kg |
| 2× motors + boom-root ring/bracket | 125 kg | 6% (combined torque+gyro fitting, §3.3) | 7.5 kg |
| Battery pack | 320 kg | 8% (crash-critical load path, §6) | 25.6 kg |
| **Total mounting structure, first cut** | | | **≈ 43 kg (94 lb)** |

**This is not a member-sized estimate** — it's a fraction against comparable GA practice, appropriate to this fidelity level (no mount geometry exists to size a truss against). Treat it as a placeholder for the CG/convergence loop, not a design number, until real bracket/fitting geometry exists (§9).

---

## 6. Battery crash case — the number the adversarial mandate asked for

At 320 kg and the §2 crash factors (applied as static ultimate, per §23.561 convention):

| Direction | Factor | Ultimate load |
|---|---:|---:|
| Forward | 9.0g | **28,253 N (6,351 lbf)** |
| Downward | 6.0g | 18,835 N (4,234 lbf) |
| Upward | 3.0g | 9,418 N (2,117 lbf) |
| Sideward | 1.5g | 4,709 N (1,059 lbf) |
| Rearward | 1.5g | 4,709 N (1,059 lbf) |

**Forward governs, at 6,351 lbf ultimate.** That is a real structural fitting requirement — comparable in magnitude to a primary landing-gear attachment, not a bracket-and-four-bolts problem. It needs a dedicated crash-load path into the pod's primary structure (strap/frame arrangement analogous to seat-track or gear-fitting practice, Niu's fitting-design method), sized and proof-tested before it is trusted (§8). Combined with SAFETY's zoning constraint (§4.4), the battery's structural footprint in the pod is a real, non-trivial integration constraint that the pod layout has not resolved yet.

---

## 7. Adversarial findings

Per the issue's mandate: I went looking for a reason this drives unacceptable structural mass or a CG problem. Two independent findings, one of which compounds a finding PROPULSION already made.

### 7.1 — Mounting structure compounds PROPULSION's own mass finding
AER-68 §6 already found the installed powertrain at **738 kg (1,627 lb), 62.6% of MTOW**, and flagged a 427 lb (36%) overrun against the trade study's own cited 1,200 lb useful-load figure. Adding this document's first-cut mounting-structure mass (§5, ~43 kg / 94 lb — itself likely an underestimate, since it's a fraction against non-crash-critical GA practice applied to a battery mount that the crash case (§6) shows is *not* a typical GA mount problem):

**Installed powertrain + mounts ≈ 781 kg (1,722 lb) — 66.2% of MTOW, and 522 lb (43.5%) over the cited 1,200 lb useful-load figure, before crew, payload, or fuel.**

This is not a new disqualifier — it's the same one PROPULSION already named, restated with the structural mass added. I'm not asserting a formal empty-weight/useful-load violation (I don't own that ledger either, same caveat AER-68 gave), but the gap is now larger, not smaller, once mounting structure is counted, and it moves in the wrong direction regardless of which generator/motor selection eventually resolves §7.1/§7.2 of AER-68 — a heavier, properly-rated electric machine (AER-68 §8's own stated likely fix) will add mass on top of this, not reduce it.

### 7.2 — The battery crash case is a real structural driver, independent of the electric-machine disqualifier
Even setting the generator/motor power shortfall aside entirely, the 320 kg battery pack's forward crash load (6,351 lbf ultimate, §6) is a genuine structural sizing problem that exists regardless of which combo survives the board's disposition call — Combo 1 (the Emrax/Rotax fallback) still carries a comparably large battery reserve per the trade study's own architecture. **This is not a disqualifier by itself** (mass problems here have levers — a different chemistry, a smaller reserve, a different pack layout), but it is a real, quantified structural requirement that the pod layout needs to resolve with an actual crash-load path, not a placeholder bracket, before detailed pod structure is drawn.

### 7.3 — What did NOT turn out to be a problem
The boom-root motor/prop mount location is structurally favorable (§1, §4.3) — real CAD confirms it sits at the pod's own aft structure, not cantilevered on the free boom span, which is the good case for both static bending and (qualitatively) whirl flutter. The boom tube's gross torsional capacity is not a constraint for the torque-reaction case (§4.3, ~1% utilization at an assumed 3 mm wall). Neither is a green light on its own — the local fitting is still unsized, and whirl flutter is still a separate required analysis — but neither is a disqualifier candidate either.

---

## 8. Verdict

**Not a structural disqualifier by itself, but a real compounding finding on top of PROPULSION's already-named electric-machine disqualifier, plus one independent, quantified structural driver (the battery crash case) that needs resolution regardless of the board's Combo-3-vs-Combo-1 disposition call.**

- The mount *locations and load-case shapes* derived here are reusable once a corrected generator/motor selection lands (per AER-68 §8's own framing) — the boom-root motor placement, the torque-reaction methodology, and the crash-case treatment all carry over.
- The mount *masses and specific torque numbers* will need re-sizing the moment the electric machine selection changes size class (a genuinely 96 kW+-continuous machine, per AER-68 §7.1, is a physically larger, heavier unit than the Hyper9 line).
- **Escalating to AER-56 (parent), per the issue's own instruction**, with §7's findings as the structural input to the same disposition call AER-68 already opened there.

**Never signing off primary structure on analysis alone** (hard boundary): every number above is a first-cut hand analysis. The tests this needs before any of it is trusted: (1) static proof-load test of each mount fitting to its ultimate case, once real geometry exists; (2) ground-run/bench vibration survey of the engine mount before first flight, to confirm the isolation target in §3.5 against a real transmissibility measurement, not the closed-form estimate; (3) whirl-flutter analysis (separate from this static work) once real mount stiffness is known for the boom-root motor mount, ahead of a ground-vibration test.

---

## 9. Open items

1. **Engine + turbo siting** (firewall-forward vs. pod-internal) — not sited in the CAD or any design doc I have access to. Owner: PROPULSION/Structures joint, or CAD Operator once directed.
2. **Battery pack station within the pod** — constrained by SAFETY's HV-wiring zoning requirement (AER-62 §4) but not resolved to a real station/CG. This is a direct input to the CG/convergence loop I own reporting into — I cannot post a real structural-mass-by-station table until this exists.
3. **Boom wall thickness/temper** — not yet a Structures-frozen number; my §4.3/§5 sanity check used an assumed 3 mm 6061-T6 placeholder only. A real boom structural sizing pass is a separate, not-yet-opened piece of work.
4. **Real prop assembly MOI** — EXTRAP (§3.3), same evidence class as AER-68's own prop-mass estimate. A vendor/analog figure would tighten the gyroscopic case materially.
5. **NetGain fault/locked-rotor torque figure** — not published; the 3× multiplier in §3.2 is my own placeholder, not vendor-sourced.
6. **AERO's Vₐ** — needed to replace the non-conservative V=155KTAS placeholder in §3.3's gyroscopic pitch-rate derivation.
7. **Real mount/bracket geometry** — nothing in this document is FEM-checked because no bracket geometry exists yet to mesh. Once PROPULSION's corrected generator/motor selection (or the board's Combo-1 fallback decision) lands, this is the natural next step: CAD Operator drafts real mount geometry, Structures runs CalculiX against it, cross-checked against the hand values above.
8. **Whirl flutter** — explicitly not cleared by this document (§4.3, §8); required once mount stiffness is known.
9. **Corrected generator/motor selection** — AER-68's own open item; every mass/torque number in this document inherits it (§0).

---

*Analysis by STRUCTURES, MAOS Design Board*
*Version 1 — 2026-08-10. First-cut mount interface, load-case, and mounting-structure-mass analysis for the Combo-3 powertrain per AER-69: real CAD-sourced boom/pod/nacelle geometry (§1), FAR23-analog load-case derivation with cert basis explicitly flagged unconfirmed (§2–§3), per-mount requirements (§4), first-cut mounting-structure mass (§5), the battery crash-load number the adversarial mandate asked for (§6), and adversarial findings that compound, but do not independently repeat, PROPULSION's already-named electric-machine disqualifier (§7–§8).*
*R&D guidance for Experimental Amateur-Built development. Not a certification claim.*
