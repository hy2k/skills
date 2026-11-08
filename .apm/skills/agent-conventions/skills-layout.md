# Skills layout

Where the agent skills in this repo come from, and which copy is authoritative when you want to change one.

## Which posture is this repo

The two are mutually exclusive: a tree is either generated from a manifest, or it is the source itself. Fill in the row that matches, and delete the other section.

| | Generated (APM) | Hand-authored |
| --- | --- | --- |
| Source of truth | `.apm/skills/<name>/` | `.agents/skills/<name>/` |
| `.agents/skills/` | output of `apm install` | the source |
| After editing a skill | run `apm install` | nothing to run |
| Versions | `apm.yml` pins, `apm.lock.yaml` commits | git |
| Adding a skill | directory under `.apm/skills/`, then install | directory under `.agents/skills/` |
| Drift | `git diff --exit-code .agents` after install | none — git is the record |

This repo is: `<generated | hand-authored>`.

## If generated

- **Never hand-edit `.agents/skills/`.** The next install overwrites the edit, and a committed edit reads as drift.
- **Edit `.apm/skills/<name>/`**, then run `apm install`.
- **A skill from another package has no local home.** Change the pin, or fork the package; do not patch the deployed copy.
- **Commit `<apm.yml` and `apm.lock.yaml` always, plus `.agents/skills/` when the repo wants a clone to need no install>**. `apm_modules/` is gitignored and rebuilt.
- **Regeneration is byte-stable**, so drift is checkable: after an install, `git diff --exit-code .agents` must be clean. `apm audit` reports content drift and hidden characters.
- **Pin a tag.** Use a commit when the upstream has none, and `path:` for a skill inside a monorepo. Never pin a branch: the lockfile records a commit, and a branch silently changes what that commit was supposed to be.

## If hand-authored

- **`.agents/skills/` is the source.** Edit it directly; that directory is what the harness loads, so there is no second copy to keep in step.
- **There is no install step and no lockfile.** What is committed is what runs.
- **A skill taken from elsewhere is a fork.** Record where it came from and keep its licence, in the skill or in the repo's third-party notices. An unattributed copy is the failure mode this line exists to prevent.
- **One directory per skill, and the directory name is the identity** — it must match `name:` in the frontmatter.
- **Do not introduce `.apm/skills/` later without migrating.** Two copies of one skill is worse than either posture alone: edits land in the copy nothing reads.
