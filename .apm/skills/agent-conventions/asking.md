# Asking

How this repo's agent puts a decision to its owner: which tool it calls, and the field names that tool expects.

Every skill that interviews — `grilling` above all — reads this file before asking anything. A round written as prose costs a round trip and loses the selection; a round in the wrong shape fails outright, because these schemas are closed.

## The tool in use here

| | |
| --- | --- |
| Tool | `<ask \| question \| ask_user_question \| AskUserQuestion \| none>` |
| Multi-select key | `<multi \| multiple \| multi_select \| multiSelect \| n/a>` |
| Question `id` | `<required \| omit>` |
| Questions per round | `<3–4 \| harness cap>` |

Fill the row for the harness this repo actually runs, from the table below. A repo driven by more than one harness gets one row per harness.

## What each harness expects

Sources: `oh-my-pi` `33a063c`, `opencode` `a9a067f`, `deepseek-harness` `fcc0ad46`, `pi` `5ea81d2`, and the Claude Code docs at `code.claude.com` — all read 2026-11-09.

| | `ask` (omp / oh-my-pi) | `question` (opencode) | `ask_user_question` (deepseek-harness) | `AskUserQuestion` (Claude Code) | `question` (pi extension) |
| --- | --- | --- | --- | --- | --- |
| Container | `questions[]`, at least 1 | `questions[]`, non-empty | `questions[]` | `questions[]`, one to four | single `question`, no array |
| `id` per question | **required** | absent | **required** | absent | absent |
| `header` | optional | **required** | optional | **required** | absent |
| `options[]` | required | required | optional | required | required |
| `options[].description` | optional | **required** | optional | **required** | optional |
| Multi-select key | `multi` | `multiple` | `multi_select` | `multiSelect` | absent |
| Marking the recommendation | `recommended: <index>`, or the label suffix | label suffix, option first | label suffix, option first | no field; the label carries it | prose only |
| Free-text answer | an `Other (type your own)` row | a `Type your own answer` row | returned as `custom` on the answer | an `Other` row, plus a notes field | a `Type something.` row |

Two columns are narrower than they look:

- **Claude Code** requires `header` and `description` by type, has no recommendation field at all, and leaves the free-text row to the app — the CLI shows an `Other` row, an Agent SDK app supplies its own.
- **pi** ships no question tool in its core; the column describes its documented extension example, so treat it as "only if the extension is installed".

## Portable rules

These hold in every harness above, so they need no per-harness note:

- **One `questions[]` entry per question.** Never fold two questions into one entry.
- **2–4 options.** More than four is a survey, not a question, and the smallest cap in the family is four.
- **Always give a short `header` and a one-sentence `description` per option.** Two harnesses require both; the rest ignore them.
- **Put the recommended option first and end its label with ` (Recommended)`.** opencode and deepseek-harness expect exactly that; omp checks for the suffix before adding its own, so it never doubles; Claude Code has no recommendation field, so the suffix is simply part of the label.
- **Never add your own free-text option.** Every harness appends one, and omp rejects a label that collides with its reserved rows.
- **When the session has no such tool, write the round as prose and say that you did.**

## Facts are the agent's job

A question round carries decisions. Anything checkable — a file, a config value, a version, a count — is looked up before the round is composed, never offered to the owner as a choice. Tag `[INFERENCE]` rather than asserting.
