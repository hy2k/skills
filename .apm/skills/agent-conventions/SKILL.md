---
name: agent-conventions
description: Use when a repo needs its agent conventions pinned down before the other engineering skills run — where research, decision records, product specs, in-flight tickets, and throwaway files belong; where issues are tracked; the triage label vocabulary; and the domain doc layout. Writes the per-repo docs the other skills read.
disable-model-invocation: true
---

# Agent conventions

Scaffold the per-repo configuration the engineering skills assume:

- **Artifact paths**: where each kind of file an agent writes actually belongs
- **Issue tracker**: where issues live (GitHub by default; local markdown is also supported out of the box)
- **Triage labels**: the strings used for the five canonical triage roles
- **Domain docs**: where the glossary and ADRs live, and the consumer rules for reading them

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write.

Every one of these has a **default**. Take the default without asking unless exploration found evidence the repo already decided otherwise. The point of this skill is to write the file, not to run a questionnaire.

## Process

### 1. Explore

Look at the current repo to understand its starting state. Read whatever exists; don't assume:

- `git remote -v` and `.git/config`: is this a GitHub repo? Which one?
- `AGENTS.md` and `CLAUDE.md` at the repo root: does either exist? Is there already an `## Agent skills` section in either?
- `docs/agents/`: does this skill's prior output already exist?
- `docs/research/`, `docs/adr/`, `docs/specs/`, `docs/glossary.md`, `GLOSSARY.md`, `GLOSSARY-MAP.md`: which of the documented layouts is already in use?
- `.scratch/`, `.tmp/`, `TMPDIR` in the environment: signs of an existing local-markdown tracker and a throwaway convention
- Is the `triage` skill installed? (a `triage` skill folder alongside this one, or `triage` in your available skills.) This decides whether the triage section runs at all.
- Monorepo signals: a `pnpm-workspace.yaml`, a `workspaces` field in `package.json`, or a populated `packages/*` with its own `src/`. These are present only in a genuinely large multi-package repo; their absence means single-context, which is almost every repo.

### 2. Present findings and ask

Summarise what's present and what's missing. Then take the sections in order. One section, one answer, then the next. Skip any section exploration already settled.

Lead each section with the recommended answer so the user can accept it in a word. Give a one-line explainer only when the choice genuinely branches.

**Section A: Artifact paths.**

> Explainer: an agent that writes a file has to pick a directory, and picking wrong is how research notes end up in the ticket folder or screenshots end up committed. This file is the one place that answers it.

Default posture: adopt the five-tier layout below verbatim. Offer a deviation only when exploration found the repo already uses different paths.

| Artefact | Default path |
| --- | --- |
| Research record | `docs/research/NNNN-slug.md` |
| Decision record | `docs/adr/NNNN-slug.md` |
| Shipped product spec | `docs/specs/<feature>.md` |
| In-flight spec and tickets | `.scratch/<feature>/spec.md`, `.scratch/<feature>/issues/NN-slug.md` |
| Throwaway artefact | `$TMPDIR`, else `<repo>/.tmp/` |

Write the agreed layout to `docs/agents/artifact-paths.md` using [artifact-paths.md](./artifact-paths.md) as the seed.

**Section B: Issue tracker.**

> Explainer: the "issue tracker" is where issues live for this repo. Skills like `to-tickets`, `triage`, and `to-spec` read from and write to it. They need to know whether to call `gh issue create`, write a markdown file under `.scratch/`, or follow some other workflow you describe. Pick the place you actually track work for this repo.

Default posture: these skills were designed for GitHub. If a `git remote` points at GitHub, propose that. If a `git remote` points at GitLab (`gitlab.com` or a self-hosted host), propose GitLab. Otherwise (or if the user prefers), offer:

- **GitHub**: issues live in the repo's GitHub Issues (uses the `gh` CLI)
- **GitLab**: issues live in the repo's GitLab Issues (uses the [`glab`](https://gitlab.com/gitlab-org/cli) CLI)
- **Local markdown**: issues live as files under `.scratch/<feature>/` in this repo (good for solo projects or repos without a remote)
- **Other** (Jira, Linear, etc.): ask the user to describe the workflow in one paragraph; the skill will record it as freeform prose

