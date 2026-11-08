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
| Throwaway artefact | `$TMPDIR`, else `<repo>/.tmp/` | Screenshots, downloads, bakes, one-off scripts. Never committed. |

## Rules

- **A throwaway never lands in `.scratch/` or `docs/`.** If it has a shelf life, it goes in `.tmp/`; if it has to survive a session, it goes in `.scratch/`.
- **Research output is not a ticket.** Evidence with sources goes to `docs/research/`; a task goes to `.scratch/`.
- **A shipped spec is not an in-flight spec.** `.scratch/<feature>/spec.md` steers an agent; `docs/specs/<feature>.md` describes what the product must keep doing.
- **`.scratch/` is neither a tracker nor documentation.** When every ticket is resolved, anything product-level moves to `docs/` and the directory is deleted.
- **`docs/` is authoritative; `.scratch/` is disposable.** A fact belongs in `docs/` the moment it stops being a work item.
- **Nothing regenerable is committed.** Downloaded archives, build output, and scratch renders stay out of git.
