---
status: accepted
date: 2026-11-17
---

# A local ticket declares type, state, and edges in frontmatter

Ticket metadata moves into the frontmatter block: `type` as free text, `status` as one closed vocabulary spanning triage and lifecycle, `blocked_by` and `subsumed_by` as refs, and `recommended_order` as an array whose index gives the position. The vocabulary gains `deferred`, keeps `claimed`, and ends at `resolved` and `subsumed`; a ticket is open unless its status is `deferred`, `resolved`, `subsumed`, or `wontfix`.

## Considered Options

- **Deferral as a separate `deferred_by` field**: two mechanisms for one state, and a frontier check would have to read both. Rejected.
- **Terminal states as prose comments**: a closed ticket stays unreadable to a script, and the directory has to be scanned to learn what remains. Rejected.
- **`recommended_order` as one display string** (`5 of 6 | 02 → 03 → 04`): the position and the chain duplicate each other, and a script splits on `|`. Rejected.
- **Closed `type` enum**: `grilling` and `task` already differ per repo, and a new kind would edit the format. Free text with example rails. Chosen.

## Consequences

- The frontier is open, unblocked, and unclaimed; `wontfix` and `subsumed` close a ticket without claiming delivery.
- `subsumed_by` mirrors the ADR's `superseded_by` and may name another feature directory, such as `place-structures/01`.
- Deferral details — owner, date, and the condition — live in the body; the status is the only gate.
- A remote tracker uses labels and close states; `resolved` and `subsumed` become close reasons there.
- The local ticket template and the local tracker seed change in the wiring commit.
