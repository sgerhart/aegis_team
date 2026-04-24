# Aegis Teams Coordination Workspace

This workspace is a temporary coordination area for Aegis/AegisFlux development and eventual Clarion integration. It is not the long-term product source of truth.

## Source of Truth

- `../aegisflux/` - AegisFlux/Aegis host and workload platform code, product docs, schemas, agent/backend implementation, UI, and test assets.
- `../clarion/` - Clarion platform code, authoritative Clarion architecture, context graph, policy intelligence, network integrations, and customer UI.
- `./` - Cross-project planning, work orders, migration notes, and coordination artifacts while Aegis and Clarion are being brought together.

Durable implementation documentation should move into the owning product repo. Use this workspace for planning until the work has a stable home.

## Current Planning

1. [Aegis Development and Clarion Integration Plan](plans/AEGIS_DEVELOPMENT_AND_CLARION_INTEGRATION_PLAN.md) - Formal plan for Aegis host/workload development, test labs, AI-agent detection validation, and Clarion integration.
2. [Visibility and Observability Work Orders](work_orders/visibility_observability/README.md) - Phase 1 work orders for Windows visibility, backend ingest, detection evidence, and Clarion mapping.

## Current Role of This Workspace

Keep here:

- Cross-repo work orders
- Planning drafts
- Sprint/current coordination
- Open questions and decisions that affect both AegisFlux and Clarion
- Temporary bridge documents before they move to product repos

Move out of here when stable:

- AegisFlux event schemas -> `../aegisflux/schemas/` or `../aegisflux/docs/`
- Aegis agent/backend design docs -> `../aegisflux/docs/`
- Aegis test lab setup -> `../aegisflux/docs/` or `../aegisflux/tests/`
- Clarion context graph or policy docs -> `../clarion/docs/`
- Clarion UI or platform architecture docs -> `../clarion/docs/`

Archive here:

- Historical team notes
- Superseded plans
- Old implementation reports
- Consolidation artifacts that are no longer current

## Active Structure

- `plans/` - Current cross-project plans.
- `work_orders/` - Executable work orders for active development phases.
- `agent/` - Historical or transitional agent documentation.
- `backend/` - Historical or transitional backend documentation.
- `integration/` - Historical or transitional API/integration documentation.
- `archive/` - Old, completed, or superseded documents.
- `CONSOLIDATION_PLAN.md` - Historical documentation consolidation plan.

## Near-Term Direction

The current priority is Aegis visibility and observability:

1. Build the Windows test machine baseline.
2. Define v1 visibility event schemas.
3. Implement Windows process, network, and DNS visibility.
4. Store and query visibility events in AegisFlux.
5. Add non-blocking AI-agent and automation detection evidence.
6. Draft the Clarion mapping for Aegis host/workload events.

No enforcement, SGT changes, or quarantine workflows should be treated as Phase 1 scope.

## Naming and Boundary

AegisFlux is the host/workload part of Clarion. Clarion remains the broader policy intelligence and orchestration platform. AegisFlux should mature as a clean subsystem first; integration should happen through explicit event/API contracts rather than an early repo merge.

## Document Hygiene Rules

- Put durable product docs in the product repo that owns the behavior.
- Keep this workspace focused on coordination and cross-project planning.
- Do not duplicate product documentation here unless it is temporary planning material.
- When a plan becomes implementation-ready, move the durable parts into `aegisflux/` or `clarion/`.
- Archive stale documents rather than keeping them in active navigation.

**Last Updated:** April 24, 2026  
**Status:** Active coordination workspace
