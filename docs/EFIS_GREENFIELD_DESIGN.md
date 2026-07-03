# EFIS Greenfield Design — "if we started pyEfis over from nothing"

Status: DRAFT v1 (2026-07-03, Bill + Claude). A thought experiment with teeth:
every prescription below traces to a scar we actually have. This is not a
plan to rewrite pyEfis tomorrow — §10 is honest about migration — but it is
the north star the retrofits should converge toward.

---

## 0. The evidence base (why each lesson is a lesson)

This document was drafted at the end of a period that included: the
instrument-registry migration (all 27 types), the standards-grounded
verification workstream (FAA-cited specs + xfail gap catalogs), the
configurator twin-fidelity grind (bat wings, fonts, opacity, trend tape),
the SVS morphing campaign (phase-anchor, morph band, Track 1b per-level
textures, the ridge sawtooth), the quoted-string config segfault and boot
crash-loop, the netfix chunked-list bug, the NAVSRC "invisible mode" incident,
the open-GCU design, and years of accumulated config-layering archaeology.
Every one of those is a design input here.

## 1. What we would keep (the load-bearing ideas)

1. **The hub-and-spoke bus.** Every box talks through a flight-data bus,
   never directly. This is the single best decision in the current stack —
   it made the Octavi bridge a 60-line bench toy and makes the GCU possible.
2. **Declarative screens.** YAML screens on a normalized grid, built by a
   screenbuilder. The *idea* is right; the execution (raw setattr, config
   layering) is what hurt.
3. **The fail/old/bad annunciation convention.** Never show invalid data as
   live. In the new design it stops being a convention and becomes an
   enforced contract (§4).
4. **The makerplane-data model.** Signed packs, manifest as the contract,
   atomic swap, currency tracking. We extend this pattern to *instruments
   themselves* (§3).
5. **Python at the edges.** Approachable for the experimental-aviation
   community. The greenfield changes *what* you write in Python (providers,
   not paint code), not whether you write Python.

## 2. The core inversion: instruments as data, not code

**The single biggest tax in the current design is that every instrument's
appearance is written twice** — once in QPainter for the device, once in
JS/SVG for the configurator — and kept aligned by hand under a fidelity
rule. Weeks of twin-matching (aircraft symbols, fonts, tick widths, opacity)
are the direct cost. A greenfield removes the possibility of drift:

> An instrument's rendering is a **declarative scene definition** — typed
> primitives (arc, tick ring, tape, needle, text, polygon, band), data
> bindings, and state rules — interpreted by ONE renderer that compiles to
> the device (GL) and to the browser (WebGL/canvas or wasm) from the same
> source.

- The device panel and the configurator preview become *the same picture by
  construction*. The fidelity rule dissolves.
- The manual/wiki screenshots, palette icons, and editor previews are all
  generated artifacts of the definition.
- 90% of instruments (gauges, tapes, dials, HSI, annunciators) fit the
  declarative model. The other 10% (SVS terrain, moving map) are **native
  layer providers** (§5) with an explicit escape hatch, not a back door.

The declarative definition carries, per element, the *why*: an optional
`req:` tag citing the standards requirement it satisfies (VSI-DISP-001
style). Verification tooling reads it (§7).

## 3. Packaging: the provider model (Bill's instinct, refined)

Each instrument is an **instrument pack** — an independently versioned unit
with a manifest:

```yaml
# manifest.yaml (illustrative)
id: org.makerplane.vsi-trend-tape
version: 2.1.0
api: efis-pack/1
provides: instrument
consumes:                    # bus subscriptions, with quality expectations
  - key: VS
    required: true
produces: []                 # e.g. a baro-set instrument produces BARO
properties:                  # TYPED, ranged, labelled — the editor schema IS this
  maxvs:   {type: int, default: 2500, min: 500, max: 10000, unit: ft/min}
  bg_opacity: {type: percent, default: 100}
scene: scene.yaml            # the declarative rendering
previews: [{VS: 500}, {VS: -1200}]
requirements: requirements.yaml   # FAA-cited spec rows + status
tests: catalog/                   # conformance + golden renders
```

- **Distribution = the makerplane-data pattern**: packs are built, signed,
  and pulled exactly like navdata. The configurator palette and the device
  install from the same signed pack registry. Community instruments become
  possible without forking the core — the MakerPlane "instrument store."
