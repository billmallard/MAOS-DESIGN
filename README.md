# MAOS-DESIGN

Open-source airframe and aircraft-level design repository for MAOS concepts.

## Status

Concept and architecture phase.

## Safety Notice

This repository is for research and experimental development only. It is not approved for manned flight and must not be used as-is in safety-critical operation without formal system safety assessment, verification, validation, and regulatory compliance.

The project targets the Experimental Amateur-Built category. FAA certification is not a current design constraint, but engineering decisions should still follow conservative aerospace and safety-critical best practices where practical.

## Role in MAOS Multi-Project Architecture

This repository is the aircraft-level design authority.

- Owns geometry, configuration baselines, and cross-subsystem packaging constraints.
- Owns top-level trade studies (weights, aero, structures, integration constraints).
- Publishes system interface requirements consumed by subsystem repositories.

Related repositories:

- MAOS-FCS owns flight control system implementation and verification.
- MAOS-ECS owns environmental control and pressurization subsystem development.
- aerocommons is the website and program-level communications hub.

## Mission

Develop a coherent aircraft design baseline that a determined amateur builder can realistically fabricate, assemble, and maintain while preserving strong safety margins.

Core goals:

- Keep the design buildable with common tools and realistic supplier pathways.
- Drive subsystem integration through explicit interfaces and documented assumptions.
- Maintain transparent trade-space decisions with traceable rationale.
- Evolve the baseline only when supported by analysis and test evidence.

## Scope

In scope:

- Aircraft configuration and geometry baselines.
- Mass properties, packaging envelopes, and center-of-gravity studies.
- Structural concept decisions and manufacturing approach guidance.
- Wing, empennage, landing gear, and pod/boom integration constraints.
- Interface control documents for propulsion, FCS, ECS, and electrical subsystems.
- Design decision records and change rationale.

Out of scope:

- Subsystem firmware implementation details (owned by subsystem repos).
- Standalone subsystem verification campaigns not tied to aircraft-level integration.

## Design Governance

Key working rules:

- One unresolved coupling at a time between major novel subsystems.
- Freeze interfaces before deep implementation in dependent repositories.
- Gate major configuration changes by evidence, not schedule pressure.
- Document failure modes and degraded-operation assumptions explicitly.

Starter template: `INTERFACE_CONTROL_DOCUMENT_TEMPLATE.md`

## Suggested Repository Layout

- docs  Architecture, requirements, and decision records
- cad  CAD models and exchange artifacts
- analysis  Weight, aero, structural, and integration studies
- configs  Baseline configuration sets and controlled parameters
- interfaces  Aircraft-level interface control documents

## Verification and Evidence

- Trace top-level requirements to analyses and test plans.
- Keep assumptions, margins, and confidence levels explicit.
- Record design deltas with rationale and expected downstream impacts.
- Require integration impact notes for changes that affect subsystem interfaces.

## Contribution Expectations

- Propose changes with data, not only preference.
- Make interface impacts explicit in pull requests.
- Keep file formats as open as practical (STEP, DXF, STL, CSV, Markdown).
- Do not claim certification, compliance approval, or airworthiness status.

## Current Milestones (As of 2026-04-14)

Current repository maturity is bootstrap-level (README-only baseline).

Near-term milestones:

1. Establish baseline repository structure (`docs`, `cad`, `analysis`, `configs`, `interfaces`).
2. Publish top-level aircraft configuration baseline and controlled assumptions list.
3. Create initial interface control documents for subsystem boundaries (FCS, ECS, propulsion, electrical).
4. Add first round of weight and packaging trade study artifacts with explicit margin assumptions.
5. Define change-control workflow for configuration updates that affect subsystem repos.

## Knowledge Migration

- Article-derived subsystem migration notes: `docs/ARTICLE_KNOWLEDGE_MIGRATION_2026Q2.md`

## Licensing

This repository uses a dual-license model:

- Source code: PolyForm Noncommercial 1.0.0 (`LICENSE-CODE`)
- Documentation and non-code design content: CC BY-NC-SA 4.0 (`LICENSE-DOCS`)

Commercial use is not granted by default. For commercial licensing, contact `contact@aerocommons.org`.

Contribution and file classification guidance: `CONTRIBUTING.md`

## Program Context

MAOS is an open-source experimental aircraft development effort. The aircraft has not yet flown. All design values are targets subject to revision as analysis and testing mature.