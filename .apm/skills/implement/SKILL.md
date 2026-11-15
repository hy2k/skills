---
name: implement
description: "Implement a piece of work based on a spec or set of tickets — sequentially on the current branch, or in parallel across worktrees when the user asks for concurrency."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

Use `tdd` where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end. Where the repo carries `docs/agents/quality-gates.md`, its commands and its full-versus-targeted rule win over this sentence — and a gate that did not run is reported as not run.

Once done, use `code-review` to review the work.

Commit your work to the current branch.

## Mode

Sequential is the default and covers every ordinary case: work one ticket at a time, on the current branch, one commit per ticket.

When tickets carry `Recommended order`, run them in that sequence, one at a time, one writer per shared surface. The line is advisory, not a blocker, but a shared surface is a mutex: two sessions on it produce conflicts, stale generated output, or two writers on one service.

The set the user named is the scope: the spec's tickets, or the single ticket they point at. Work the frontier of that set only, and stop when no open ticket in it remains. Do not pick up tickets from another feature directory or another parent issue, even when they are unblocked; name them to the user instead.

Switch to parallel mode only when the user asks for it in words — "in parallel", a named worktree branch, an integration branch, or a PR — or when they point at a branch that is already a worktree. **"All tickets" or "the whole spec" is not a request for concurrency**: it means implement every one of them, sequentially, unless concurrency was asked for separately.

Never choose parallel mode on your own. Worktrees, merge commits and a review pass on an integration branch change the shape of history, and that is the user's call to make. If the user does ask for it on a set where tickets carry `Recommended order` or share a surface, name the shared surface and get an explicit override before starting.

## Parallel mode

Read the tracker doc from `docs/agents/issue-tracker.md`. If it is missing, tell the user to run the `agent-conventions` skill.

The goal is the entire spec implemented on a single **integration branch**, with every ticket resolved the way the issue tracker closes work.

The tickets are not a list of steps. They are a **task graph** with blocking relationships between them, so there is always a **frontier** of tickets ready to be grabbed.

Communication to and from subagents should be sparse. Communicate primarily through **context pointers**: to the spec, tickets, research notes, and previous commits. Don't duplicate information already available via pointers.

**Implementer subagents** should be run in the background where possible for maximum concurrency.

### Steps

1. Read the spec and tickets to understand the task graph.

2. (optional) Use an **exploration subagent** to conduct any exploration required by the tickets — relevant codebase files or external documentation. It should save its markdown notes in `$TMPDIR` (`.tmp/` in the checkout), never in the repo, so implementer subagents can read them without committing them.

3. Create the integration branch. If the issue tracker closes work through PRs, or the user asks for one, open a draft PR after the first merge in step 5 (a branch with no commits ahead of main can't open one), marked as closing the spec and tickets.

4. Use **implementer subagents** to implement each ticket, each in its own worktree on its own branch. Each implementer subagent:
   - confirms its worktree is based on the integration branch before starting, and resets onto it if not;
   - calls `tdd` to build the ticket;
   - merges the integration branch tip into its own branch before reporting done.

5. Once an **implementer subagent** completes, merge its work to the integration branch with a **merger subagent**.

6. If this changes the **frontier** of available tickets, kick off more **implementer subagents** to work on the new tickets. This allows for maximum concurrency. Where the set carries `Recommended order`, start the next ticket in that sequence before any later one that shares its surface.

7. Once all tickets are complete, call `code-review` on the integration branch. Fix all issues raised by the code review in a single **implementer subagent**.

8. If a draft PR exists, mark it ready for review. Otherwise, resolve each ticket the way the issue tracker closes work, and report the integration branch.

9. Clean up all **implementer subagent** worktrees.