- **Repo-per-instrument: yes for *packaging*, cautious for *source*.**
  Independent versioning, tests, specs, and releases per instrument: yes,
  that is the provider model's point. But literal one-repo-per-instrument
  has real costs we'd feel immediately — this session's *font fix touched
  ten instruments in one commit*; cross-cutting changes (a new quality flag,
  a palette convention) become 30-PR days, and the version compatibility
  matrix becomes its own job. Recommendation: **a monorepo of packs**
  (or 3–4 grouped repos: core-flight, engine, nav, experimental) where each
  pack is independently versioned and released, plus solo repos for
  genuinely independent/community instruments. The *contract* (manifest,
  pack API) is what guarantees decoupling — not repo boundaries.

## 4. The bus, redesigned (fdx — flight data exchange)

Keep the philosophy; fix the wire and the semantics. Scars: the netfix
chunked-list bug, semicolons-in-descriptions corrupting the protocol, the
NAVSRC invisible-mode incident, no set-point persistence (#84), two writers
fighting (xplane vs stratux), quality flags as ad-hoc strings.

1. **Typed, versioned wire protocol** (CBOR or protobuf; length-prefixed
   frames — never delimiter-parsed text). Schema registry with semver;
   clients negotiate. A field can never corrupt the framing.
2. **Quality is structural**: every value carries {source, timestamp,
   quality: OK|OLD|BAD|FAIL, ttl}. The renderer *cannot* draw a value
   without its quality — the fail/old/bad convention becomes type-enforced.
3. **Source arbitration in the bus**: keys declare an ownership policy
   (single-writer, priority list, or last-writer) and the bus enforces it,
   annunciates active source, and exposes it as data (the HSI shows *which*
   GPS is feeding it). SIM/FLIGHT interlock is a bus mode, not a convention.
4. **Persistence classes**: volatile | journaled (set-points: BARO, bugs,
   NAVSRC — survive restart, #84 never happens) | config.
5. **Record/replay as a bus feature**: every session is a flight-data
   recording by default (ring buffer + save-on-event). Replay drives the
   full panel deterministically — debugging, regression tests, and Auspex
   oracles all feed on this. (The frame clock is already decoupled in
   current pyEfis — P4; greenfield keeps time explicit everywhere: bus
   time vs wall time vs sim time.)
6. **CAN-FIX bridge** as a first-class provider, so the hardware ecosystem
   (knob heads, engine monitors) is unchanged.

## 5. The host: compositor + supervised providers

Scars: one bad config string segfaulted the whole panel into a crash loop;
`self.svs` clobbered by an options dict; paint-order hacks; a broken
SIGUSR1 screenshot that blanks the live GL; GL that can't render offscreen
so *capturing a reference image means stopping the aircraft's display*.

1. **A thin compositor host** owns the screen: layers with explicit z-order,
   damage tracking, and a GL context it controls. Instruments render into
   layers — declarative scenes via the built-in renderer, native providers
   (SVS, map) into their own layer textures with a narrow API (bind, render
   w/ pose, resize). No widget can clobber another's state because there is
   no shared attribute soup.
2. **Supervised isolation**: providers run in-process by default (fast) but
   behind a fault boundary; a crashing provider degrades to a red-X layer,
   not a dead panel. The bus daemon, host, and heavy providers are separate
   processes under a supervisor.
3. **Config transactions with rollback**: a new screen config is applied as
   a transaction; if the panel fails to reach "first healthy frame" the host
   auto-reverts to last-known-good and annunciates. The v58 crash-loop
   becomes a 5-second self-healing event. A minimal built-in "safe screen"
   (attitude + altitude + airspeed from the declarative core) can always
   render.
4. **Headless by design**: the same host renders offscreen (EGL surfaceless)
   for CI, golden images, manual screenshots, and the configurator's
   server-side previews. The screenshot service is a bus command
   (`panel.screenshot` → file + notification) — the GCU gets a screenshot
   button for free, and nobody ever again stops a running panel to capture
   its picture.
5. **One config store** with layering that is *introspectable*: every
   effective value answers "where did I come from" (file, layer, device
   push, default). The preferences.yaml/.custom/managed_*/mtime-sentinel
   archaeology — a full session was once spent proving which file was live —
   is replaced by `efis config explain screens.PFD.instruments[3].maxvs`.

## 6. Properties and schemas: one source of truth

Scars: raw setattr from YAML (quoted "5" segfaulting pitch-ladder geometry),
the editor schema as a hand-curated retrofit, option-application order vs
scene-build order mysteries.

- The manifest's `properties` block IS the schema: typed, ranged, defaulted,
  labelled, versioned. The host validates and coerces at load; an invalid
  value is a rejected config transaction with a precise error, never a
  segfault at paint time.
- The configurator generates its editor UI from the manifest. The device
  and editor cannot disagree about what an instrument accepts.
- Property changes are events; instruments react through bindings — no
  resize-order dependency (the heading-bug scene-build lesson).

## 7. Verification as a first-class artifact (the Auspex tie-in)

The current standards workstream (FAA corpus → spec IDs → xfail catalogs →
implement → visual verify) was bolted on years in — and it works. Greenfield
bakes it into the pack format:

- `requirements.yaml` per pack: requirement rows with citations
  (AC 23.1311-1C §8.11 p.24 …), status, and links to tests. The
  "certifiable-quality, not certified" posture becomes machine-readable.
- **Conformance kit** every pack must pass in CI: schema fuzzing (every
  property at min/max/wrong-type — the quoted-string class of bug dies
  here), quality-flag behavior (feed OLD/BAD/FAIL, assert annunciation),
  golden renders at declared preview poses (headless host), and the
  standards catalog with xfail gap tracking.
- Record/replay (§4.5) + golden renders make Auspex-style oracles natural:
  scenario in, rendered frames + bus traces out, rubric-checked.

## 8. The configurator's new job

Mostly *less* work: no twins to maintain (same renderer), no schema export
pipeline (manifests are the schema), palette/preview assets generated. Its
real jobs become: layout UX, pack registry browsing/install, device pairing
and signed config push (already built), and fleet/version management
(which device runs which pack versions). The SVS preview keeps the
data-driven approach we just built — a native provider compiled for the
web is the eventual same-pixels answer.

## 9. Language and runtime (pragmatics, not fashion)

- **Bus daemon + host core**: a compiled runtime pays off here (Rust is the
  obvious candidate: the GIL fights behind #74, the collector stalls, and
  the render-thread discipline all become non-events; wasm compilation of
  the scene renderer gives the browser the literal same code). This is the
  one place a rewrite buys structural safety.
- **Providers and tooling**: Python SDK first-class. The community writes
  Python (or anything that speaks the bus protocol — it's just CBOR frames).
- **Declarative scenes**: the format matters more than the engine. Start as
  YAML/JSON with a strict schema; treat "compile to QML/Skia/WebGL" as
  renderer backends.
- If a full compiled core is too big a bite: the same architecture works
  with a Python host + Qt scene graph, accepting the GIL discipline we've
  already learned. The architecture is the point; the language is a dial.

## 10. Honest migration (because we won't actually start from nothing)

The good news: the last two months of retrofits ARE the strangler pattern
for this design, already in motion.

| Greenfield piece | Existing seed |
|---|---|
| Manifest/schema-first properties | instrument registry + Props + editor schema exporter |
| Conformance kit | verification specs + xfail catalogs + render_instrument |
| Signed pack distribution | makerplane-data packtools + device pull |
| Bus quality/persistence/arbitration | fix-gateway (needs the fdx upgrades, incremental) |
| Compositor/native-layer split | the SVS layer inside AI (needs extraction) |
| Same-pixels configurator | data-driven SVS preview (first instance of the idea) |
| Control-head providers | open-GCU design doc + Octavi bridge |

A plausible sequence without a big bang: (1) fdx wire upgrade inside
fix-gateway (typed frames, quality struct, journaled keys) behind a compat
netfix shim; (2) define the pack manifest and make the registry emit it
(near-zero delta from today's schema); (3) build the declarative scene
renderer for ONE instrument (the trend tape is small and freshly specced),
render it on-device and in the configurator from the same file — that's the
proof that kills the twin problem; (4) conformance kit around it; (5)
migrate instruments opportunistically, exactly like the registry migration
went; (6) extract the compositor last, when enough instruments are
declarative that the host is thin.

## 11. Governance and licensing footnote

Packs invite community contribution; the GPLv2 core + MIT pack-SDK split
(mirroring pyEfis-GPL / makerplane-data-MIT today) keeps instrument authors
unencumbered while protecting the core. The pack registry needs a signing
policy (who may sign what) — the makerplane-data key model extends.

---

*Filed as the north star. The next concrete act that serves it: when the
trend tape (or any small instrument) next needs real work, write it as a
declarative scene + manifest and render that from both hosts — one
afternoon, and the thesis is tested on live hardware.*
