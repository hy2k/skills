# Metadata format

One YAML frontmatter block per file-backed artefact. The block carries metadata; the H1 and everything after it carry the record. This file ships with the skills, is normative, and is not a per-repo seed: it states the format, the `agent-conventions` skill writes the surrounding docs.

## Which artefacts carry a block

| Artefact | Block | Keys |
| --- | --- | --- |
| ADR, `docs/adr/NNNN-slug.md` | yes | `status`, `date`, `supersedes`, `superseded_by` |
| Research record, `docs/research/NNNN-slug.md` | yes | `date` |
| Local ticket, `.scratch/<feature>/issues/NN-slug.md` | yes | `type`, `status`, `blocked_by`, `recommended_order`, `subsumed_by` |
| Shipped spec, `docs/specs/<feature>.md` | no | Body only: it mirrors landed reality and is edited to keep mirroring it, so a date or status would go false |
| In-flight spec, `.scratch/<feature>/spec.md` | no | Body only |
| Glossary, handoff note | no | Body only |

A remote tracker keeps its native fields: labels for `status`, issue type or label for `type`, native dependencies for `blocked_by`, and the closed state for a terminal status. An issue body never carries a frontmatter block.

Omit a key that does not apply. Presence means the fact exists; `blocked_by: []` is noise.

## ADR

```markdown
---
status: partially-superseded
date: 2026-11-16
superseded_by: [0019, 0020]
---

# {Decision, stated as the answer}

- 0019 — the goal.
- 0020 — the web-first venue line.
```

| Key | Required | Value |
| --- | --- | --- |
| `status` | yes | `proposed`, `accepted`, `partially-superseded`, `superseded`, `deprecated` |
| `date` | yes | The ISO date the record became `accepted`. A `proposed` record carries its draft date and updates it on acceptance; the date never changes afterwards. |
| `supersedes` | no | List of ADR numbers this record replaced, wholly or in part: `[0007]`. |
| `superseded_by` | with a retired status | List of ADR numbers that took over the answer: `[0019, 0020]`. |

- **The scope of a partial retirement is one bullet per edge, directly under the H1.** Each bullet names the ref and the clause it replaced. A fully `superseded` record needs no bullets: the whole answer moved.
- **A `partially-superseded` record stays editable in the parts no bullet names.** Changing a part a bullet names is a new record, never an edit. A `superseded` or `deprecated` record accepts metadata and typo fixes only.
- **The status transition is the only date edit.** `proposed` to `accepted` rewrites `status` and `date`; every other status change rewrites `status` and the supersession keys alone.
- The gate, the body template, the numbering rules, and when a retirement is a new record live in [`adr-format.md`](../domain-modeling/adr-format.md).

## Local ticket

```markdown
---
type: task
status: ready-for-agent
blocked_by: [05]
recommended_order: [02, 03, 04, 05, 06, 07]
---

# 05: Bring the place online
```

| Key | Required | Value |
| --- | --- | --- |
| `type` | yes | Free text. Rails: `task` (observable behaviour), `bug` (a defect), and the wayfinder kinds `grilling`, `research`, `prototype`. |
| `status` | yes | The triage roles from `triage-labels.md` plus `claimed`, `deferred`, `resolved`, `subsumed` |
| `blocked_by` | no | Refs of the tickets that gate this one. The ticket is unblocked when every ref is `resolved`. |
| `recommended_order` | no | The whole shared-surface sequence, this ticket included, as refs. Position is the ticket's index plus one; the total is the list length. Omit it when the set shares no surface. |
| `subsumed_by` | with `status: subsumed` | The ref of the ticket that absorbed this one. |

A ticket is **open** unless `status` is `deferred`, `resolved`, `subsumed`, or `wontfix`. The frontier is open, unblocked, and unclaimed. `claimed` records a session's claim; `resolved` records delivered work; `subsumed` records work absorbed elsewhere, with `subsumed_by` naming where. Deferral details — who deferred it, when, and until what — live in the body; the status is the only gate.

The H1 keeps the number: `# 05: Title`.

## Research record

```markdown
---
date: 2026-11-09
---

# {What the record answers}
```

`date` is the ISO date the record was written. No status: a research record is evidence, and a later record replaces it by being newer.

## Refs

- Same feature directory: the zero-padded number, `05`.
- Another feature directory: `<feature-slug>/NN`, for example `place-structures/01`.
- Remote tracker: the platform's own reference, `#12`.

Refs apply wherever a key names a ticket or an ADR: `blocked_by`, `subsumed_by`, `recommended_order`, `supersedes`, `superseded_by`.

## YAML style

- `snake_case` keys; arrays as flow lists, `[05, 06]`.
- ISO dates unquoted: `date: 2026-11-16`.
- Refs stay unquoted. YAML reads `05` as `5`, so the zero padding is cosmetic: a script compares numerically, and a qualified ref such as `place-structures/01` stays a string.
- Quote a value only when YAML would misread it: it contains `:` or `#`, starts with `-`, `?`, `[`, `{`, `*`, `&`, `!`, `|`, `>`, `%`, or `@`, or is a bare `true`, `false`, `null`, or number.
- Never repeat a field as a body line. The block is the only metadata.

## Migration

The `agent-conventions` refresh converts every ADR, research record, and local ticket in one commit per repo:

- Conversion is a formatting edit. Values move from body lines and bullets into the block; no claim changes. Retired records convert too, because the block is not a claim.
- Delete the old line or bullet once its value is in the block: `**Status**`, `**Date**`, `Type`, `Blocked by`, `Recommended order`, `Deferred-by`.
- A partial-supersession scope sentence moves to the bullet list under the H1, one bullet per ref.
- A remote tracker needs no migration; reconcile its labels with the status vocabulary and leave the body alone.
