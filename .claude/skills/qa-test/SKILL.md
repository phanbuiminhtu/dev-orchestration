---
name: qa-test
description: Phase 4 (Verify) of the dev-team flow, run as its own step. Runs the qa agent (integration + acceptance testing against the PRD's acceptance criteria, actively trying to break the product) and the security agent (whole-codebase vulnerability pass), files every defect as a bug issue, routes fixes back through the engineer→review→merge loop, and re-verifies until zero critical/major issues remain — then STOPS so the human team can try the product themselves (UAT). Use when the build is merged and needs testing/QA/security review, when the user says "test it" / "QA this" / "security review the app" / "run acceptance tests", or invokes /qa-test. Do NOT use for implementing features (/build), deploying (/ship), or the unattended full run (/auto-dev).
---

# /qa-test — Phase 4: Verify (standalone)

You are the Orchestrator. The standing rules in `CLAUDE.md` apply (never write
product code yourself; every fix merges through review; only you file work
items — workers file findings, you turn them into bug issues). This runs **one
phase and stops** so the human team can do their own UAT.

## Guard (do this first)
- Require a merged build (build tasks closed on the board / merged to `main`).
  If the app isn't built yet, stop: "Nothing to verify — run `/build` first."
- Read the board for any already-open bugs; fold them into this round rather
  than duplicating.

## Procedure
1. Dispatch the `qa` agent with the PRD + run instructions. File every defect
   it reports as a bug issue (severity-labelled: critical / major / minor).
2. Dispatch the `security` agent on the whole codebase. File its findings the
   same way.
3. Route bugs through the build loop (engineer → `code-reviewer` → merge). A bug
   bounced by QA twice escalates to the `engineer` with `model: opus`, reviewed
   by `code-reviewer` with a `model: opus` override.
4. Re-dispatch `qa` (passing the list of previously-filed bugs) to re-verify.
   Re-dispatch `security` only on security-relevant fixes. Repeat until zero
   open critical or major issues — both block sign-off.

## STOP — hand to the team
When critical/major issues are clear, present: what QA and security found, what
was fixed, and any remaining minor issues. Then give the team **exact steps to
run and try the product themselves**, mapped to the PRD's acceptance criteria,
and say:

> QA and security are clear of critical/major issues. Try the product yourselves
> using the steps above and check it against the acceptance criteria. When you
> sign off, run `/ship`. Anything you reject, tell me and I'll fix-and-re-verify.

Record the verification status on the board so `/ship` and a fresh session know
Phase 4 is clear.
