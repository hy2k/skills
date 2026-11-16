---
status: accepted
date: 2026-11-16
---

# ADR numbers are append-only; a record holds one question

`docs/adr/` holds one current answer per question, and an accepted record is never reopened: a changed answer is a new record that supersedes the old one, a number is never reused and a block is never renumbered. Recording the same question twice is the defect this rule removes, and the retirements stay readable against the code of their day. The only rename allowed repairs a duplicate number, and it rewrites every reference in the same change.

## Considered Options

- **Collapse repair**: merge the losers, delete them, renumber the block once. Rejected: correctness then depends on remembering to run a repair, and deletion erases the only statement of why the old answer looked right.
- **Current-answer wording**: `artifact-paths.md` said "one current answer per decision", and `adr-format.md` left reuse and renumbering unstated — read together, a tidy block may be renumbered and git holds the history. Rejected: git history is not reachable from a link, and every renumber is a repository-wide link edit.
- **Strict append-only**: chosen.

## Consequences

- Superseded records stay on disk, so the directory grows with dead answers.
- Review-driven tuning of a constant produces a research record, not an ADR.
- Linking is one-directional at authoring time and bidirectional after retirement.
