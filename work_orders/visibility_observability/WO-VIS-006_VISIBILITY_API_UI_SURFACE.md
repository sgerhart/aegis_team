# WO-VIS-006: Visibility API and UI Surface

**Status:** Draft  
**Phase:** Visibility and Observability  
**Primary owner:** Backend / UI  

## Goal

Expose the first useful investigation workflow for Aegis visibility data.

## Scope

API and UI/API surface only. This work order assumes events are already ingested and stored.

## Investigation Views

Minimum API or UI should answer:

- What processes ran on this device?
- What process created this outbound connection?
- What parent process launched this process?
- What user/session owned this process?
- What DNS/domain context is associated with this flow?
- Was this process classified as browser, IDE helper, script, local agent, automation, or unknown?
- What evidence supports the classification?

## Deliverables

- Device visibility endpoint
- Process detail endpoint
- Flow detail endpoint
- Detection/risk finding endpoint
- Timeline query by device and time range
- Minimal UI or API output for investigation path
- Example investigation for `cursor.exe -> python.exe -> model/API destination`

## Acceptance Criteria

- User can query a Windows device and see recent processes.
- User can select a process and see parent, command line, user, and outbound flows.
- User can select a flow and see destination/DNS context.
- User can see early AI/automation detection evidence when present.
- The surface is read-only and cannot apply enforcement.

## Dependencies

- WO-VIS-005
- WO-VIS-007 for detection evidence display

## Risks

- Existing UI may assume current Aegis policy/agent objects only. Keep first surface simple.
- Avoid building a polished investigation UI before the data model stabilizes.

## Completion Evidence

- API examples or screenshots
- Recorded investigation path using lab scenario data
