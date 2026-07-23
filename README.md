# AI Dev-Team Kit

A portable Claude Code configuration that runs an AI development team —
from customer requirements to production-ready software — with you (or your
customer) approving at exactly three gates.

Design spec: `docs/superpowers/specs/2026-07-16-ai-dev-team-orchestration-design.md`

## How to use

1. **Copy this folder's contents into a new, empty project directory**
   (must include the hidden `.claude/` directory and `CLAUDE.md`):
   ```bash
   mkdir -p ~/projects/my-new-product && cp -r CLAUDE.md .gitignore .claude README.md ~/projects/my-new-product/
   ```
2. **Start Claude Code there on an Opus-tier model** (the session model IS
   the Orchestrator):
   ```bash
   cd ~/projects/my-new-product && claude --model opus
   ```
3. **Kick off Phase 1:**
   ```
   /discovery-feature The customer wants a booking website for their yoga studio: ...
   ```
   (Or `/discovery-feature requirements.md` if the requirements are in a file.)
4. **Work phase by phase.** Each command does one phase and STOPS so your team
   can review and debate before moving on. Run them in order:
   - `/discovery-feature` → the spec (PRD)
   - `/architect` → HTML mockups + architecture + task plan
   - `/build` → engineer → review → merge loop
   - `/qa-test` → QA + security, fix loop, then your team's UAT
   - `/ship` → deploy + docs + handover
5. Receive the handover: running product, docs, runbook.

**Hands-off alternative:** once your team is happy to let the AI finish on its
own, run `/auto-dev` — it resumes from wherever the repo is and drives every
remaining phase to completion (auto-merge through review, auto-deploy to prod)
with no stops. Start it and walk away.

## What's inside

| Path | What it is |
|---|---|
| `CLAUDE.md` | Orchestrator standing rules: roles, model tiers, review loop, failure handling |
| `.claude/agents/` | 7 worker agents with pinned models (haiku engineers, sonnet reviewers/QA/security/devops/UX, haiku tech-writer) |
| `.claude/skills/*/` | Six phase commands: `/discovery-feature`, `/architect`, `/build`, `/qa-test`, `/ship` (one phase each, stop for team review), plus `/auto-dev` (unattended full run) |

Escalation (stuck tasks → Opus) and the Fable advisor are dispatched by the
Orchestrator with model overrides — no extra files needed.

## Notes

- Works with or without GitHub: with a remote + `gh` it uses real issues and
  PRs; without, it keeps a `BOARD.md` + `tasks/` files in git. Either way,
  full project state lives in the repo and survives crashes.
- Scope: designed for web apps and APIs.
- In Phase 5 the tech-writer replaces this README with the product's own
  README — that's intended; this file's job ends at kickoff.