Record the choice in `docs/agents/issue-tracker.md`. The GitHub and GitLab templates carry a "PRs as a request surface" flag, defaulted **off**. Leave it off and don't raise it: a user who wants external PRs in the triage queue can flip the flag in the file later.

**Section C: Triage label vocabulary.** Skip this section entirely if the `triage` skill isn't installed (exploration told you), since an uninstalled skill needs no labels.

If it is installed, ask exactly one question:

> Do you want to keep the default triage labels? (recommended: **yes**)

The defaults are the five canonical roles, each label string equal to its name: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. On **yes**, write them as-is. Only if the user says no, usually because their tracker already uses other names (e.g. `bug:triage` for `needs-triage`), collect the overrides so `triage` applies existing labels instead of creating duplicates.

**Section D: Domain docs.** Default to **single-context** (one `GLOSSARY.md` + `docs/adr/` at the repo root). This fits almost every repo; write it without asking.

Offer **multi-context** (a root `GLOSSARY-MAP.md` pointing to per-context `GLOSSARY.md` files) only when exploration found monorepo signals. Then confirm which layout they want.

**Section E: Question channel.**

> Explainer: the skills that interview — `grilling` above all — need to know which structured question tool this repo's harness exposes, because every harness names the fields differently. Prose rounds cost a round trip and lose the selection; a round in the wrong shape fails outright, since these schemas are closed.

Default posture: write the row for the harness **this session** is running, which you can read off your own tool list — `ask`, `question`, `ask_user_question`, `AskUserQuestion`, or none. Add a row for each further harness the repo is driven by, if the user names one. Do not ask which harness it is when your own tool list already answers it.

Write it to `docs/agents/asking.md` using [asking.md](./asking.md) as the seed.

### 3. Confirm and edit

Show the user a draft of:

- The `## Agent skills` block to add to whichever of `CLAUDE.md` / `AGENTS.md` is being edited (see step 4 for selection rules)
- The contents of `docs/agents/artifact-paths.md`, `docs/agents/asking.md`, `docs/agents/issue-tracker.md`, `docs/agents/domain.md`, and `docs/agents/triage-labels.md` (the last only when `triage` is installed)

Let them edit before writing.

### 4. Write

**Pick the file to edit:**

- If `CLAUDE.md` exists, edit it.
- Else if `AGENTS.md` exists, edit it.
- If neither exists, ask the user which one to create; don't pick for them.

Never create `AGENTS.md` when `CLAUDE.md` already exists (or vice versa); always edit the one that's already there.

If an `## Agent skills` block already exists in the chosen file, update its contents in-place rather than appending a duplicate. Don't overwrite user edits to the surrounding sections.

The block:

```markdown
## Agent skills

### Artifact paths

[one-line summary of where research, decisions, specs, tickets, and throwaways go]. See `docs/agents/artifact-paths.md`.

### Asking

[one-line summary: the question tool this harness exposes and how it marks a recommendation]. See `docs/agents/asking.md`.

### Issue tracker

[one-line summary of where issues are tracked]. See `docs/agents/issue-tracker.md`.

### Triage labels

[one-line summary of the label vocabulary]. See `docs/agents/triage-labels.md`.

### Domain docs

[one-line summary of layout: "single-context" or "multi-context"]. See `docs/agents/domain.md`.
```

Include the `### Triage labels` sub-block, and write `docs/agents/triage-labels.md`, only when `triage` is installed and Section C ran. When it isn't, both are omitted.

Then write the docs files using the seed templates in this skill folder as a starting point:

- [artifact-paths.md](./artifact-paths.md): the five-tier layout
- [asking.md](./asking.md): the question channel, per harness
- [issue-tracker-github.md](./issue-tracker-github.md): GitHub issue tracker
- [issue-tracker-gitlab.md](./issue-tracker-gitlab.md): GitLab issue tracker
- [issue-tracker-local.md](./issue-tracker-local.md): local-markdown issue tracker
- [triage-labels.md](./triage-labels.md): label mapping (only if `triage` is installed)
- [domain.md](./domain.md): domain doc consumer rules + layout

For "other" issue trackers, write `docs/agents/issue-tracker.md` from scratch using the user's description.

### 5. Done

Tell the user the setup is complete and which engineering skills will now read from these files. Mention they can edit `docs/agents/*.md` directly later; re-running this skill is only necessary if they want to switch issue trackers or restart from scratch.
