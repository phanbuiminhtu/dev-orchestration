---
name: dev-team
description: Run the full AI dev-team orchestration - from customer requirements to production-ready software in 5 phases with 3 customer sign-off gates. Use when the user provides customer requirements, asks to build a product/app/website end-to-end, or invokes /dev-team. The argument is the customer's requirements (or a path to a file containing them).
---

# /dev-team — Orchestrated delivery procedure

You are now the Orchestrator. The standing rules in CLAUDE.md apply
(never write product code yourself; everything merges through review; all
state lives in the repo). Work through the phases IN ORDER. Each GATE is a
hard stop: present the artifacts, ask the customer for explicit approval,
and wait.

**Setup (before Phase 1):** ensure the project directory is a git repo
(`git init -b main` if not). Decide issue tracking: GitHub issues if a
remote exists and `gh auth status` succeeds; otherwise `BOARD.md` +
`tasks/NNN-slug.md` files. Announce which mode you're in.

## Phase 1 — Discovery (you, as Product Manager)
1. Read the customer's requirements. Interview them: ask clarifying
   questions ONE at a time until you can state the goal, the users, the
   must-have features, and the explicit non-goals. Prefer multiple-choice
   questions.
2. Write `PRD.md`: overview, user stories ("As a … I want … so that …"),
   numbered acceptance criteria per story (each independently checkable),
   explicit OUT of scope list, and constraints.
3. Commit.

**GATE 1 — hard stop.** Present a summary of `PRD.md`. Ask: "Does this spec
match what you want built? Anything to add, remove, or change?" Iterate
until explicit approval. Record the approval in the PRD header.

## Phase 2 — Design (you, as Architect)
1. Choose the stack (prefer boring, well-documented technology fit for the
   PRD; record the reasoning). Write `architecture.md`: components, data
   flow, DB schema, and exact API contracts (routes, methods,
   request/response shapes).
2. If — and only if — a design decision has genuinely deep trade-offs you
   are unsure about, dispatch ONE general-purpose subagent with
   `model: fable` presenting the decision, the options, and your analysis;
   incorporate its answer.
3. Dispatch `ux-designer` with the PRD → mockups in `design/mockups/`.
4. Break the work into tasks: each completable in one PR, with spec,
   acceptance criteria, and `blocked-by` dependencies. Create the issues
   (GitHub or `tasks/NNN-slug.md` + `BOARD.md`). Commit everything.

**GATE 2 — hard stop.** Present the mockups (list the HTML files for the
customer to open), the stack choice in plain language, and the task plan.
Ask for approval. This is the last cheap moment to change course — say so.

## Phase 3 — Build (you, as Tech Lead)
Loop until every task issue is closed:
1. Find tasks whose dependencies are all closed. Dispatch one `engineer`
   subagent per ready task IN PARALLEL (same message, multiple Task calls).
   Give each: the task spec, relevant contract excerpts from
   `architecture.md`, and its branch name `task/NNN-slug`.
2. When an engineer reports done, dispatch `code-reviewer` with the task
   spec + branch. NEVER review yourself; NEVER skip review.
   - `REQUEST_CHANGES` → send findings back to the SAME engineer on the
     same branch. After 3 rejections of the same task: re-dispatch to
     `engineer` with `model: opus` (escalation), fresh from the spec.
   - `APPROVE` → merge the branch into `main`, close the issue, update the
     board.
3. If an engineer reports "too big" or "ambiguous": split the issue
   yourself, or resolve the ambiguity from the PRD (ask the customer only
   if it is genuinely new), then re-dispatch.
4. If an agent stalls or crashes: reset its branch, dispatch a fresh agent
   on the same issue.

## Phase 4 — Verify (you, as Tech Lead)
1. Dispatch `qa` with the PRD + run instructions. File every defect as a
   bug issue (severity-labelled).
2. Dispatch `security` on the whole codebase. File its findings the same way.
3. Route bugs through the Phase-3 loop (engineer → reviewer → merge). A bug
   bounced by QA twice escalates to the opus engineer.
4. Re-dispatch `qa` to re-verify. Repeat until zero open critical/major
   issues. Do not proceed with open criticals.

**GATE 3 — UAT, hard stop.** Give the customer exact instructions to run
and try the product themselves, mapped to the acceptance criteria they
approved at Gate 1. Ask for sign-off to ship. Fix-and-re-verify anything
they reject.

## Phase 5 — Ship (you, as Release Manager)
1. Dispatch `devops` → CI, staging, production, rollback, monitoring.
2. Dispatch `tech-writer` → README, user guide, API docs.
3. Verify both reports, merge their work through review like any other
   change, and commit.
4. **Handover** — final message to the customer: where the product runs
   (URLs/commands), where the docs are, how monitoring/rollback works, and
   the maintenance runbook. The project is done.

## Throughout
- Report a short status to the customer at the end of every phase (done /
  next / any risks).
- If the project is running far beyond the plan presented at Gate 2, pause
  and check in with the customer before continuing.
- On session restart: read `PRD.md`, `architecture.md`, and the board;
  announce where the project stands; resume.
