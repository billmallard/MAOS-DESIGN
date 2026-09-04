> **Status addendum v2 — 2026-09-04 (STRUCTURES).** Re-sized against PROPULSION's live-recommended corrected package instead of the disqualified Hyper9 hardware, and against SAFETY's now-published crash-load spec instead of my own placeholder. Three things changed since the 2026-08-22 v1 addendum:
>
> 1. **PROPULSION passed a kill gate on a corrected electric machine (AER-73, MAOS-ICE PR #16, 2026-08-10) and narrowed a price-vs-capability frontier to one recommended package (PR #18 V2, 2026-08-11): Twin EMRAX 268 (MV/CC winding) + BAMOCAR D3-700/400 controllers, "Package A," $57,200 used/salvage (14% over the $50k line), installed powertrain mass ≈634 kg — 14% *lighter* than the disqualified Hyper9 selection, not heavier.** This directly falsifies my own v1 §7.1 claim that "a genuinely 96 kW+-continuous machine... is a physically larger, heavier unit than the Hyper9 line" and that the mass gap "moves in the wrong direction... regardless of which selection eventually resolves." That claim was reasoned, at the time, from AER-68's own cost-sanity heuristic (expensive aviation-class machines tend to be heavy) — it was a plausible inference, not a sourced number, and the real vendor data now available contradicts it. Flagging the correction here rather than quietly fixing it — see revised §7.1.
> 2. **SAFETY published the MAOS emergency-landing load-factor requirement (AER-74, 2026-08-10)** — the exact factor set I had adopted as an unconfirmed placeholder (9.0g/6.0g/3.0g/1.5g/1.5g) is now the board-owned spec, and it explicitly names the battery-retention case (§6 below) as a confirmed requirement, not an assumption. My crash numbers don't change; their cert basis does — from "my own placeholder" to "SAFETY-established, AER-74."
> 3. **No board disposition has landed on AER-56.** As of this pass (2026-09-04), Elon/Bill have not yet picked a point on PROPULSION's frontier — the most recent activity is Bill's 2026-08-11 request that PROPULSION also survey the APU/starter-generator market (done, closed negative, per AER-59's 2026-08-23 weekly pass) and PROPULSION's continuing weekly market watch (H3X HPDM-250 and Evolito E800 logged as of 2026-08-30, neither displacing Package A as the standing recommendation). **This document sizes against Package A because it is PROPULSION's own live recommendation, not because it has been chosen** — the mount locations and load-case method carry over regardless of the board's eventual pick; the specific mass/torque numbers are Package-A-specific and will move again if the board picks differently (or if PROPULSION's weekly watch turns up something that displaces Package A).
>
> **Read this document for the method and the load-case shapes** (boom-root mount placement, torque-reaction and gyroscopic-fault-case derivation, crash-case treatment of the battery pack) — those carry over to whichever package is eventually selected. Read §1, §3.2, §5, §6, §7 for the numbers current as of this pass.

---

# Combo-3 Powertrain — Structural Mounting & Load-Path Requirements (AER-69)

**Status:** First-cut hand analysis (Niu/Bruhn/Timoshenko method, FAR23-analog load-case derivation). No FEM this pass — mount bracket geometry is not drawn yet, so there is nothing to mesh. Experimental Amateur-Built. Not a certification claim.
**Date:** 2026-08-10 (v1), revised 2026-09-04 (v2 — see status addendum above)
**Owner:** STRUCTURES
**Origin:** [AER-69](/AER/issues/AER-69), downstream of [AER-68](/AER/issues/AER-68) (PROPULSION detailed spec, MAOS-ICE PR #15) and [AER-73](/AER/issues/AER-73) (PROPULSION corrected-machine kill gate + frontier, MAOS-ICE PR #16/#18), child of [AER-56](/AER/issues/AER-56).

**Read this first if you're short on time:** §7 (adversarial findings) and §8 (verdict). The v2 headline is a self-correction: my v1 assertion that a corrected electric machine would make the mass problem worse is wrong, on real vendor data. The corrected package is lighter, the mass/useful-load gap narrows (though it does not close), and the boom-root torque-reaction case is now sized against a real manufacturer torque rating instead of an unsourced 3× fault-torque placeholder. The battery crash case remains the single largest quantified structural driver in the package, and its cert basis is now a confirmed board requirement (AER-74), not a placeholder.

---

## 0. Dependency on the board's still-open disposition call

Combo-3-as-originally-specified (NetGain HyPer9 AC-X144/AC-X1) stays disqualified — AER-68's finding (§8 of that document) is unchanged: 38 kW continuous generator against a 96 kW floor, 76 kW combined motors against a 155 kW climb floor, both manufacturer-published ceilings, not bench-test-pending questions. **What's new since my v1 pass:** PROPULSION ran the kill gate that finding demanded (AER-73) and found a real, sourced, corrected machine class — EMRAX 268 (MV winding, combined cooling) — that clears both floors with margin (22% generator, 51% climb combined), at $57,200 used/salvage (14% over the $50k line) and 634 kg installed, and narrowed a subsequent price-vs-capability frontier (single-motor variants, both disqualified — one on a corrected datasheet figure, one on SAFETY's AER-81 ruling) to that same package as the standing recommendation.

**No board pick has been made.** AER-56 is still `todo`; PROPULSION's own tracking (AER-59, weekly cadence) confirms no disposition as of 2026-09-04. **I am sizing this pass against Package A (Twin EMRAX 268) because it is the only real, sourced, mission-clearing candidate on the table** — sizing against the already-dead Hyper9 numbers a second time would be exactly the "polish a part that's about to change" waste Elon's order warns against, and Package A is materially more likely to be the eventual hardware than a disqualified selection is. But this is still not a board pick: every number below inherits that dependency, same discipline as v1. If the board instead selects a single-motor package, falls back to Combo 1, or a weekly market-watch pass displaces Package A, the load-case *method* and boom-root mount *location* carry over; the masses and torques in §3–§6 do not.

---

## 1. What's real geometry vs. what's still an assumption

Unlike AER-68/AER-73's own passes (no CAD connection in either), I pulled live geometry from the MAOS-ClaudeCAD Onshape model in the original v1 pass (2026-08-10) and have not re-pulled it this revision — no signal that the boom/pod/nacelle geometry has moved since. Real, sourced numbers below are marked **[CAD]**; everything else is carried from PROPULSION's BOM **[BOM]** or is my own first-cut engineering assumption **[ASSUM]**, flagged as such.

| Item | Value | Source |
|---|---|---|
| Boom OD | 305 mm (12 in) | **[CAD]** bounding box, `boom` part studio, 2026-08-10 pull — not re-verified this revision |
| Boom length | 5.03 m (16.5 ft), global station X ≈ 3.048–8.077 m | **[CAD]**, same pull |
| Boom root station | X ≈ 3.048 m (global), inside the pod's own aft envelope (pod spans X = 0–4.267 m) | **[CAD]**, same pull |
| Nacelle (motor/prop fairing) envelope | ⌀686 mm × 914 mm (27 in × 36 in), global station X ≈ 3.12–4.04 m | **[CAD]**, same pull — confirms the nacelle sits *at* the pod/boom junction |
| Pod envelope | 4.267 m (14 ft) long × 1.53 m (60 in) wide × 1.49 m (58.6 in) tall | **[CAD]**, same pull |
| Boom wall thickness, material, temper | Not yet a frozen Structures number — 3 mm 6061-T6 assumed for the sanity checks in §4.3 only, not a spec | **[ASSUM]**, open item |
| Engine + turbo mass/envelope | 105 kg, ~650×550×500 mm | **[BOM]** AER-68 §1 rows 1–2, unchanged by AER-73 |
| Belt reduction mass/envelope | 12 kg, ~300×250×150 mm; ratio corrected to ~1.44:1 (was ~2:1) to match EMRAX 268's 4,500 RPM limiting speed | **[BOM]** AER-73 PR#16 §3 row 2 |
| Generator (EMRAX 268 MV/CC + BAMOCAR D3-700/400) mass/envelope | **30.4 kg** (21.9 kg motor + 8.5 kg controller) — down from Hyper9's 63 kg. Motor envelope: **⌀268 mm × ~114 mm axial** (combined-cooling variant) — EXTRAP from EMRAX's published dimensional envelope for this frame, not independently re-pulled by PROPULSION or me this pass; flagged for confirmation before it's load-bearing for a bracket design. Controller: 355×230×135 mm (BAMOCAR D3-700/400, sourced, AER-73 PR#16 §2) | **[BOM]** AER-73 PR#16 §3 row 3, motor envelope **[ASSUM]** |
| 2× propulsion motors (EMRAX 268 MV/CC + BAMOCAR D3-700/400) mass/envelope | **60.8 kg total** (30.4 kg ea) — down from Hyper9's 125 kg. Same motor/controller envelope as the generator, identical unit | **[BOM]** AER-73 PR#16 §3 row 4 |
| Battery pack mass/envelope | 320 kg finished, ~1,200×600×250 mm module footprint (pre-layout) — unchanged, battery selection not touched by AER-73 | **[BOM]** AER-68 §1 row 6 |
| Coaxial contra-rotating prop assembly | 40 kg, ⌀1.1–1.3 m stack, 0.6 m axial — unchanged **but flagged: PROPULSION's own open item (AER-73 PR#16 §6 item 4) notes the actuator-disk sizing this envelope traces to was built on the disqualified AC-X1's 3,300 RPM/110 Nm characteristic, not EMRAX 268's 4,500 RPM/250 Nm one — a re-check with AERO is still owed and hasn't landed.** I'm carrying the envelope forward as the best available number, not as a confirmed re-sized figure. | **[BOM]** AER-68 §1 row 13, re-check owed per AER-73 |

**A finding restated from v1, still true:** the issue text conditions the boom-root mount on "the tail-boom-root ring arrangement if carried." **It is not carried.** Both the disqualified package and the corrected one use two conventional cylindrical motors (Hyper9's radial-flux frame before, EMRAX 268's axial-flux "pancake" frame now) driving a coaxial contra-rotating prop directly on a shaft line — not a rim/ring motor wrapping the boom.

**Favorable finding, unchanged and still true:** the twin-motor/prop-hub assembly sits at the boom *root* (nacelle at X ≈ 3.12–4.04 m), essentially coincident with the pod's own aft structure, not cantilevered on the free ~5 m boom span. This placement does not itself clear whirl flutter (§7.4) — that is still a separate required analysis once real mount stiffness is known — but it remains the favorable case, and it is unaffected by which electric machine occupies that location.

**A new favorable finding this pass, worth naming explicitly:** the generator and both propulsion motors are now the *same physical unit* (EMRAX 268 MV/CC + BAMOCAR D3-700/400, ×3), where the disqualified package used two different frame classes (AC-X144 generator, AC-X1 motor). **This is a real parts-commonality win** — one motor mount bracket design, proven at three installation points instead of two different bracket designs for two different component families. It directly serves the buildability principle (fewer unique parts to design, fit-check, and stock) and is worth flagging to PROPULSION/CAD as a reason to keep this package's mount interface standardized across all three positions if the board adopts it.

---

## 2. Certification basis

Per standing STRUCTURES practice, until the CEO/board formally freezes a cert basis for the whole aircraft:

- **Flight/maneuver envelope:** Part 23-derived / ASTM F44 utility class. Limit maneuvering **+3.8g / −1.52g**; ultimate = 1.5 × limit. **Still unconfirmed** — no board freeze found this pass.
- **Emergency landing (crash) design factors: now a confirmed SAFETY requirement, not a placeholder.** [AER-74](/AER/issues/AER-74) (SAFETY, 2026-08-10) established **forward 9.0g, downward 6.0g, upward 3.0g, sideward 1.5g, rearward 1.5g**, applied as static ultimate directly (not limit×1.5, per the regulation's own §23.561(b)(2)/CAR 3.386 convention SAFETY adopted) — the identical figures I had carried as my own unconfirmed placeholder in v1. AER-74 explicitly scopes this to cover "any flight-critical retention item," named by consequence, and explicitly names the battery pack and "the engine/generator/motor mount groups STRUCTURES already sized in AER-69" as in scope. **This is the one load case in this document that has moved from "flagged assumption" to "board-owned spec" since v1** — nothing else in this cert-basis list has been frozen.
- **Engine mount torque factor (Hayabusa, reciprocating):** 14 CFR §23.361-style table, 4-cylinder reciprocating engine → limit torque = **2× mean/steady torque**, ultimate = 1.5× that. Unchanged methodology — the ICE side of this package didn't change in AER-73.
- **Electric machine fault/peak torque: now a real manufacturer number, not a placeholder multiplier.** v1 used an unsourced **3× rated-torque** fault multiplier because no NetGain fault-torque figure existed. EMRAX publishes both a continuous torque rating (**250 N·m**, MV/HV/LV windings alike — this is a real steady-state envelope figure) and a peak (60 s) power rating (**210 kW at the 4,500 RPM limiting speed**, all windings). I derive a peak torque bound from that: **T_peak ≈ P_peak / ω_limit ≈ 446 N·m** — a real vendor number, but flagged as a *first-cut lower bound*: it assumes peak torque occurs at the limiting speed, which is true only if EMRAX 268's torque-speed curve is power-limited across its full range. If the real curve is closer to constant-torque at low RPM (common for PM machines before the power-limited region), true peak torque at low RPM could be materially higher. **PROPULSION/CAD should pull the actual EMRAX 268 torque-speed curve before this number is trusted past first-cut.** This replaces the 3× placeholder as the governing torque-reaction case in §3.2 — a real improvement in evidence class even with the caveat attached.
- **Gyroscopic load case:** 14 CFR §23.371-style method — pitch rate at pull-up to limit load factor, ω = (n−1)·g / V. **I still do not have AERO's Vₐ figure** (unchanged open item). I substitute V = 155 KTAS (cruise TAS), still a **non-conservative stand-in** — flagged, unchanged from v1. **Prop spin rate for the gyroscopic moment is revised this pass**: v1 used 3,300 RPM (the disqualified AC-X1's rated speed); I now use **4,500 RPM (EMRAX 268's limiting speed)** as the conservative upper bound, since the actual prop-side operating RPM under the corrected motor is PROPULSION's own open item (§1 above) — pending AERO's re-check, I'm sizing to the machine's ceiling rather than guessing a lower operating point.

**Everything above except the crash factors is unconfirmed and will re-size the moment the board picks different numbers or AERO publishes Vₐ.** Stated once here; every load case in §3–§6 inherits it.

---

## 3. Load cases (method, then applied per mount in §4–§6)

### 3.1 Steady thrust
Unchanged from v1: reacted axially through the mount into the boom-root structure (motors/prop) or through the pod's forward structure (engine, if firewall-forward — siting still an open item, §9). Cruise thrust = 826 N (AERO-unconfirmed installed-efficiency figure, unchanged chain). Small relative to torque/gyroscopic/crash cases; not governing for any mount.

### 3.2 Torque reaction (engine, generator, prop) — now sized to real per-machine ratings, not a mission-target workaround
**v1 methodology note, now superseded and worth stating why:** v1 sized every electric-machine mount to the *mission power target* rather than the disqualified hardware's low rated ceiling, reasoning that the mount geometry would need to carry mission torque through whichever machine eventually got selected. **That workaround is no longer necessary.** EMRAX 268 genuinely clears the mission floor at its own rated-continuous point (117 kW continuous vs. 96 kW cruise / 155 kW climb combined across two units) — hardware and mission target are now compatible, so I size directly to the machine's own published ratings, which is a stronger evidentiary basis than a target-traced-to-the-architecture-decision figure.

| Reaction point | Steady/continuous torque | Peak/fault torque basis | Limit | Ultimate |
|---|---:|---|---:|---:|
| Engine crank (mount reacts equal/opposite) | 151.5 N·m (111.7 lbf·ft) at ~6,500 RPM, 103.1 kW crank power to deliver 96 kW electrical through gen(96%)+belt(97%) losses | §23.361-style, 4-cyl factor = 2× | 302.9 N·m (223.5 lbf·ft) | **454.4 N·m (335.2 lbf·ft)** |
| Generator (EMRAX 268 MV/CC + BAMOCAR), belt input at ~4,500 RPM | 203.1 N·m at the mission cruise point (real, below the 250 N·m continuous rating — real headroom) | MFR peak torque, T≈446 N·m at limiting speed (§2) | 445.6 N·m (328.7 lbf·ft) | **668.5 N·m (493.0 lbf·ft)** |
| Each propulsion motor (identical EMRAX 268 unit) | 250 N·m continuous rating (real, MFR) — mission climb point runs each motor at ~50% of rated power per PROPULSION's own finding (AER-73 PR#18), real headroom | Same MFR peak torque basis | 445.6 N·m | **668.5 N·m (493.0 lbf·ft)** |

**The generator and both propulsion-motor mounts now share one governing torque-reaction requirement (668.5 N·m ultimate)**, because all three positions carry the identical EMRAX 268 + BAMOCAR unit — a direct, quantified benefit of the parts-commonality finding in §1. This is **not** "nearly 3× the engine mount's ultimate torque" the way v1's Hyper9-based generator case was; it's about 1.5× the engine mount's, a materially smaller spread. **Still flagged:** the 446 N·m peak-torque figure is a first-cut lower bound pending the real torque-speed curve (§2) — treat the "Ultimate" column as provisional, not final, until PROPULSION/CAD source it.

### 3.3 Gyroscopic (prop)
Per-disk polar MOI unchanged from v1: **1.88 kg·m²** (k≈0.4 × 15 kg blade-mass-per-disk × 0.56 m² — EXTRAP, same evidence category as PROPULSION's own prop-assembly mass estimate; this figure is a property of the prop disk, not the motor, so it is unaffected by the electric-machine correction). **Spin rate revised to 4,500 RPM** (EMRAX 268's limiting speed, conservative upper bound — §2). At the placeholder pitch rate (V=155 KTAS, n=3.8g, unchanged): **limit gyroscopic moment ≈ 305.2 N·m (225.1 lbf·ft) per disk, ultimate ≈ 457.8 N·m (337.7 lbf·ft)** — up from v1's 224/336 N·m figures, because the higher assumed spin rate more than offsets nothing else changing; this is the conservative direction and is called out as such.

**Contra-rotation cancellation — same finding as v1, still real, still not universal.** Net gyroscopic moment on shared structure approximately cancels in the symmetric both-motors-running case; it does **not** cancel in the single-motor-inoperative case, which stays the governing combined case for the boom-root fitting:

**Combined boom-root fitting, single-motor-out case (simple sum of torque-reaction ultimate + gyroscopic ultimate, still a conservative bound, not a rigorous multi-axis interaction): 668.5 + 457.8 ≈ 1,126.2 N·m (830.6 lbf·ft)** — down from v1's 1,345 N·m (a 16% reduction), despite the more conservative 4,500 RPM spin-rate assumption, because the torque-reaction term dropped by more than the gyroscopic term rose.

### 3.4 Crash / emergency landing (AER-74-established factors, §2)
Applied as static ultimate inertia loads at each mass's own CG, reacted through its mount into primary structure — methodology unchanged from v1, cert basis now confirmed rather than placeholder (§2). See §6 for the battery-specific numbers.

### 3.5 Vibration / isolation (Hayabusa reciprocating engine)
Unchanged from v1 — the ICE side of the package is untouched by AER-73's electric-machine correction. Firing frequency 40 Hz (idle) to 217 Hz (cruise); target mount natural frequency fn ≤ 13–20 Hz per standard elastomeric aerospace/homebuilt-conversion mount practice. Folded into the mass fraction in §5, same caveats as v1.

---

## 4. Per-mount requirement set

### 4.1 Engine + turbo (105 kg)
Unchanged from v1 — location still not sited (open item, §9, owner: pod-layout/PROPULSION joint). Load cases: steady thrust (reaction only), torque reaction (§3.2, now 454.4 N·m ultimate — the lowest of the three torque-reaction requirements in the package, a change from v1 where it sat in between), crash 9g/6g/3g/1.5g/1.5g (now AER-74-confirmed), vibration isolation (governing for mount topology — soft/isolated, not rigid).

### 4.2 Belt reduction + generator (42.4 kg combined — down from 75 kg)
Shared shaft-line assembly, unchanged treatment from v1. **Alignment tolerance is now for a ~1.44:1 belt drive** (revised from AER-73's own correction to match EMRAX 268's 4,500 RPM limiting speed, versus v1's ~2:1 assumption) — same real requirement, revised ratio. Load cases: torque reaction (§3.2, 668.5 N·m ultimate — now tied with the motor mounts, not the clear outlier it was in v1), crash (AER-74), vibration (secondary, as v1).

### 4.3 2× propulsion motors + prop hub interface (60.8 kg combined — down from 125 kg — at boom root, real CAD location, §1)
**Location unchanged, real, confirmed:** nacelle envelope ⌀686×914 mm at global station X≈3.12–4.04 m, coincident with the pod's own aft structure. Mount design should still tie the motor mount ring into the pod's aft frame/bulkhead, not the free boom tube alone.

**Boom torsion sanity check, revised** (Timoshenko/Bredt thin-wall torsion, same 3 mm 6061-T6 wall placeholder, §1): reacting the revised single-motor-out combined ultimate torque (1,126.2 N·m) through the boom's gross section gives **τ ≈ 2.62 MPa**, against a 6061-T6 ultimate shear allowable of ~207 MPa — **about 1.3% utilization**, essentially unchanged from v1's ~1% finding despite the case itself moving. **The boom tube's gross section remains not the constraint — the local fitting still is**, and is still unsized pending real bracket geometry (§9).

Load cases: steady thrust (both motors), torque reaction per motor (§3.2), **gyroscopic — governing, single-motor-out case (§3.3, 1,126.2 N·m ultimate combined, revised)**, crash (AER-74), vibration (secondary, unchanged reasoning from v1).

**Whirl flutter is not cleared by this analysis** — unchanged from v1, still the required follow-on once real mount stiffness (Kθ, Kψ) is known. The favorable root-mounted location is unaffected by which motor occupies it.

### 4.4 Battery pack (320 kg, unchanged)
Unchanged from v1 in every respect — battery selection, mass, and SAFETY's zoning requirement (AER-62 §4) are untouched by AER-73's electric-machine correction. Load cases: crash (§6, governing, AER-74-confirmed), vibration (low). Station within the pod still unresolved (open item, §9).

---

## 5. First-cut mounting-structure mass estimate

Same fraction-based EXTRAP method as v1, applied to the corrected carried masses:

| Mount group | Carried mass (v2) | Fraction | Structure mass (v2) | (v1, for comparison) |
|---|---:|---:|---:|---:|
| Engine + turbo | 105 kg | 5% | 5.25 kg | 5.3 kg |
| Belt reduction + generator | 42.4 kg | 6% | 2.54 kg | 4.5 kg |
| 2× motors + boom-root bracket | 60.8 kg | 6% | 3.65 kg | 7.5 kg |
| Battery pack | 320 kg | 8% | 25.6 kg | 25.6 kg |
| **Total mounting structure, first cut** | | | **≈ 37.0 kg (81.6 lb)** | 43 kg (94 lb) |

**Still not a member-sized estimate** — a fraction against comparable GA practice, unchanged fidelity caveat from v1. The battery line (unchanged, still the largest single contributor) is the one that most needs real geometry before this stops being a placeholder, per its own crash-critical load path (§6).

---

## 6. Battery crash case — unchanged numbers, upgraded cert basis

At 320 kg (unchanged) and the §2 crash factors — **now AER-74-confirmed, not my own placeholder**:

| Direction | Factor | Ultimate load |
|---|---:|---:|
| Forward | 9.0g | **28,253 N (6,351 lbf)** |
| Downward | 6.0g | 18,835 N (4,234 lbf) |
| Upward | 3.0g | 9,418 N (2,117 lbf) |
| Sideward | 1.5g | 4,709 N (1,059 lbf) |
| Rearward | 1.5g | 4,709 N (1,059 lbf) |

**Forward governs, at 6,351 lbf ultimate — numerically identical to v1, because the battery mass and the crash factors are both unchanged.** What changed is the standing of this number: AER-74 names this exact load path as a confirmed requirement ("confirms STRUCTURES's 6,351 lbf forward case as a requirement, not an assumption") and adds two real design constraints beyond magnitude — (1) within ultimate, the pack must stay attached via a dedicated load path; (2) beyond ultimate, any higher-order mount failure must release the pack *away* from the occupant survivable volume, never toward it, and the post-crash rest position must not breach the HV-bay zoning requirement (AER-62) or block egress. **This is now a real, board-owned fitting-design requirement, not a flagged assumption** — it needs a dedicated crash-load path into the pod's primary structure (strap/frame arrangement analogous to seat-track or gear-fitting practice, Niu's fitting-design method), sized and proof-tested before it is trusted (§8), with the failure-direction constraint designed in from the start, not added after a static-margin pass.

---

## 7. Adversarial findings

Per the issue's mandate: I went looking for a reason this drives unacceptable structural mass or a CG problem — this pass corrects one v1 finding that turned out to be wrong on real data, confirms one that still stands, and states plainly what's unchanged.

### 7.1 — v1's mass-compounding finding was directionally wrong; corrected here

**v1 claimed:** "the gap... moves in the wrong direction regardless of which generator/motor selection eventually resolves §7.1/§7.2 of AER-68 — a heavier, properly-rated electric machine... will add mass on top of this, not reduce it." **This was wrong, and it's worth saying plainly rather than quietly revising the number.** PROPULSION's real, sourced kill-gate finding (AER-73, EMRAX 268) is **91 kg for three machines combined (generator + 2 motors, 30.4 kg each) against the disqualified selection's 188 kg (63 + 125 kg)** — a 97 kg reduction, not an increase. My v1 reasoning inferred "properly-rated aviation-class machines cost more, and expensive usually means heavy" from AER-68's own cost-sanity heuristic; that inference doesn't hold here because EMRAX's higher power density (axial-flux topology, real aviation/EV-industrial pedigree) beats the EV-conversion-class Hyper9 line on mass at a *higher* rated-continuous power, not just a higher price. **Lesson for future passes: don't extrapolate a mass trend from a cost trend without a sourced number — flag the inference as an inference, which v1 did not do clearly enough.**

**Corrected finding:** installed powertrain + mounts, Package A ≈ **671.2 kg (1,479.8 lb), 56.9% of MTOW**, and **279.8 lb (23.3%) over** the trade study's cited 1,200 lb useful-load figure, before crew/payload/fuel — down from v1's 66.2% / 43.5% over. **This is real, quantified improvement, not a disqualifier resolved** — 23.3% over a useful-load figure is still a large gap, and the mounting-structure estimate itself (§5) is still a placeholder fraction, likely optimistic for the battery's crash-critical mount specifically (same caveat as v1). I'm not asserting a formal empty-weight/useful-load violation (still not my ledger, same caveat as v1 and AER-68), but the direction of travel matters: **a corrected machine selection is a real lever on this gap, not a wash and not a worsening.** That's a materially different message to carry into the board's disposition call than v1's.

### 7.2 — The battery crash case remains a real structural driver, now on a confirmed requirement, not a placeholder
Unchanged conclusion from v1, strengthened basis: even setting the electric-machine question aside entirely, the 320 kg battery pack's forward crash load (6,351 lbf ultimate, §6) is a genuine structural sizing problem that exists regardless of which combo survives the board's disposition call — Combo 1 (the Emrax/Rotax fallback, still on the table as an alternative to Package A) carries a comparably large battery reserve per the trade study's own architecture. **This is not a disqualifier by itself** (mass problems here have levers), but AER-74 has now made it a confirmed, named requirement with a failure-direction constraint attached, not a flagged assumption — the pod layout needs to resolve this with an actual crash-load path before detailed pod structure is drawn, and that need has gotten more concrete, not less, since v1.

### 7.3 — The torque-reaction case improved on real data, not just on paper
Worth naming as its own finding, distinct from the mass point in §7.1: v1's generator mount case (using the 3× fault-torque placeholder on the disqualified hardware's higher belt-reduced torque) was **nearly 3× the engine mount's ultimate torque**. The corrected package's real MFR peak-torque figure brings the generator and motor mounts down to **~1.5× the engine mount's ultimate** — a smaller spread, a real vendor number instead of a placeholder multiplier, and (per §1) now a *single* mount-bracket design serving all three positions instead of two different ones. This is a genuine simplification win, not just a smaller number — fewer unique parts, per the buildability principle.

### 7.4 — What did NOT turn out to be a problem, still
Unchanged from v1: the boom-root motor/prop mount location remains structurally favorable (§1, §4.3) — coincident with the pod's own aft structure, not cantilevered on the free boom span. The boom tube's gross torsional capacity remains not a constraint (§4.3, ~1.3% utilization even at the revised, higher combined-case torque). Neither is a green light — the local fitting is still unsized, whirl flutter is still a separate required analysis — but neither is a disqualifier candidate.

---

## 8. Verdict

**Not a structural disqualifier. A real, quantified improvement over v1's own finding on the mass/useful-load gap, with one v1 claim explicitly corrected, plus the battery crash case now resting on a confirmed board requirement instead of a placeholder.**

- The mount *locations and load-case shapes* derived here remain reusable regardless of the board's eventual Package-A-vs-alternatives call — boom-root motor placement, torque-reaction methodology, crash-case treatment, and the AER-74 cert basis all carry over.
- The mount *masses and specific torque numbers* are Package-A-specific (Twin EMRAX 268) and will need re-sizing if the board picks differently — but they are now sized against PROPULSION's actual standing recommendation, not against hardware already known to be disqualified.
- **v1's claim that a corrected machine would worsen the mass picture is retracted (§7.1).** The corrected picture is better, though not resolved: 56.9% of MTOW and 23.3% over the cited useful-load figure remain real numbers the board needs in its disposition call.
- **Escalating to AER-56 (parent) and cross-posting to AER-73**, per standing practice, with §7's corrected findings as the current structural input to the same disposition call.

**Never signing off primary structure on analysis alone** (hard boundary), unchanged from v1: every number above is a first-cut hand analysis. Tests still needed before any of it is trusted: (1) static proof-load test of each mount fitting to its ultimate case, once real geometry exists; (2) ground-run/bench vibration survey of the engine mount before first flight; (3) whirl-flutter analysis once real mount stiffness is known for the boom-root motor mount, ahead of a ground-vibration test; (4) the EMRAX 268 torque-speed curve should be pulled from the real datasheet before the §3.2 peak-torque figure is trusted past first-cut (new this pass, §2).

---

## 9. Open items

1. **Engine + turbo siting** — unchanged open item, owner: PROPULSION/Structures joint or CAD Operator once directed.
2. **Battery pack station within the pod** — unchanged open item; direct input to the CG/convergence loop I own reporting into.
3. **Boom wall thickness/temper** — unchanged open item; still an assumed 3 mm 6061-T6 placeholder.
4. **Real prop assembly MOI** — unchanged open item, still EXTRAP.
5. **EMRAX 268 real torque-speed curve** — **new this pass.** §2/§3.2's 446 N·m peak-torque figure is a first-cut lower bound derived from peak-power/limiting-speed; the real curve shape (constant-torque region vs. power-limited region) could raise this materially. Owner: PROPULSION/CAD, pull EMRAX's actual torque-speed curve rather than the summary table.
6. **AERO's Vₐ** — unchanged open item, needed to replace the non-conservative V=155 KTAS placeholder.
7. **Real mount/bracket geometry** — unchanged open item; no FEM this pass, nothing to mesh yet. Next step once the board picks a package: CAD Operator drafts real mount geometry (potentially a single shared bracket design across all three EMRAX 268 positions, per §1's parts-commonality finding), Structures runs CalculiX, cross-checked against the hand values above.
8. **Whirl flutter** — unchanged open item, still not cleared by this document.
9. **EMRAX 268 motor envelope (⌀268×~114mm)** — **new this pass**, flagged EXTRAP in §1: not independently re-pulled from EMRAX's current datasheet by me or confirmed by PROPULSION this pass. Needed before a bracket can be drawn.
10. **Prop actuator-disk re-sizing against the corrected motor's 4,500 RPM/250 N·m operating point** — **carried from PROPULSION's own open item (AER-73 PR#16 §6 item 4), restated here because it directly affects §3.3's gyroscopic MOI/spin-rate inputs.** Owner: PROPULSION + AERO consult.
11. **Board disposition on AER-56/AER-73** — unchanged in kind from v1, restated: every mass/torque number in this document inherits it. As of this pass, still `todo` / `blocked` respectively.

---

*Analysis by STRUCTURES, MAOS Design Board*
*Version 1 — 2026-08-10. First-cut mount interface, load-case, and mounting-structure-mass analysis for the Combo-3 powertrain per AER-69, sized against the (subsequently disqualified) NetGain Hyper9 selection.*
*Version 2 — 2026-09-04. Re-sized against PROPULSION's corrected-machine kill-gate finding and standing frontier recommendation (AER-73, Twin EMRAX 268, MAOS-ICE PR #16/#18) and SAFETY's now-published crash-load spec (AER-74). Corrects v1's §7.1 claim that a corrected electric machine would worsen the mass/useful-load gap — real vendor data shows the opposite (91 kg vs. 188 kg for the three electric machines combined). Revises the torque-reaction case from an unsourced 3× fault-torque placeholder to a real manufacturer peak-torque figure (flagged as a first-cut lower bound pending the actual torque-speed curve). Crash-case numbers unchanged; cert basis upgraded from placeholder to AER-74-confirmed board requirement. No board disposition has landed on AER-56/AER-73 as of this revision — every number here remains provisional on that call.*
*R&D guidance for Experimental Amateur-Built development. Not a certification claim.*
