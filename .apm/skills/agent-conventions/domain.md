# Domain Docs

How the engineering skills should consume this repo's domain documentation when exploring the codebase.

## Paths

- **Glossary**: `docs/glossary.md`
- **Glossary map**: `docs/glossary-map.md` (multi-context repos only; omit this line in a single-context repo)
- **Decision records (ADRs)**: `docs/adr/`
- **Context glossary**: `src/<context>/docs/glossary.md`
- **Context ADRs**: `src/<context>/docs/adr/`

This block is the declaration. Read it before probing a default. A declared path with no file means no glossary: the file is created lazily when the first term is resolved, not filled in ahead of need.

## Before exploring, read these

- **The glossary** at the declared path. If a **glossary map** is declared, read it to find the contexts, then read each glossary relevant to the topic.
- **The ADRs** in `docs/adr/` that touch the area you're about to work in. In multi-context repos, also check `src/<context>/docs/adr/` for context-scoped decisions.

If no glossary exists, **proceed silently**. Don't flag its absence; don't suggest creating one upfront. The `domain-modeling` skill (reached via `grilling` and `improve-codebase-architecture`) creates it lazily when terms or decisions actually get resolved.

## File structure

Single-context repo (most repos):

```
/
├── docs/
│   ├── glossary.md
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

Multi-context repo (declared in the Paths block):

```
/
├── docs/
│   ├── glossary-map.md
│   └── adr/                          ← system-wide decisions
└── src/
    ├── ordering/
    │   ├── docs/
    │   │   ├── glossary.md
    │   │   └── adr/                  ← context-specific decisions
    │   └── ...
    └── billing/
        ├── docs/
        │   ├── glossary.md
        │   └── adr/
        └── ...
```

## Use the glossary's vocabulary

When your output names a domain concept (in an issue title, a refactor proposal, a hypothesis, a test name), use the term as defined in the glossary. Don't drift to synonyms the glossary explicitly avoids.

If the concept you need isn't in the glossary yet, that's a signal: either you're inventing language the project doesn't use (reconsider) or there's a real gap (note it for `/domain-modeling`).

## Flag ADR conflicts

If your output contradicts an existing ADR, surface it explicitly rather than silently overriding:

> _Contradicts ADR-0007 (event-sourced orders), but worth reopening because…_

Put the conflict and the amendment you propose to the owner, and wait for that decision before writing code. A record that steers the code and is then silently diverged from is worse than no record: the next session reads the false version as direction.

## Docs are the record; the agent file is the index

- **`docs/` is authoritative.** `README.md` covers what a human needs to build and run; `docs/` covers what the product must do and why.
- **The agent file is an index, not a copy.** It names where each rule lives and states only what has no other home. A rule written in both places drifts; when they disagree, the doc wins.
- **Update the doc in the same change as the behaviour.** A change that leaves a doc describing the old state is unfinished, not deferred.
- **Grep for what the change renames.** A term the change removes or renames is still named in the docs; update every hit, scoped to the docs the change touches.
- **A spec is a contract, not a mirror.** Rules and observable behaviour belong in it; exact values, file paths and internal structure belong to the code. If a sentence would need editing when a dependency, file or default changes, it is not spec text.
