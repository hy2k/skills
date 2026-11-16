---
name: to-tickets
description: Break a plan, spec, or the current conversation into a set of tracer-bullet tickets, each declaring its blocking edges and, where tickets share a surface, its place in an advisory recommended order; published to the configured tracker (edges as text in one file per ticket locally, or native blocking links on a real tracker).
disable-model-invocation: true
---

# To Tickets

Break a plan, spec, or conversation into a set of **tickets**: tracer-bullet vertical slices, each declaring the tickets that **block** it.

The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run the `agent-conventions` skill.

File locations come from `docs/agents/artifact-paths.md` when it exists. Otherwise: research records in `docs/research/NNNN-slug.md`, shipped product specs in `docs/specs/<feature>.md`, in-flight specs and tickets under `.scratch/<feature>/`, and  ephemeral artefacts in `$TMPDIR` (`.tmp/` in the checkout) — never committed.

## Process

### 1. Gather context

Work from whatever is already in the conversation context. If the user passes a reference (a spec path, an issue number or URL) as an argument, fetch it and read its full body and comments.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code. Ticket titles and descriptions should use the project's domain glossary vocabulary, and respect ADRs in the area you're touching.

Look for opportunities to prefactor the code to make the implementation easier. "Make the change easy, then make the easy change."

### 3. Draft vertical slices

Break the work into **tracer bullet** tickets.

<vertical-slice-rules>

- Each slice cuts a narrow but COMPLETE path through every layer (schema, API, UI, tests): vertical, NOT a horizontal slice of one layer
- A completed slice is demoable or verifiable on its own
- Each slice is sized to fit in a single fresh context window
- Any prefactoring should be done first

</vertical-slice-rules>

Give each ticket its **blocking edges**: the other tickets that must complete before it can start. A ticket with no blockers can start immediately.

**Wide refactors are the exception to vertical slicing.** A **wide refactor** is one mechanical change (rename a column, retype a shared symbol) whose **blast radius** fans across the whole codebase, so a single edit breaks thousands of call sites at once and no vertical slice can land green. Don't force it into a tracer bullet; sequence it as **expand–contract**. First expand: add the new form beside the old so nothing breaks. Then migrate the call sites over in batches sized by blast radius (per package, per directory), each batch its own ticket blocked by the expand, keeping CI green batch to batch because the old form still exists. Finally contract: delete the old form once no caller remains, in a ticket blocked by every migrate batch. When even the batches can't stay green alone, keep the sequence but let them share an integration branch that all block a final integrate-and-verify ticket; green is promised only there.

**Ordering is not blocking.** A shared file, exporter, generated artifact, running service, or verification command does not gate a ticket; it orders sessions. Reserve the blocking edge (`blocked_by` locally, `## Blocked by` on a real tracker) for a dependency that would make the blocked work wrong or wasted if done first. If starting a ticket first would only force a rebase, a regeneration, or waiting for another session, that is an ordering concern, not a gate.

### 4. Map shared surfaces

Before publishing, list the surfaces each proposed ticket touches:

- files and modules
- generated artifacts: generated source, fixtures, build output
- running services or sessions: dev server, Studio, database
- build and check commands
- verification state: fixtures, seed data, database rows, or a branch/worktree that a check reads or mutates

If two tickets name the same surface, assign a **recommended sequence** now, while the context is loaded. Do not ask the owner to re-derive the order later. Record the reason in that ticket's Comments, naming only the surfaces and edges that touch it. Keep it short, and only for an edge that is not obvious: a ticket that may close as subsumed, or one that samples an artifact another ticket produces.

The sequence is advisory: separate sessions, in that order, one writer per shared surface. It never blocks. A session may jump it and pay the conflict, rebase, or regeneration cost. If every ticket owns its surfaces, or the set is one ticket, there is no sequence: omit the field, and parallel sessions are safe.

### 5. Quiz the user

Present the proposed breakdown as a numbered list. For each ticket, show:

- **Title**: short descriptive name
- **Blocked by**: which other tickets (if any) must complete first
- **Recommended order**: its position and the whole sequence, when the set has one
- **What it delivers**: the end-to-end behaviour this ticket makes work

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the blocking edges correct: does each ticket only depend on tickets that genuinely gate it?
- Is the recommended order right: does each shared surface have one writer, and does the sequence start with the ticket most likely to absorb others?
- Should any tickets be merged or split further?

Iterate until the user approves the breakdown.

### 6. Publish the tickets to the configured tracker

Publish the approved tickets. **How** depends on the tracker the `agent-conventions` skill configured; the tickets are the same either way, only the shape of the blocking edges changes:

