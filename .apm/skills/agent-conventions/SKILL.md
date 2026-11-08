---
name: agent-conventions
description: Use when a repo needs its agent conventions pinned down before the other engineering skills run — where research, decision records, product specs, in-flight tickets, and ephemeral files belong; where issues are tracked; the triage label vocabulary; and the domain doc layout. Writes the per-repo docs the other skills read.
disable-model-invocation: true
---

# Agent conventions

Scaffold the per-repo configuration the engineering skills assume:

- **Artifact paths**: where each kind of file an agent writes actually belongs
- **Asking**: which structured question tool this harness exposes, and how it marks a recommendation
- **Issue tracker**: where issues live (GitHub by default; local markdown is also supported out of the box)
- **Triage labels**: the strings used for the five canonical triage roles
- **Domain docs**: where the glossary and ADRs live, and the consumer rules for reading them
- **Quality gates**: which commands must pass, and how far they reach for a given change
- **Long-running processes**: how to start a server this session owns, and how to leave everyone else's alone
- **Skills layout**: where the skill tree comes from, and which copy is authoritative

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write.

Every one of these has a **default**. Take the default without asking unless exploration found evidence the repo already decided otherwise. The point of this skill is to write the file, not to run a questionnaire.

**Brownfield is the normal case.** Any repo that does not yet pin this package already carries some of these rules in prose, somewhere in its agent file, so the first run there is a merge rather than a write. See [Merging into an existing repo](#merging-into-an-existing-repo).

## Process

### 1. Explore

Look at the current repo to understand its starting state. Read whatever exists; don't assume:

- `git remote -v` and `.git/config`: is this a GitHub repo? Which one?
- `AGENTS.md` and `CLAUDE.md` at the repo root: does either exist? Is there already an `## Agent skills` section in either? Which subsections does it carry, and what are they called?
- **The rest of that file, not just the block.** Which top-level sections already state a rule this skill now owns — documentation, quality gates, servers, skills, asking, or a temp-file rule buried under another heading? The merge step routes them into docs; list them now, with their heading text, so step 4 can propose one migration.
- `docs/agents/`: does this skill's prior output already exist? Read each file that does — an existing doc is edited, never replaced.
- `docs/research/`, `docs/adr/`, `docs/specs/`, `docs/glossary.md`, `GLOSSARY.md`, `GLOSSARY-MAP.md`: which of the documented layouts is already in use?
- `.scratch/`, `.tmp/`, `TMPDIR` in the environment: signs of an existing local-markdown tracker and an ephemeral-artefact convention
- Is the `triage` skill installed? (a `triage` skill folder alongside this one, or `triage` in your available skills.) This decides whether the triage section runs at all.
- `apm.yml` and `.apm/skills/`: is the skill tree APM-managed, and does the repo own any skills? `git ls-files .agents/skills | head -1` says whether the tree is committed. A populated `.agents/skills/` with no manifest means the tree is hand-authored source, not output.
- Task runners and their scripts: `package.json` scripts, `pyproject.toml`, `mix.exs`, `mise.toml`, a `Makefile`. These name the quality gates.
- A server command: a `dev` or `preview` script, a framework config, a `docker compose` service. Its presence decides whether the long-running-processes section runs.
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
| Ephemeral artefact | `$TMPDIR`, else `<repo>/.tmp/` |

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

**Section F: Quality gates.**

> Explainer: an agent that cannot name the checks it ran cannot say the work is done. This file makes "it works" checkable, and it is the one the verification rule points at.

Default posture: read the commands out of whatever already owns them — `package.json` scripts, `pyproject.toml`, `mix.exs`, `mise.toml` — and write them in. Ask only which of them is the full sequence and which are the targeted checks.

Write it to `docs/agents/quality-gates.md` using [quality-gates.md](./quality-gates.md) as the seed. Skip the section when exploration found no runnable check at all, and say so.

**Section G: Long-running processes.**

> Explainer: anything that binds a port outlives the session that started it. The expensive failure is not the leaked port, it is stopping the owner's server or deleting a lock file to clear it.

Default posture: write the table for the server exploration found — command, foreground switch, granted port range, lock file, readiness check. When the repo has no server, skip the section and say so.

Write it to `docs/agents/long-running-processes.md` using [long-running-processes.md](./long-running-processes.md) as the seed.

**Section H: Skills layout.**

> Explainer: an agent has to know which copy of a skill it may edit. Where the tree is generated, an edit to `.agents/skills/` is lost on the next install; where it is hand-authored, a rule that belongs in `.agents/skills/` has no other home and writing it under `.apm/` puts it where nothing reads.

Default posture: `apm.yml` present means **generated**; no manifest with a populated `.agents/skills/` means **hand-authored**. Write the matching half of the doc, delete the other, and fill the "this repo is" line. Skip the section only when the repo has no skill tree at all.

Write it to `docs/agents/skills-layout.md` using [skills-layout.md](./skills-layout.md) as the seed.

### 3. Confirm and edit

Show the user a draft of:

- The `## Agent skills` block to add to whichever of `CLAUDE.md` / `AGENTS.md` is being edited (see step 4 for selection rules)
- The contents of every doc this run writes — including an existing one being brought to the template's shape, with its repo-specific facts carried into the fill-in slots

Let them edit before writing.

### 4. Write

**Pick the file to edit:**

- If `CLAUDE.md` exists, edit it.
- Else if `AGENTS.md` exists, edit it.
- If neither exists, ask the user which one to create; don't pick for them.

Never create `AGENTS.md` when `CLAUDE.md` already exists (or vice versa); always edit the one that's already there.

#### Merging into an existing repo

**Any repo that does not yet pin this package is brownfield**, and a brownfield repo usually already carries most of these rules in prose. So the first run is a merge, not a write: produce the docs, then move what already exists into them.

**Two surfaces carry the same rules, and both have to be checked.**

1. The `## Agent skills` block, when the file has one.
2. Top-level sections elsewhere in the same file. Every subject this skill owns usually appears somewhere already — under headings like `## Documentation`, `## Quality gates`, `## Long-running processes`, `## Verification`, `## Agent skills`, `## Asking the owner`. A rule also hides under a heading that is not its own, so read for the rule, not for the heading.

Match by **subject**, never by heading text. Then route what you find:

| An existing section is likely to hold | The doc that now owns it |
| --- | --- |
| Documentation, Docs, Writing specs | `domain.md` |
| Temporary files, Temp, Scratch | `artifact-paths.md` — the temp-path rule only |
| Quality gates, Verification, Checks | `quality-gates.md` |
| Long-running processes, Dev server, Servers | `long-running-processes.md` |
| Agent skills, Skills, APM | `skills-layout.md` |
| Asking, Interview questions, Grilling questions | `asking.md` |
| Issue tracker, Tickets, Triage | `issue-tracker.md`, `triage-labels.md` |

**A mixed section contributes only the part this package owns.** `## Sandbox` is the usual case: it holds harness policy — how the sandbox is enforced, the profile file, network and port grants, each with its reason — next to one rule about where temporary files go. Move that rule into `artifact-paths.md` and leave the policy where it is. Sandbox policy belongs to the harness and changes with it; this package states the consequence an agent must act on (a write-only `/tmp`), never the mechanism.

**Move the rule, never copy it.** Once its content is in the doc, the section goes — the index line already names that doc, so a leftover pointer section is the duplication this package removes. A section that also carries something the package does not own keeps only that part.

**Repo-specific facts travel into the doc's fill-in slots**, not into the block: a port range, a lock-file path, a command, a PASS criterion. Each template has a table for those.

**Show the diff before deleting prose.** A top-level section often carries something the doc does not cover — a business boundary, a consent rule, a domain fact, a repository map. Move what the doc owns, leave the rest, and let the owner see the before and after. Report which sections were removed and what stayed.

**Adopt in one run.** A half-migrated repo keeps two copies of the same rule, which is the drift this package exists to remove, so a brownfield run takes the block and every top-level section it owns in the same change. Split only when the owner asks for it, and name what was left.

**Running twice changes nothing.** If the block and the docs already say what this run would say, report that and write nothing.

Within the `## Agent skills` block itself:

- **`## Agent skills` is the integration point.** Keep the heading exactly as it is. Never create a second one, never rename it, never nest the block under another heading.
- **The block is an index, not a summary.** One line naming `docs/agents/` and the files in it. A `### Heading` whose body is "See `docs/agents/<subject>.md`" repeats the filename the reader can already see, and costs one edit per doc whenever the set changes.
- **Normalize an existing block to the index.** A repo carrying `### Asking` / `### Interview questions` / `### Triage labels` subsections gets them collapsed into the index line — after moving any wording worth keeping into the matching doc. Brownfield means adopting this package's shape, not preserving the old one; a repo that keeps its own layout leaves every later run guessing which shape is current.
- **Existing docs adopt the template as well.** A `docs/agents/quality-gates.md` written by an older run is rewritten to the current template with its repo-specific facts carried into the fill-in slots. Two shapes of the same doc across repos is the same guessing problem one level down.
- **Only an explicit instruction at invocation preserves a deviation.** If the owner asks this run to keep the repo's own structure, keep it and say so in the report. Absent that, adopt.
- **Leave the sections this package does not own untouched.** `### Project conventions`, `### Business boundaries`, a usage-rules block, a repository map, a consent rule — those are the repo's, not ours to normalize.
- **Write the doc before the pointer.** A pointer to a file that does not exist is worse than no pointer.

The block, when the repo has none:

```markdown
## Agent skills

Repo conventions live in `docs/agents/`; read the one matching the subject before acting — `artifact-paths.md` (where files go), `asking.md` (question tool), `issue-tracker.md`, `triage-labels.md`, `domain.md` (glossary, ADRs), `quality-gates.md`, `long-running-processes.md`, `skills-layout.md`.

Skills under `.agents/skills/` may name another harness's tools or slash commands. Treat those as intent, not a requirement: use the equivalent capability in the current harness, and where none exists, follow the skill's workflow with the tools available.
```

Build the index line from the docs this run actually writes: omit any file whose section was skipped, so the index never names a document that does not exist. Write `triage-labels.md`, and name it in the index, only when `triage` is installed and Section C ran. Keep the harness paragraph even when everything else is trimmed — it is the one rule that applies to every skill in the tree.

Then write the docs files using the seed templates in this skill folder as a starting point:

- [artifact-paths.md](./artifact-paths.md): the five-tier layout
- [asking.md](./asking.md): the question channel, per harness
- [issue-tracker-github.md](./issue-tracker-github.md): GitHub issue tracker
- [issue-tracker-gitlab.md](./issue-tracker-gitlab.md): GitLab issue tracker
- [issue-tracker-local.md](./issue-tracker-local.md): local-markdown issue tracker
- [triage-labels.md](./triage-labels.md): label mapping (only if `triage` is installed)
- [domain.md](./domain.md): domain doc consumer rules + layout
- [quality-gates.md](./quality-gates.md): commands, scoping, and the verification rule
- [long-running-processes.md](./long-running-processes.md): ownership, the auto-background trap, bounded runs
- [skills-layout.md](./skills-layout.md): generated tree, which copy to edit, pinning

For "other" issue trackers, write `docs/agents/issue-tracker.md` from scratch using the user's description.

### 5. Done

Tell the user the setup is complete and which engineering skills will now read from these files. Mention they can edit `docs/agents/*.md` directly later; re-running this skill is only necessary if they want to switch issue trackers or restart from scratch.
