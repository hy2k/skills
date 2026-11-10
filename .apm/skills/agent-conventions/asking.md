# Asking

How this repo's agent puts a decision to its owner: which tool it calls, which shape that tool declares, and the field names it expects.

Every skill that interviews — `grilling` above all — reads this file before asking anything. A round written as prose costs a round trip and loses the selection; a round in the wrong shape fails outright, because these schemas are closed.

## Detect the shape

The tool's name and the harness version do not decide the shape; its schema does. Read the schema before composing the round.

| Shape | What the schema shows | Container |
| --- | --- | --- |
| HTML form | an `html` parameter described as one `<form>` | one HTML `<form>` string |
| Structured questions | a `questions[]` parameter | an array of question objects |

An `ask` that exposes `questions[]` is structured; a `question` that exposes `html` is a form.

Sources: omp's `ask` tool schema, prompt, parser, and release notes v18.18.3–v18.18.6 — read 2026-11-10; `opencode` `a9a067f`, `deepseek-harness` `fcc0ad46`, `pi` `5ea81d2`, and the Claude Code docs at `code.claude.com` — carried from the previous revision, read 2026-11-09. The version tags are provenance only: the schema, not the harness or its version, decides the shape.

## The tool in use here

| | |
| --- | --- |
| Tool | `<ask \| question \| ask_user_question \| AskUserQuestion \| none>` |
| Shape | `<html form \| questions[] \| none>` |
| Multi-select | `<checkbox group or select multiple \| multiple \| multi_select \| multiSelect \| n/a>` |
| Question grouping | `<fieldset + legend \| questions[] entry \| single question>` |
| Questions per round | `<a few \| 3–4 \| harness cap>` |

Fill the row for the tool this session exposes; read its shape from the schema. A repo driven by more than one harness gets one row per harness.

## HTML form shape

omp's `ask` exposes this shape; the contract is the same for any tool whose schema takes `html`. The tool's own description and parser are authoritative; this section is the working contract. Send one `html` string holding exactly one `<form>` of plain semantic HTML. Never `<script>`, `<style>`, or `style=`; never add a submit button — the UI supplies Submit and Cancel. Hosts with a form surface render it whole; hosts without one prompt each named field in turn.

- **Every control needs a `name`.** The answer arrives as JSON keyed by `name`. A form with no named controls is rejected.
- **One-of:** `<input type="radio">` in a group sharing one `name`. **Many-of:** `<input type="checkbox">` in a group sharing one `name`, or `<select multiple>`. **Long lists:** `<select>`.
- **Free input:** `<input type="text|number|date|email|url">` and `<textarea>`; `placeholder`, `min`, `max`, `pattern`, and `required` are enforced. `number` returns a number; text returns text.
- **Group each question.** Use `<fieldset>` with `<legend>`, or a `<p>` immediately before the group. Wrap every control in a `<label>`.
- **Mark the recommendation by preselection.** Put `checked` on the radio or checkbox, `selected` on the option, or `value` on the text field. The harness reads only that preselection; option order and label text are presentation. A timeout, when the harness enforces one, submits the current values and says no answer arrived; do not read that as consent.
- **Never add an "Other" option or a text field for it.** Radio and checkbox groups get one automatically: its text becomes the radio's value or an extra checkbox array entry. An unchecked choice group or lone checkbox is omitted from the JSON.
- **Context belongs in the form:** `<p>`, `<ul>`, `<table>`, `<pre>`. Pictures go in `<img src="…" alt="…">` — an http(s) URL, a `data:` URI, or a path relative to the working directory — or inline `<svg>`, as their own blocks; inside a `<label>` only the `alt` text shows.
- **Keep it short:** a few questions, 2–5 options each. Ask follow-ups in a later call.

Example:

```html
<form>
  <p>Sessions live in memory and are lost on restart.</p>
  <fieldset>
    <legend>Session store</legend>
    <label><input type="radio" name="store" value="sqlite" checked> SQLite: one file, no new service</label>
    <label><input type="radio" name="store" value="redis"> Redis: shared across instances, needs ops</label>
  </fieldset>
  <label>Session lifetime (days) <input type="number" name="ttl_days" value="30" min="1"></label>
</form>
```

Submitted untouched, this answers `{"store": "sqlite", "ttl_days": 30}`.

## Structured question shape

The table covers the common structured tools. For any other, read its schema; the field names are exact.

| | `question` (opencode) | `ask_user_question` (deepseek-harness) | `AskUserQuestion` (Claude Code) | `question` (pi extension) |
| --- | --- | --- | --- | --- |
| Container | `questions[]`, non-empty | `questions[]` | `questions[]`, one to four | single `question`, no array |
| `id` per question | absent | **required** | absent | absent |
| `header` | **required** | optional | **required** | absent |
| `options[]` | required | optional | required | required |
| `options[].description` | **required** | optional | **required** | optional |
| Multi-select key | `multiple` | `multi_select` | `multiSelect` | absent |
| Marking the recommendation | label suffix, option first | label suffix, option first | no field; the label carries it | prose only |
| Free-text answer | a `Type your own answer` row | returned as `custom` on the answer | an `Other` row, plus a notes field | a `Type something.` row |

Two columns are narrower than they look:

- **Claude Code** requires `header` and `description` by type, has no recommendation field at all, and leaves the free-text row to the app — the CLI shows an `Other` row, an Agent SDK app supplies its own.
- **pi** ships no question tool in its core; the column describes its documented extension example, so treat it as "only if the extension is installed".

## Portable rules

These hold across both shapes, with the shape-specific convention named inline:

- **One question per entry or group.** Use one `questions[]` entry, one `<fieldset>`, or one `<p>`-led group; never fold two questions into one.
- **Keep the round small.** A structured round is 3–4 questions with 2–4 options each; a form is a few questions with 2–5 options each. More is a survey.
- **Label the question and each option.** Structured tools want a short `header` where the schema has one and a one-sentence `description` per option where required; a form wants a `<legend>` and option text that states the tradeoff.
- **Mark the recommendation.** Structured: put it first and end its label with ` (Recommended)`, or fill the tool's dedicated recommendation field where one exists; if it has neither, say it in the question text. Form: preselect it with `checked`/`selected`/`value`.
- **Never add your own free-text option or "Other" row.** Every structured tool and every radio/checkbox group appends one.
- **When the session has no question tool, write the round as prose and say that you did.**

## Facts are the agent's job

A question round carries decisions. Anything checkable — a file, a config value, a version, a count — is looked up before the round is composed, never offered to the owner as a choice. Tag `[INFERENCE]` rather than asserting.
