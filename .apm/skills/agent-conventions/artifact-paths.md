# Artifact paths

Where each kind of agent-written file belongs in this repo. Skills read this
file before writing anything; when it is absent they fall back to the defaults
listed here.

| Artefact | Path | Notes |
| --- | --- | --- |
| Research record | `docs/research/NNNN-slug.md` | Numbered sequentially, never renumbered. Every claim carries its source inline. |
| Decision record (ADR) | `docs/adr/NNNN-slug.md` | One current answer per decision. Only a decision with live alternatives earns one. |
| Shipped product spec | `docs/specs/<feature>.md` | Observable behaviour only, no implementation detail. |
| In-flight spec and tickets | `.scratch/<feature>/spec.md`, `.scratch/<feature>/issues/NN-slug.md` | Disposable; deleted once the work lands. |
| Ephemeral artefact | `$TMPDIR`, else `<repo>/.tmp/` | Screenshots, downloads, bakes, one-off scripts. Never committed. |

## Rules

- **An ephemeral artefact never lands in `.scratch/` or `docs/`.** A screenshot, a download, a bake, a one-off script — it has a shelf life, so it goes in `$TMPDIR`. `.scratch/` is committed work despite its name: a spec or a ticket that has to survive the session. Opposite lifetimes, so do not read the directory as this tier.
- **Disposable code is not an ephemeral artefact.** A prototype, a repro harness, or anything else that must run inside the project to answer its question belongs next to what it exercises and is captured on a branch — never in `.tmp/`. `$TMPDIR` is the directory that is never committed; a branch is the one that is. The skill that builds it says which branch.
- **`$TMPDIR` is the scratch root.** Under a sandbox `/tmp` is often write-only, so a file written there cannot be read back — resolve `$TMPDIR` and use what it names, falling back to `<repo>/.tmp/` when it is unset.
- **Research output is not a ticket.** Evidence with sources goes to `docs/research/`; a task goes to `.scratch/`.
- **A shipped spec is not an in-flight spec.** `.scratch/<feature>/spec.md` steers an agent; `docs/specs/<feature>.md` describes what the product must keep doing.
- **`.scratch/` is neither a tracker nor documentation.** When every ticket is resolved, anything product-level moves to `docs/` and the directory is deleted.
- **`docs/` is authoritative; `.scratch/` is disposable.** A fact belongs in `docs/` the moment it stops being a work item.
- **Nothing regenerable is committed.** Downloaded archives, build output, and one-off renders stay out of git.
