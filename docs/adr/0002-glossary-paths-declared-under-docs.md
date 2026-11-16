---
status: accepted
date: 2026-11-16
---

# Glossary paths are declared in `docs/agents/domain.md`, with defaults under `docs/`

Where a repo keeps its glossary, and how a skill finds it, is declared once in the Paths block of `docs/agents/domain.md`: `docs/glossary.md` for a single glossary, `docs/glossary-map.md` for a multi-context map, `src/<context>/docs/glossary.md` for a context glossary, and the matching `docs/adr/` paths for decisions. A declared path wins; a declared path with no file means no glossary, because creation is lazy. Without the declaration, a skill reads `docs/glossary-map.md`, then `docs/glossary.md`, then proceeds silently. Root `GLOSSARY.md` and `GLOSSARY-MAP.md` are migration inputs only, moved by setup and never probed at runtime.

## Considered Options

- **Keep root defaults with a declared override.** Rejected: root documentation competes with `README.md` and the agent file, and two default layouts keep the probe ambiguous.
- **Declare the path with no default.** Rejected: a repo that installs the skills before running setup would have no glossary lookup at all.
- **Declare the path and keep a root fallback.** Rejected: a stale declaration could silently read a file the repo did not name, which is the failure the declaration exists to remove.
- **Declared paths with `docs/` defaults.** Chosen.

## Consequences

- A repo that never runs setup and keeps only a root glossary sees no glossary until setup migrates it.
- Setup moves root files with `git mv` and rewrites references in the README, `docs/**`, and the agent file; code references are reported, not edited.
- Context glossaries follow context ADRs under `src/<context>/docs/`, so a context keeps one documentation directory.
