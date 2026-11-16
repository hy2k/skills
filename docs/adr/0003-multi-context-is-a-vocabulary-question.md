---
status: accepted
date: 2026-11-16
---

# Multi-context is a vocabulary question, not a package-shape signal

A repo is multi-context when a term means different things in different areas, so setup asks one question on the first run, and only when `docs/agents/domain.md` is absent: "Does any term mean different things in different areas?" A yes collects the context names and glossary paths in one follow-up, writes `docs/glossary-map.md`, and creates each `src/<context>/docs/glossary.md` lazily; a no writes the single-context layout. Package count, workspace files, and runtime count are not the test: one model can span several runtimes, and several packages can share one vocabulary.

## Considered Options

- **Monorepo signals (`pnpm-workspace.yaml`, `workspaces`, `packages/*`).** Rejected: the signal describes directory shape, not vocabulary, and misreads a multi-runtime single-context repo.
- **Conflict evidence only: scan existing glossaries for one term with two definitions.** Rejected: the check cannot run before the first glossary exists, which is exactly when the layout is chosen.
- **Single-context only; multi-context on explicit request.** Rejected: the option stays invisible to most owners.
- **One vocabulary question at first setup.** Chosen.

## Consequences

- Setup asks one extra question, once.
- A repo with several runtimes and one glossary stays single-context.
- Refresh does not re-ask; changing the layout is an explicit setup invocation.
