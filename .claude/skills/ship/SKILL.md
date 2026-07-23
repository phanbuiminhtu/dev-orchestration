---
name: ship
description: Phase 5 (Ship) of the dev-team flow, run as its own step. After the product passes QA/UAT, dispatches the devops agent (CI pipeline, staging deploy, production deploy, rollback plan, monitoring) and the tech-writer agent (README, user guide, API docs), merges their work through review, then delivers the handover — where it runs, where the docs are, how rollback/monitoring work. Use when a verified product is ready to release/deploy/document, when the user says "ship it" / "deploy" / "release" / "write the docs and hand off", or invokes /ship. Deploys where the project's deploy target/credentials are already configured. Do NOT use for building features (/build), QA (/qa-test), or the unattended full run (/auto-dev).
---

# /ship — Phase 5: Ship (standalone)

You are the Orchestrator. The standing rules in `CLAUDE.md` apply (never write
product code yourself; the devops/tech-writer output merges through review like
any other change). This is the final phase — it ends with a handover, not a
stop-and-wait.

## Guard (do this first)
- Require Phase 4 clear (no open critical/major bugs on the board). If QA hasn't
  passed, stop: "Not verified yet — run `/qa-test` first."
- Check the deploy target + credentials are actually configured (e.g. Vercel
  token, Fly config, AWS creds — whatever the stack needs). If deploy isn't
  wired, do everything you can (CI, docs, staging if reachable) and stop before
  production with a clear note: "Production deploy blocked — provide X." Don't
  pretend to deploy to a target that doesn't exist.

## Procedure
1. Dispatch the `devops` agent → CI pipeline, staging deploy, production deploy,
   rollback plan, and monitoring. Prefer staging → smoke test → production so a
   bad build never lands in prod unchecked.
2. Dispatch the `tech-writer` agent → README, user guide, and API docs, written
   from the PRD and the final merged code.
3. Verify both reports; merge their work through `code-reviewer` review; commit.

## Handover — deliver to the team
Final message: where the product runs (URLs/commands), where the docs live, how
monitoring and rollback work, and the maintenance runbook. The project is done.
