# Issue tracker: Local Markdown

Issues and specs for this repo live as markdown files in `.scratch/`.

## Conventions

- One feature per directory: `.scratch/<feature-slug>/`
- The spec is `.scratch/<feature-slug>/spec.md`
- Implementation issues are one file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01`, never a single combined tickets file
- Every ticket opens with the frontmatter block defined in `metadata-format.md` in the `agent-conventions` skill. `type` and `status` are always present; `blocked_by`, `recommended_order`, and `subsumed_by` appear only when they apply
- `status` takes the triage role strings from `triage-labels.md` plus `claimed`, `deferred`, `resolved`, and `subsumed`. A ticket is open unless its status is `deferred`, `resolved`, `subsumed`, or `wontfix`
- `blocked_by` lists the refs that gate the ticket: bare numbers within the feature, `<feature-slug>/NN` across features. The ticket is unblocked when every ref is `resolved`
- `recommended_order` lists the whole shared-surface sequence as refs, this ticket included; its index plus one is this ticket's position. It orders sessions, never gates
- `subsumed_by` names the ticket that absorbed this one, and is required when `status: subsumed`
- Comments and conversation history append to the bottom of the file under a `## Comments` heading

## When a skill says "publish to the issue tracker"

Create a new file under `.scratch/<feature-slug>/` (creating the directory if needed).

## When a skill says "fetch the relevant ticket"

Read the file at the referenced path. The user will normally pass the path or the issue number directly.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a file with one **child** file per ticket.

- **Map**: `.scratch/<effort>/map.md` (the Notes / Decisions-so-far / Fog body).
- **Child ticket**: `.scratch/<effort>/issues/NN-<slug>.md`, numbered from `01`, with the question in the body. The frontmatter records `type` (`research`/`prototype`/`grilling`/`task`) and `status`.
- **Blocking**: `blocked_by` in the frontmatter. A ticket is unblocked when every file it lists is `resolved`.
- **Frontier**: scan `.scratch/<effort>/issues/` for files that are open, unblocked, and unclaimed; first by number wins.
- **Claim**: set `status: claimed` and save before any work.
- **Resolve**: append the answer under an `## Answer` heading, set `status: resolved`, then append a context pointer (gist + link) to the map's Decisions-so-far in `map.md`.
- **Subsumed**: set `status: subsumed` and `subsumed_by: <ref>`, with the reason in the body. A subsumed ticket is off the frontier.
- **Deferred**: set `status: deferred`, and record who deferred it, when, and until what, in the body. A deferred ticket is off the frontier.
