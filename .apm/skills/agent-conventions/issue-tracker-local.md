# Issue tracker: Local Markdown

Issues and specs for this repo live as markdown files in `.scratch/`.

## Conventions

- One feature per directory: `.scratch/<feature-slug>/`
- The spec is `.scratch/<feature-slug>/spec.md`
- Implementation issues are one file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01`, never a single combined tickets file
- Build tickets may carry a `- **Recommended order:**` bullet (`<n> of <total> | <NN> → <NN> → …`) near the top of the file when two tickets share a surface. It orders sessions; it is not a gate. Wayfinding children do not carry it
- Triage state is recorded as a `- **Status:**` bullet near the top of each issue file (see `triage-labels.md` for the role strings)
- Metadata is a bullet list, one bullet per field. Bare `Field: value` lines written back to back merge into one paragraph in a Markdown preview, so a field never sits on its own plain line
- Comments and conversation history append to the bottom of the file under a `## Comments` heading

## When a skill says "publish to the issue tracker"

Create a new file under `.scratch/<feature-slug>/` (creating the directory if needed).

## When a skill says "fetch the relevant ticket"

Read the file at the referenced path. The user will normally pass the path or the issue number directly.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a file with one **child** file per ticket.

- **Map**: `.scratch/<effort>/map.md` (the Notes / Decisions-so-far / Fog body).
- **Child ticket**: `.scratch/<effort>/issues/NN-<slug>.md`, numbered from `01`, with the question in the body. Two bullets near the top record its type and state: `- **Type:**` (`research`/`prototype`/`grilling`/`task`) and `- **Status:**` (`claimed`/`resolved`).
- **Blocking**: a `- **Blocked by:** NN, NN` bullet near the top. A ticket is unblocked when every file it lists is `resolved`.
- **Frontier**: scan `.scratch/<effort>/issues/` for files that are open, unblocked, and unclaimed; first by number wins.
- **Claim**: set the `Status` bullet to `claimed` and save before any work.
- **Resolve**: append the answer under an `## Answer` heading, set the `Status` bullet to `resolved`, then append a context pointer (gist + link) to the map's Decisions-so-far in `map.md`.
