# hy2k-skills

Engineering workflow skills for AI agents: grill the plan, write the spec, cut
the tickets, implement, review. Installable with
[APM](https://microsoft.github.io/apm/) into any repo, on any harness that reads
`.agents/skills/`.

A derivative of [mattpocock/skills](https://github.com/mattpocock/skills) v1.3.1
— curated, harness-neutral, and explicit about where an agent's files belong.
See [Provenance](#provenance).

## Quick links

- [Set it up in another repo with an agent](#set-it-up-in-another-repo-with-an-agent)
- [Update an installed repo with an agent](#update-an-installed-repo-with-an-agent)

## Install

```yaml
# apm.yml in the consuming repo
dependencies:
  apm:
    - git: https://github.com/hy2k/skills
      ref: v0.2.0
```

Then:

```
apm install
```

Skills land in `.agents/skills/<name>/`. Pin a tag, not a branch — the lockfile
records the resolved commit and content hash, so a fresh clone resolves
byte-for-byte.

## Set it up in another repo with an agent

Copy this prompt into an agent that can read the web and run `apm`. It is the
same for a repo with no `apm.yml` (greenfield) and one that already installs
skills (brownfield):

```text
Use https://github.com/hy2k/skills to replace mattpocock/skills @apm.yml

Read that repo's README first, then check this repo's apm.yml:
- apm.yml exists: swap any mattpocock/skills dependency for the git dependency
  on https://github.com/hy2k/skills, pinned to its latest tag, and leave the
  rest of the file untouched.
- no apm.yml: add the dependency in the shape that README shows.

Run `apm install`, then run the package's `agent-conventions` skill once. Keep
both `apm.yml` and `apm.lock.yaml` in the repo: the lockfile pins the resolved
commit and content hash.

If `apm audit` fails because `TMPDIR` sits inside this checkout, follow "apm
audit under a sandbox" in that README instead of skipping the audit.
```

## Run the setup once per repo

After `apm install`, invoke the `agent-conventions` skill from your harness
(`skill://agent-conventions` under omp, `/agent-conventions` under Claude Code,
or whatever your runtime's skill picker is called). It is marked
`disable-model-invocation`, so it never fires on its own.

`agent-conventions` writes four small docs under `docs/agents/` and a summary
block into the repo's existing `AGENTS.md` or `CLAUDE.md`:

| Doc | Answers |
| --- | --- |
| `artifact-paths.md` | where research, decisions, specs, tickets, and throwaways go |
| `issue-tracker.md` | where issues live — GitHub, GitLab, or local markdown |
| `triage-labels.md` | which label strings the five triage roles map to |
| `domain.md` | where the glossary and ADRs live, and how to read them |

Every other skill reads those files instead of guessing.

## Update an installed repo with an agent

APM updates the skill tree, not the `docs/agents/` files that `agent-conventions`
wrote. After a release, copy this prompt into an agent that can read the web and
run `apm`, with the new tag. It pins the tag, refreshes the docs from the new
seed templates, and keeps repo-specific facts.

```text
Use https://github.com/hy2k/skills at tag <tag> to update this repo.

Read that tag's README.md and CHANGELOG.md first. Then:
- Update the hy2k/skills git dependency in apm.yml to ref <tag>; leave the rest
  untouched.
- Run `apm install`.
- Run the `agent-conventions` skill in refresh mode. Compare every
  docs/agents/*.md in this repo with the matching seed template in the installed
  skill, apply the template changes, and keep repo-specific facts and sections
  this package does not own.
- Update the `## Agent skills` index only if the set of docs changed.
- Run `apm audit` and the repo's quality gates. Report each doc changed and each
  local deviation kept.

If `apm audit` fails because `TMPDIR` sits inside this checkout, follow "apm
audit under a sandbox" in that README instead of skipping the audit.
```

## The workflow

| Step | Skill |
| --- | --- |
| Stress-test a plan before writing it | `grilling` — records the ADRs and terms it settles once you confirm |
| Turn the conversation into a spec | `to-spec` |
| Cut the spec into tracer-bullet tickets | `to-tickets` |
| Work an unknown effort down a map of tickets | `wayfinder` |
| Implement a ticket test-first | `tdd`, `implement` — sequential on the current branch by default, worktree-parallel only when you ask for it |
| Review the diff | `code-review` |
| Diagnose a failure before patching it | `diagnosing-bugs` |
| Record a decision or a term | `domain-modeling` |
| Gather evidence from primary sources | `research` |
| Draft the PR body | `pr` |
| Snapshot a session for the next agent | `handoff` |
| Provision what only a human can | `wizard` |

## Where files go

The package states this rather than inferring it:

| Artefact | Default path |
| --- | --- |
| Research record | `docs/research/NNNN-slug.md` |
| Decision record | `docs/adr/NNNN-slug.md` |
| Shipped product spec | `docs/specs/<feature>.md` |
| In-flight spec and tickets | `.scratch/<feature>/` |
| Throwaway artefact | `$TMPDIR`, else `<repo>/.tmp/` |

A repo overrides any of these in `docs/agents/artifact-paths.md`.

## Authoring

Skills are primitives under `.apm/skills/<name>/SKILL.md`; `apm.yml` declares
the package. Release shape is a single plugin:

```
apm install                      # self-install, writes apm.lock.yaml (gitignored)
apm compile --validate           # frontmatter and primitive check
apm audit                        # hidden-Unicode and drift scan
apm pack --format agent-plugin   # portable Agent Plugins v1 bundle in build/
git tag v0.2.0 && git push --tags
```

`apm compile` without `--validate` exits 1 here: this package ships no
instructions, so there is no `AGENTS.md` to generate. `--validate` is the check
that applies. `build/`, `.agents/skills/`, `apm_modules/` and `apm.lock.yaml`
are gitignored — a fresh clone reproduces them from `.apm/` and `apm.yml`.

### apm audit under a sandbox

`apm audit` replays the install into a temp directory and refuses a scratch path
inside the project tree. If your sandbox profile sets `TMPDIR` inside the
checkout, point it outside for the audit:

```sh
mkdir -p ~/.cache/apm-audit
TMPDIR=~/.cache/apm-audit apm audit
```

`--no-drift` skips the replay but loses drift coverage. `APM_TEMP_DIR` and
`apm config temp-dir` do not move this scratch tree.

That is a local workaround, not the fix. The durable options are a `TMPDIR`
outside the checkout in the shared sandbox profile, or upstream `apm` making
audit scratch honour `APM_TEMP_DIR`/`temp-dir` instead of falling back to the
working directory.

Consumers pin the tag. Move the `Unreleased` entries in `CHANGELOG.md` under the
new version, bump `version:` in `apm.yml` and `plugin.json` together, then tag
`v<version>`.

## Provenance

Derivative of [mattpocock/skills](https://github.com/mattpocock/skills) at tag
`v1.3.1`, commit `24fe0ef7737efae15c87225755e9f6f5965e4888`, MIT-licensed,
Copyright (c) 2026 Matt Pocock.

It is a derivative, not a fork that tracks upstream: upstream is not a
dependency of this package and is never merged into it, and releases here are
tagged independently. Skills under `.apm/skills/` are upstream text except
where git history records an edit — the seeding commit and every later one say
what diverged and why.

## Licence

MIT, both copyright holders named in [LICENSE](./LICENSE).
