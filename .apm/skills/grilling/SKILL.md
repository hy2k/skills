---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

**Put the round through the harness's question tool whenever the session has one** — `ask`, `question`, `ask_user_question` or `AskUserQuestion` — rather than writing it as prose. One question per entry or group; the shape, field names, recommendation convention and portable rules are in `docs/agents/asking.md` when the repo carries one, and in the tool's own description otherwise. The markdown format below is the fallback for a session with no such tool, and for the record the round leaves in a transcript.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

→ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

→ <your recommended answer>
```

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

**When a decision needs the reason, not just the pick, ask for it explicitly.** In a form, add a named free-text field next to the options; with a structured tool, ask a follow-up. A rationale belongs to the question, not to a single option, even when the host lets the owner annotate one choice.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.

## Recording

Once the user confirms, record what settled: a decision that beat live alternatives as an ADR, a newly named term in the glossary. Where those files go comes from `docs/agents/artifact-paths.md` when the repo carries one; otherwise `docs/adr/NNNN-slug.md` and the glossary at the repo root.

Skip anything the repo's own rules say needs no record — a choice with one sane answer, something the code already settles, or a sentence a spec states is not an ADR. And never write a record for a question a later round may reopen: a record answering a question the session has already moved past is worse than no record.

When the conversation produced a plan rather than decisions — something to build next — the output is a spec or tickets, not an ADR. Say which and stop there.
