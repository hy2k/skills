# Quality gates

The checks that must pass before an agent may claim a change works, and how far they reach for a given change.

## Commands

| Gate | Command | Runs on |
| --- | --- | --- |
| Format | `<...>` | the whole tree |
| Lint | `<...>` | the whole tree |
| Types | `<...>` | the whole tree |
| Tests | `<...>` | the whole tree |
| Build | `<...>` | only when the change reaches shared code or config |

Fill these from whatever already owns them — `README.md`, `package.json` scripts, `pyproject.toml`, `mix.exs`, the task runner in `mise.toml`. A gate this repo does not have is left out; it is absent, not optional.

## Scope the gate to the change

- **Behavior, shared code, build config, or dependencies** — run the full sequence.
- **A session-sized change** — one post, one CSS tweak, one doc record, one dependency bump with no code change — run only the targeted checks named above.
- **A narrow scope never permits presenting or committing broken work.** If a targeted check is inconclusive, or the change turns out to reach shared behavior after all, run the full sequence.

## Verification is not a passing build

- **Run the thing.** A build that compiles, a test that passes and a type check that is clean are each evidence that the code is well-formed, not that the change works.
- **UI work means observing the rendered result** — the page, the interaction, the state after the click. A green build is not a substitute.
- **Never modify generated output to make a gate pass.** Fix the generator, or say the gate is wrong. Editing `dist/`, a lockfile, or a committed artifact to silence a check hides the defect that produced it.
- **Do not silence a diagnostic.** A `# noqa`, a `@ts-ignore`, a skipped test or a widened type is a claim that the finding is wrong; make that claim explicitly or fix the finding.

## Reporting

Say which gates ran and how each exited. A gate that did not run is named as not run, with the reason. Partial verification reported as full is worse than a stated gap.