- **Local files** → write one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order (blockers first), then in recommended order within that constraint. Each file opens with the frontmatter block from `metadata-format.md`: `blocked_by` lists the refs that gate it, and `recommended_order` carries the whole shared-surface sequence as refs, this ticket included. Use the per-ticket file template below: one ticket per file, never a single combined file.
- **A real issue tracker (GitHub, Linear, …)** → publish one issue per ticket in dependency order (blockers first), then in recommended order, so each ticket's blocking edges can reference real identifiers. Use the platform's native blocking / sub-issue relationship for gates where it has one; otherwise set each ticket's "Blocked by" to the blocking issues. Never encode the recommended order as a blocking link. After all issues exist, edit each body's `## Recommended order` section in a second pass, with the real issue numbers, which need not ascend in run order: `2 of 4 | #10 → #12 → #11 → #13`. Apply the `ready-for-agent` triage label unless instructed otherwise; the tickets are agent-grabbable by construction.

When the set changes after publication — a ticket is added, removed, or reordered — rewrite the order in every ticket in the set in one pass: `recommended_order` in a local ticket's block, or the `## Recommended order` section on a real tracker. A later addition can leave the chain running out of numeric order; the field is authoritative.

Work the **frontier** of the named set only: open tickets in that feature's `issues/` directory locally, or the parent issue's open children on a real tracker, whose blockers are all done. A local ticket is open unless `status` is `deferred`, `resolved`, `subsumed`, or `wontfix`. Stop when that set has no open ticket left. Other feature directories and unrelated parent issues are not in scope, and their tickets are not frontier candidates. When open tickets in the set carry `recommended_order`, follow that sequence within the feature, in separate sessions, one writer per shared surface; otherwise start with the lowest number. The field is advisory and never blocks, and a ticket may still close as subsumed: `status: subsumed` with `subsumed_by` naming the ticket that absorbed it. Run sessions in parallel only when no ticket carries the field and no surface is shared.

Do NOT close or modify any parent issue.

<local-ticket-template>

---
type: <task | bug | grilling | research | prototype>
status: ready-for-agent
---

# <NN>: <Ticket title>

## What to build

The end-to-end behaviour this ticket makes work, from the user's perspective, not a layer-by-layer implementation list.

## Acceptance criteria

- [ ] Acceptance criterion 1
- [ ] Acceptance criterion 2

</local-ticket-template>

Add `blocked_by` when another ticket gates this one and `recommended_order` when the set shares a surface; omit both otherwise. Keys and YAML style: [`metadata-format.md`](../agent-conventions/metadata-format.md).

<issue-template>

## Parent

A reference to the parent issue on the tracker (if the source was an existing issue, otherwise omit this section).

## What to build

The end-to-end behaviour this ticket makes work, from the user's perspective, not layer-by-layer implementation.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by

- A reference to each blocking ticket, or "None (can start immediately)".

## Recommended order

<position> of <total> | #<n> → #<n> → …

</issue-template>

Omit `Recommended order` for a single ticket or a set with no shared surface.

## Worked example: one exporter, one generated place

Three tickets: `01` extends the export record, `02` renders a report from that record, `03` verifies the rendered report. They share the export module (`01`, `02`), the generated report fixture (`02`, `03`), and the verification command (`02`, `03`). None gates another, so no ticket carries `blocked_by`.

```markdown
---
type: task
status: ready-for-agent
recommended_order: [01, 02, 03]
---

# 01: Extend the export record with the report name

## What to build

...

## Acceptance criteria

- [ ] ...

## Comments

Shared surface: the export module, with 02. This ticket changes the record shape 02 consumes, so it runs first. 02 may close as subsumed if this ticket already renders the name.
```

```markdown
---
type: task
status: ready-for-agent
recommended_order: [01, 02, 03]
---

# 02: Render the report from the exported record

## What to build

...

## Acceptance criteria

- [ ] ...

## Comments

Shared surfaces: the export module with 01, and the generated report fixture and verification command with 03. It follows 01 so it samples the new record; it may close as subsumed.
```

```markdown
---
type: task
status: ready-for-agent
recommended_order: [01, 02, 03]
---

# 03: Verify the rendered report

## What to build

...

## Acceptance criteria

- [ ] ...

## Comments

Shared surfaces: the generated report fixture and verification command with 02. It runs last because it samples the rendered artifact.
```

On a real tracker the same three tickets carry `1 of 3 | #10 → #12 → #13`; the ids exist only after creation, so the chain is written in a second pass.

In either form, avoid specific file paths or code snippets: they go stale fast. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.
