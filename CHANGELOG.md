# Changelog

All notable changes to this package are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this package uses
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `to-tickets` writes an advisory `Recommended order` line (`<n> of <total> | <NN> → <NN> → …`) when proposed tickets share a surface, records the shared surfaces and reasons in each ticket's Comments, and reserves `Blocked by` for dependencies that would make the blocked work wrong or wasted. A single ticket or a set with no shared surface carries no order line, and `Recommended order` is never encoded as a blocking link.
- `implement` runs a set that carries `Recommended order` sequentially, one writer per shared surface, and asks for an explicit override before parallel mode on a set that shares one.

### Changed

- `to-tickets` and `implement` bind the frontier, and with it the stop condition, to the set the user named: the feature's `issues/` directory locally or the parent issue's children on a real tracker. Other feature directories and unrelated parent issues are not frontier candidates, so a run ends when that set is empty rather than roaming the repo.

## [0.2.1] - 2026-11-11

### Changed

- `asking.md` documents the separate option-note channel omp returns with HTML form answers (`notes[]`, plus `Option notes:` in the result text); `grilling` names the decision's rationale rather than a per-option note.

## [0.2.0] - 2026-11-10

### Added

- `agent-conventions` can refresh an existing `docs/agents/` setup from updated seed templates.
- README update prompt for applying a new release tag to an installed repo.

### Changed

- `asking.md` detects the question tool by schema shape (`html` form vs `questions[]`) and documents omp's HTML form contract.
- `grilling` asks for rationale explicitly when a decision needs more than the pick.

## [0.1.0] - 2026-11-09

### Added

- Initial release: engineering workflow skills for grilling, specs, tickets, implementation, review, diagnosis, domain modeling, research, PRs, handoffs, and provisioning.
