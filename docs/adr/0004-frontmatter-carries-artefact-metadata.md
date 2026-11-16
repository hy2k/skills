---
status: accepted
date: 2026-11-17
---

# One YAML frontmatter block carries metadata for every file-backed artefact

Frontmatter at the top of each ADR, research record, and local ticket, with keys typed per artefact; the body carries only the record. A rendered preview then shows a metadata table instead of bold lines or bullets, and a script reads a field without parsing prose. Shipped specs stay body-only: they mirror landed reality and are edited to keep mirroring it, so a status or date would go false. A remote tracker keeps its native fields.

## Considered Options

- **Bold inline lines under the H1** (what `ADR-FORMAT.md` prescribed): readable, but no field boundary reaches the render and each field competes with the prose. Rejected.
- **Markdown bullet lists** (what local tickets prescribed): preview-safe, yet every field renders as prose styling and a script still parses Markdown. Rejected.
- **One form chosen per repo**: consistent inside a repo and drifting between the two repos already on this skill set. Rejected.
- **One frontmatter block everywhere it applies**: chosen.

## Consequences

- `agent-conventions/metadata-format.md` is the normative key list, status vocabulary, and style; the ADR and ticket format docs point at it.
- `adr-format.md` drops the inline `**Status**` template, keeps "never in a record" applying to body prose rather than the block, and its status-transition rule gains the `date` edit.
- `ADR-FORMAT.md` and `GLOSSARY-FORMAT.md` become `adr-format.md` and `glossary-format.md`; lowercase is the preference, and uppercase stays a preference for widely recognized filenames such as `README.md`, `AGENTS.md`, `SKILL.md`, and `CHANGELOG.md`.
- Each repo converts its ADRs, research records, and tickets in one commit on the next `agent-conventions` refresh. Conversion moves values and changes no claim, so retired records convert too.
- No skill parses frontmatter today; the schema stays YAML-parseable so a check can be added later without a second migration.
- This record is the first in the new block; ADR-0001 to ADR-0003 convert with the migration.
