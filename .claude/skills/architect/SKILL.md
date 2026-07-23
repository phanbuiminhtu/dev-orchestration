---
name: architect
description: Phase 2 (Design) of the dev-team flow, run as its own step. From an approved PRD.md, choose the stack, write architecture.md (components, data flow, DB schema, API contracts), produce clickable HTML mockups of every screen via the ux-designer agent, and break the work into task issues — then STOP so the human team can open the mockups and debate the design before any code. Use when the user wants system design, an architecture doc, wireframes/mockups, a tech-stack decision, or a task breakdown, or invokes /architect. Do NOT use for gathering requirements (/discovery-feature), implementing code (/build), or the unattended full run (/auto-dev).
---

# /architect — Phase 2: Design (standalone)

You are the Orchestrator. The standing rules in `CLAUDE.md` apply (never write
product code yourself; state lives in the repo). This runs **one phase and
stops** so a human dev team can open the mockups and debate the design.

## Guard (do this first)
- Require an approved `PRD.md`. If it's missing or its header shows no approval,
  stop and say: "No approved spec yet — run `/discovery-feature` first." Don't
  design against guesses; that's the whole point of the gate.
- If `architecture.md` already exists: this is a re-run. Read it, summarize, and
  ask whether to revise or start fresh — don't silently overwrite.
- Confirm tracking mode (GitHub issues vs `BOARD.md` + `tasks/`); announce it.

## Procedure
1. Choose the stack — prefer boring, well-documented technology that fits the
   PRD. Write `architecture.md`: components, data flow, DB schema, and exact
   API contracts (routes, methods, request/response shapes). Record the
   reasoning behind the stack choice in plain language.
2. Only if a design decision has genuinely deep trade-offs you're unsure about,
   dispatch ONE general-purpose subagent with `model: fable` presenting the
   decision, the options, and your analysis; incorporate its answer.
3. Dispatch the `ux-designer` agent with the PRD → self-contained HTML mockups
   of every screen in `design/mockups/`. These are what the team opens in a
   browser to critique — the HTML is the deliverable, not a description of it.
4. Break the work into tasks: each completable in one PR, with spec, acceptance
   criteria, and `blocked-by` dependencies. Create the issues (GitHub or
   `tasks/NNN-slug.md` + `BOARD.md`). Commit everything.

## STOP — hand to the team
Do NOT start building. Present: the list of mockup HTML files (with their paths
so the team can open them), the stack choice in plain language, and the task
plan. Then say:

> Design is in `architecture.md`, mockups in `design/mockups/*.html`, tasks on
> the board. Open the mockups, review the plan, and debate with your team. When
> you've agreed, run `/build`. This is the last cheap moment to change course.

When the team approves, record it in the `architecture.md` header
(`Approved: <who> <date>`) so `/build` and a fresh session know design is
signed off.
