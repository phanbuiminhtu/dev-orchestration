---
name: discovery-feature
description: Phase 1 (Discovery) of the dev-team flow, run as its own step. Turns customer/stakeholder requirements into an approved PRD.md, then STOPS so a human dev team can review and debate the spec before design. Use whenever the user hands over product requirements, asks for a spec/PRD/requirements doc, wants to scope a new feature or app, or invokes /discovery-feature — even if they don't say "PRD". The argument is the requirements themselves or a path to a file containing them. Do NOT use for designing architecture (that's /architect), writing code (/build), or running the whole pipeline unattended (/auto-dev).
---

# /discovery-feature — Phase 1: Discovery (standalone)

You are the Orchestrator. The standing rules in `CLAUDE.md` apply (never write
product code yourself; state lives in the repo). This runs **one phase and
stops** so a human dev team can review and debate the spec before design starts.

## Guard (do this first)
- Ensure the directory is a git repo (`git init -b main` if not).
- Decide issue tracking: GitHub issues if a remote exists and `gh auth status`
  succeeds; otherwise the file board (`BOARD.md` + `tasks/`). Announce the mode.
- If `PRD.md` already exists: this is a re-run. Read it, summarize what's there,
  and ask whether to revise it or start fresh — don't silently overwrite.
- If no requirements were given as the argument: ask the user for them (or the
  path to a file) before continuing.

## Procedure
1. Read the requirements. Interview the customer: ask clarifying questions
   ONE at a time (prefer multiple-choice) until you can clearly state the
   goal, the users, the must-have features, and the explicit non-goals.
2. Write `PRD.md`: overview, user stories ("As a … I want … so that …"),
   numbered acceptance criteria per story (each independently checkable),
   an explicit OUT-of-scope list, and constraints.
3. Commit `PRD.md` directly (orchestrator-authored document — no review loop).

## STOP — hand to the team
Do NOT proceed to design. Present a short summary of `PRD.md` and say:

> Spec is in `PRD.md`. Review and debate it with your team. When you've agreed,
> run `/architect` to design it — or edit `PRD.md` and re-run `/discovery-feature`
> to revise.

When the team approves, record it in the PRD header
(`Approved: <who> <date>`) so a later phase / fresh session knows discovery is
signed off. That header line is the checkpoint — without it, downstream phases
can't tell an approved spec from a draft.
