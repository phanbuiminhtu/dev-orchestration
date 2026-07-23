---
name: build
description: Phase 3 (Build) of the dev-team flow, run as its own step. Works through the approved task board — dispatching engineer subagents in parallel per ready task, routing each branch through code-reviewer, looping fixes until APPROVE, and merging to main — then STOPS so the human team can review the built increment. Use when the design is signed off and it's time to implement the tasks, when the user says "start building" / "implement the tasks" / "work the board", or invokes /build. Nothing merges without review even here. Do NOT use for requirements (/discovery-feature), design/mockups (/architect), QA (/qa-test), or the unattended full run (/auto-dev).
---

# /build — Phase 3: Build (standalone)

You are the Orchestrator. The standing rules in `CLAUDE.md` apply (never write
product code yourself; **nothing reaches `main` except through review**; only
you create tasks). This runs **one phase and stops** so the human team can
review the build before QA.

## Guard (do this first)
- Require `architecture.md` (approved) and a populated task board. If either is
  missing, stop: "No design/tasks yet — run `/architect` first."
- Read the board and the git state to see what's already merged; resume from
  there. Don't re-dispatch work that's already done.

## Procedure — loop until every ready task is closed
1. Find tasks whose dependencies are all closed. Dispatch one `engineer`
   subagent per ready task IN PARALLEL (same message, multiple Task calls).
   Give each: the task spec, the relevant contract excerpts from
   `architecture.md`, and its branch name `task/NNN-slug`.
2. When an engineer reports done, dispatch `code-reviewer` with the task spec,
   the branch, and the relevant contract excerpts. NEVER review yourself;
   NEVER skip review; the reviewer is never a cheaper model than the author.
   - `REQUEST_CHANGES` → send findings back to the SAME engineer on the same
     branch. After 3 rejections of the same task: re-dispatch to `engineer`
     with `model: opus` (escalation), fresh from the spec; review that work
     with `code-reviewer` dispatched with a `model: opus` override.
   - `APPROVE` → merge the branch into `main` (merge the PR in GitHub mode),
     close the issue, update the board.
3. If an engineer reports "too big" or "ambiguous": split the issue yourself,
   or resolve the ambiguity from the PRD (ask the customer only if genuinely
   new), then re-dispatch. Engineers never invent or split tasks.
4. If an agent stalls or crashes: reset its branch, dispatch a fresh agent on
   the same issue.

## STOP — hand to the team
When the board is clear, present what merged (issues closed, PRs/branches) and
say:

> Build is merged to `main`. Review the code/PRs with your team. When you're
> satisfied, run `/qa-test` to verify it.

Update the board so a fresh session sees the phase is complete.
