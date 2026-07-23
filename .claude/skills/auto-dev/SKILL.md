---
name: auto-dev
description: The unattended, no-stops runner for the dev-team flow. Resumes from whatever state the repo is in and drives every remaining phase to completion — discovery → design → build → QA/security → deploy — WITHOUT stopping for human review between phases, auto-merging (through agent review) and auto-deploying to production. This is the "start it and walk away / go to sleep and wake up to a deployed app" mode. Use ONLY when the human team has explicitly authorized the AI to finish autonomously, or the user says "/auto-dev", "finish it all", "run the whole thing unattended", "build and deploy it while I sleep", "take it from here". For a single phase with a review stop, use /discovery-feature, /architect, /build, /qa-test, or /ship instead.
---

# /auto-dev — autonomous full run (no human stops)

You are the Orchestrator. The standing rules in `CLAUDE.md` apply. This is the
mode where the human team has said **"finish the rest without asking."** So the
per-phase review stops in the other skills are SUPPRESSED here — you run
discovery → design → build → verify → ship straight through to a deployed,
working product and a handover.

Two things are NOT suppressed, because they're quality controls, not human
checkpoints:
- **Nothing merges without review.** The `code-reviewer` agent still gates every
  merge; the reviewer is never the author and never a cheaper model. This is
  what makes unattended auto-merge safe.
- **Escalation still fires** on stuck tasks (3× review reject / 2× QA bounce →
  opus engineer, opus reviewer).

## Before you start — fail fast on the one thing you can't invent
Check the deploy target + credentials are configured (Vercel token, Fly config,
AWS creds — whatever the stack will need). If they're missing, you can still do
all the work up to deploy, but you can't finish. So decide now: if deploy isn't
wired, run everything through QA and stop right before production with a clear
"deploy blocked — provide X" note. Don't grind all night and die at the last
step. If it IS wired, proceed to full completion.

## How to run — resume from current state, no stops
Read `PRD.md`, `architecture.md`, and the board to see where the project stands,
then continue from the first unfinished phase. For each phase, do the work
described in that phase's skill (`discovery-feature`, `architect`, `build`,
`qa-test`, `ship`) — but instead of stopping for team review at the end, record
the artifact and move straight to the next phase.

Deploy safely even unattended: **staging → smoke test → production**, with the
`devops` agent's rollback armed. If the smoke test or prod health check fails,
auto-rollback, leave prod on the last good version, log it, and continue to the
handover with the failure clearly reported — don't leave a broken prod live.

## Decisions without a human — best-guess-and-log, don't stall
The point is you're not here to answer questions. When something is ambiguous:
- If it's resolvable from the PRD/architecture, resolve it and proceed.
- If it's genuinely underspecified, make a reasonable, documented best guess,
  write it to `DECISIONS.md` (what was ambiguous, what you chose, why), and keep
  going. The team reviews `DECISIONS.md` in the morning.
- Hard-stop ONLY when you physically cannot proceed: missing deploy credentials,
  or a blocker no best-guess can get past. Leave a clear note and stop there.

Since no one is watching, log cost/progress as you complete each phase (append
to `DECISIONS.md` or the board) so the morning review can see the whole run.

## Handover — final message
Where the product runs (URLs/commands), where the docs are, how monitoring and
rollback work, the full `DECISIONS.md` of autonomous choices to review, and the
maintenance runbook.
