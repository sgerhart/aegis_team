# WO-VIS-005: Backend Ingest and Storage

**Status:** Draft  
**Phase:** Visibility and Observability  
**Primary owner:** Backend  

## Goal

Allow the Aegis backend to receive, validate, store, and query visibility events from the Windows sensor.

## Scope

This work order covers ingest and persistence for visibility events. It does not include policy decisions or enforcement.

## Deliverables

- Ingest endpoint or message bus consumer for v1 visibility events
- Schema validation
- Durable storage for:
  - agent heartbeat
  - process events
  - flow events
  - DNS observations
  - detection events
  - risk findings
- Query model for investigation:
  - by device
  - by process
  - by flow/destination
  - by time range
  - by detection/risk finding
- Basic deduplication by `event_id`
- Ingest health metrics
- Error/dead-letter handling for invalid events

## Acceptance Criteria

- Backend accepts valid v1 visibility events.
- Backend rejects or quarantines invalid events with a useful error.
- Re-sending the same event does not create duplicate investigation records.
- Query can reconstruct a simple path:
  - device -> user -> parent process -> process -> destination -> DNS/domain -> detection
- Storage can handle the first lab machine event volume without manual cleanup.

## Dependencies

- WO-VIS-004

## Risks

- Existing backend services may have stale contracts or port inconsistencies. Confirm current service boundaries before implementation.
- Storing every raw event without retention policy can grow quickly. Define lab retention defaults.

## Completion Evidence

- Ingest API or consumer documented
- Example curl or producer command
- Stored event query examples
- Invalid event handling example
