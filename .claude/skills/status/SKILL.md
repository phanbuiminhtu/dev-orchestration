---
name: status
description: One-page project status report for the dev-team flow — current phase, board state, open bugs by severity, what's blocked, what's in review, recent merges, and approximate cost/effort so far. Use whenever the user or their team asks "where are we", "what's the status", "what's left", "what's blocking", wants a stand-up summary before a gate debate, or invokes /status. Read-only: reports state, changes nothing, dispatches no agents. Do NOT use to advance a phase (that's the phase skills) or to file/close work items.
---

# /status — project status report (read-only)

You are the Orchestrator reporting to the team. Read state, report it,
change nothing. Dispatch no agents — this must be fast and free.

## Where to read
- `PRD.md` / `architecture.md` headers → which gates are approved.
- The board: GitHub issues (`gh issue list`) if in GitHub mode, else
  `BOARD.md` + `tasks/`.
- `git log --oneline` since the last phase marker → recent merges.
- Open branches (`git branch -a`) → work in flight.
- `DECISIONS.md` if present → autonomous decisions awaiting team review.

## Report — always this exact shape
```
# Status — <project> — <date>

**Phase:** <1-5 name> (<approved gates>)
**Done:** <closed tasks / merged branches, one line each, most recent first>
**In flight:** <task → engineer/review state, e.g. "task/012 in review, 1 rejection">
**Blocked:** <task → what blocks it, or "nothing">
**Bugs:** <critical: N, major: N, minor: N — list critical/major by title>
**Decisions awaiting review:** <from DECISIONS.md, or "none">
**Next:** <the single next action, e.g. "team review of mockups, then /build">
```

Keep it to one screen. If the repo has no PRD/board at all, say the project
hasn't started and point at `/discovery-feature` — don't invent status.

## Cost note
If the session has visible token/cost usage for this project, append one
line (`**Cost so far:** ~<estimate>`). If you can't estimate honestly,
omit the line — no made-up numbers.
