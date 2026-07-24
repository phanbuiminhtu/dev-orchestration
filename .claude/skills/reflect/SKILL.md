---
name: reflect
description: Post-ship retrospective for the dev-team flow. Mines the project's own history — review rejections, QA bounce patterns, escalations, ambiguities that reached the customer, schedule vs the Gate-2 plan — and distills the lessons into LESSONS.md so the next project's phases start smarter. Use after /ship completes, at the end of a project or milestone, or when the user says "retro", "retrospective", "what did we learn", "post-mortem the project", or invokes /reflect. Do NOT use for a live production incident (that's an incident post-mortem, handled ad hoc) or for QA of the product itself (/qa-test).
---

# /reflect — post-ship retrospective

You are the Orchestrator running the team's retro. The output is
`LESSONS.md` — and its only purpose is to change behavior next time. A
lesson that doesn't name what to do differently is trivia; leave it out.

## Where the evidence is (read, don't ask)
- The board / closed issues: which tasks bounced in review, how many times,
  what the rejection reasons were. Which bugs QA bounced twice.
- Escalations: which tasks needed the opus engineer, and why.
- `git log`: task sizing (how many PRs per task issue vs the one-PR rule),
  rework commits after merge.
- `PRD.md` history: which requirements changed after Gate 1 — those were
  discovery misses.
- `DECISIONS.md` (if `/auto-dev` ran): which best-guesses the team later
  overturned.
- The Gate-2 plan vs what actually happened: schedule/scope drift.

## Write LESSONS.md
```
# Lessons — <project> — <date>

## What repeatedly went wrong
<pattern → evidence (task numbers) → root cause, max 5 items>

## What to change next project
<each: the concrete change + WHERE it lives — a line in CLAUDE.md, a task
sizing rule, a review checklist item, a PRD template question>

## What worked — keep doing
<max 3, only if genuinely notable>
```

Commit it (orchestrator-authored document — direct commit).

## Close the loop — this is the point
For each "what to change" item that belongs in the kit itself (CLAUDE.md
standing rules, an agent's instructions, a phase skill), propose the exact
edit to the user. Apply the ones they approve. A retro that only produces
a file nobody reads is theater; the edits are the deliverable.
