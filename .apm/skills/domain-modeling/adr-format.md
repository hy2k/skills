# ADR Format

ADRs live in `docs/adr/` as `NNNN-slug.md`: one file, one question, one current answer.

Create `docs/adr/` lazily, when the first record is needed.

## The gate

Offer a record only when all four hold:

1. **Hard to reverse**: changing your mind later costs real work.
2. **Surprising without context**: the code alone would make a reader ask why.
3. **A real trade-off**: there were live alternatives, and one was picked for stated reasons.
4. **One question**: the record answers that question alone.

Miss any of the four and there is no record. A tuned constant, a fixed typo, a renamed file: none of these is a decision.

The slug names the subject; the first sentence answers the question. Before writing, search for an accepted record on that subject (`rg -l '<subject>' docs/adr/`). If one exists, you are revising or superseding it, never adding beside it.

## What qualifies

- Architectural shape and lock-in: monorepo, event sourcing, database, bus, auth provider, deployment target.
- Boundaries between contexts, including the explicit no-s: who owns the data, what crosses by ID only.
- Deliberate deviations from the obvious path, so the next engineer does not "fix" a deliberate thing.

## Revise, supersede, or neither

| Change | Action |
| --- | --- |
| Wording only: adds no claim, removes none, the reader's actions are unchanged | Edit in place |
| Typo, broken link, formatting | Edit in place |
| Status transition | Edit `status`; set `date` when the record becomes accepted |
| The answer to the question changes | New record; the old one gets `status: superseded` and `superseded_by` |
| Part of the answer changes | New record; the old one gets `status: partially-superseded`, `superseded_by`, and one scope bullet per edge |
| A constraint or a rejected alternative stops holding | New record |
| Same answer, new evidence: a sweep, a review, a tuning round | No record; see Evidence |
| One of the four gate conditions was never true | No record |

The in-place test is a diff test: the change touches words, not sentences. Any added or deleted sentence that asserts something is a supersede.

## Template

```md
---
status: {proposed | accepted | partially-superseded | superseded | deprecated}
date: {YYYY-MM-DD, the date the record became accepted}
---

# {Decision, stated as the answer}

{1-3 sentences: the question, the answer, and why.}
```

The normative metadata block — keys, status vocabulary, and YAML style — is in [`metadata-format.md`](../agent-conventions/metadata-format.md).

The body is one paragraph. `Considered Options` and `Consequences` are optional; add them only when a rejected alternative or a downstream effect is worth remembering. History lives in the status chain: do not copy the replaced answer into the new record.

## Retirement

A retired record changes in one place: its frontmatter block.

```yaml
status: superseded
superseded_by: [0012]
```

The replacement carries the reverse link:

```yaml
supersedes: [0007]
```

A partial retirement lists every replacement and puts one scope bullet under the H1, naming the clause each one took:

```yaml
status: partially-superseded
superseded_by: [0019, 0020]
```

```markdown
- 0019 — the goal.
- 0020 — the web-first venue line.
```

`deprecated` is for a decision that no longer applies, with nothing replacing it. A `superseded` or `deprecated` record accepts metadata and typo fixes only; a `partially-superseded` record stays editable in the parts no bullet names. Moving metadata into the block is a formatting edit, so a retired record may receive it.

## Numbers

A number is an append-only address, not a label. Take the highest existing number plus one. Never reuse a number. Never renumber a block, and never delete a record.

The only rename allowed repairs a duplicate number: two files on `0040`. Rename the one git history added later, and rewrite every reference to the old number in the same change. `git grep -n 'ADR-0040'` must return nothing afterwards.

## Evidence

Measurements never enter a record. A benchmark, a sweep, a review of candidate values goes to `docs/research/NNNN-slug.md`; the record links it if a reader needs the proof.

One sweep table covers one build: every row carries the short commit SHA in a `commit` column, and the header states the date and that SHA. Never append rows measured on another build; start a new table.

## Never in a record

- Event logs, or a narrative of what happened.
- Ticket, review, or pull-request history.
- Merge notes: "combined 0041 and 0042".
- Counts and measurement reports. A constraint value is fine ("under 200ms per the partner contract"); the sweep that produced it is not.
- File enumerations. Name the module or the context; list no paths.
- Reviewer names. The acceptance date lives in the block; body prose carries no date.
- Design-guide material. Link it instead.

Delete test: remove each sentence and ask whether the decision still stands. If it does, the sentence does not belong.
