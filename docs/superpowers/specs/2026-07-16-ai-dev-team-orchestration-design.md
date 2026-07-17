# AI Dev Team Orchestration — Design Document

**Date:** 2026-07-16
**Status:** Approved design (spec only — no implementation yet)
**Companion visual:** https://claude.ai/code/artifact/bdbaa673-626b-4847-b6bd-92b0114b0d3a

---

## 1. Purpose

An orchestration of AI agents that operates like a professional software team
(modeled on Google's engineering practices): it takes a customer from an initial
conversation to production-ready, deployed, documented software, with the
customer involved only at three sign-off gates.

## 2. Scope

**In scope:** Web applications and APIs — websites, web apps, backend services,
dashboards. This covers the large majority of customer requests and has a
well-understood build/test/deploy pipeline.

**Out of scope (for this design):** Mobile apps, desktop software, games, data
pipelines. These can be added later as additional "deployment profiles" without
changing the core orchestration.

**Platform:** Claude models via Claude Code subagents / the Claude Agent SDK.
Concrete mechanisms in this document assume that platform.

## 3. Architecture: hierarchical team inside a fixed pipeline

Two structural decisions, chosen after comparing alternatives:

1. **The five phases run as a fixed, deterministic sequence** (assembly line):
   Discovery → Design → Build → Verify → Ship, with customer gates between
   them. This is what makes the process predictable and professional.
2. **Inside Build and Verify, a hierarchical team fans out in parallel**
   (orchestrator + workers), mirroring a real engineering team. This is what
   makes it fast and keeps each agent's context small and focused.

Rejected alternatives:
- *Pure pipeline* (one agent per stage): no parallelism, no independent code
  review, one context window holding a whole codebase degrades quality.
- *Free-form swarm* (agents self-organize): unreliable in practice — agents
  duplicate work, talk in circles, and nobody is accountable for shipping.

## 4. The five phases and three gates

| Phase | Name | Owner (hat) | Key activities | Outputs |
|---|---|---|---|---|
| 1 | **Discovery** | Orchestrator as *Product Manager* | Customer interview; extract needs (not just wants); define scope and explicit non-goals | `PRD.md`, user stories, acceptance criteria |
| — | **GATE 1** | Customer | Approves the spec (PRD) before anything is designed | Sign-off |
| 2 | **Design** | Orchestrator as *Architect* | Tech-stack choice, data flow, DB schema, API contracts; task breakdown; UX Agent produces mockups; Fable Advisor consulted for genuinely hard decisions | `architecture.md`, API contracts, DB schema, mockups, GitHub issues (one per task, with dependencies) |
| — | **GATE 2** | Customer | Approves mockups + plan — the last cheap moment to change course | Sign-off |
| 3 | **Build** | Orchestrator as *Tech Lead* | Dispatches tasks to parallel Engineer agents; every PR reviewed by an independent Reviewer; Orchestrator merges. **The Orchestrator never writes code itself.** | Merged, reviewed code with unit tests |
| 4 | **Verify** | Orchestrator as *Tech Lead* | QA Agent tests acceptance criteria and edge cases; Security Agent audits; findings become bug issues routed back through the Build loop. Nothing ships with open critical issues. | QA report, security report, fixed bugs |
| — | **GATE 3** | Customer | UAT — customer tries the real product against the Gate-1 spec | Sign-off = permission to ship |
| 5 | **Ship** | Orchestrator as *Release Manager* | DevOps Agent: CI, staging deploy, production deploy, rollback plan, monitoring; Tech Writer Agent: user guide, README, API docs | Production URL, staging URL, docs, runbook, monitoring dashboard |

"Production-ready" specifically means Phase 5 exists: deployment with rollback,
documentation, and monitoring/handover.

## 5. The team and model assignment

Four model tiers, each earning its seat by cost/capability
(prices per 1M tokens, input/output):

| Model | Price | Roles | Rationale |
|---|---|---|---|
| **Fable 5** | $10 / $50 | Advisor only (Phase 2) | Deepest reasoning; consulted only where it pays for itself |
| **Opus 4.8** | $5 / $25 | Orchestrator (all phases); Escalation Engineer | Best long-horizon agentic execution per dollar — the ideal Tech Lead; also the "senior dev" for tasks that beat the juniors |
| **Sonnet 5** | $3 / $15 | Reviewers, QA, Security, DevOps, UX | Near-Opus quality at 60% of the price; the review layer is the one place never to save money |
| **Haiku 4.5** | $1 / $5 | Engineers ×N, Tech Writer | High-volume, narrowly-scoped work; safe *because* every line passes Sonnet review |

Role definitions:

- **Orchestrator (Opus 4.8)** — one persistent agent wearing four hats across
  the project: Product Manager (P1), Architect (P2), Tech Lead (P3–P4),
  Release Manager (P5). Dispatches, unblocks, merges, talks to the customer.
  Never writes code. Consults the Fable Advisor for hard design decisions.
- **Engineer Agents ×N (Haiku 4.5)** — each owns exactly one task: implement
  it plus unit tests. Fresh context per task; knows only its task spec and the
  API contracts. Run in parallel.
- **Reviewer Agents (Sonnet 5)** — review every PR. Two invariants: the
  reviewer is never the agent that wrote the code, and never a cheaper model
  than the author.
- **Escalation Engineer (Opus 4.8)** — takes over any task rejected in review
  3× or any bug QA bounces twice.
- **QA Agent (Sonnet 5)** — integration tests, edge cases, verifies every
  acceptance criterion from the PRD; actively tries to break the product.
- **Security Agent (Sonnet 5)** — injection, auth flaws, secrets in code,
  vulnerable dependencies.
- **DevOps Agent (Sonnet 5)** — CI pipeline, staging → production deploys,
  rollback plan, monitoring and alerts.
- **Tech Writer Agent (Haiku 4.5)** — user guide, README, API docs from the
  PRD and final code.
- **UX Agent (Sonnet 5)** — wireframes/mockups in Phase 2 for Gate 2.

Deliberately **not** created: a separate Project Manager agent (that is the
Orchestrator — splitting it creates two bosses) and a Debugger agent (bugs are
just new tasks through the same loop).

## 6. Coordination: GitHub as shared memory

Agents never talk to each other directly. All coordination flows through the
repository, giving auditability (anyone can watch the board like a real team's)
and crash recovery (all state lives in GitHub, none in agent memory).

| GitHub action | Who | When |
|---|---|---|
| Create task issues | Orchestrator only | End of Phase 2 — spec, acceptance criteria, dependencies (`blocked by #N`), labels |
| Create bug issues | QA & Security agents | Phase 4 — repro steps, severity, linked acceptance criterion |
| Open PRs | Engineer agents | One issue = one branch = one PR (`Closes #N`) |
| Review / approve | Reviewer agents | On every PR |
| Merge PRs | Orchestrator only | After reviewer approval — merge rights belong to the Tech Lead alone |
| Board, milestones, labels | Orchestrator | Repo setup at start of Phase 3 |

Governing rule: **the Orchestrator is the only agent that creates *work*
(task issues); workers create only *findings* (bug issues).** This prevents
scope creep with no one accountable.

### The review loop (every change, no exceptions)

```
Tech Lead assigns issue → Engineer (Haiku) → opens PR → Reviewer (Sonnet)
    ✓ approve            → Orchestrator merges
    ✗ changes requested  → back to the same Engineer, loop until approved
    ⤴ rejected 3×        → task reassigned to the Escalation Engineer (Opus)
```

## 7. Failure-handling rules

| Failure | Rule |
|---|---|
| Reviewer rejects a PR 3× | Task escalates to the Opus Escalation Engineer |
| QA bounces the same bug 2× | Same escalation path |
| Agent stalls or crashes mid-task | Orchestrator detects inactivity timeout → kills the agent, resets the branch, dispatches a fresh agent on the same issue. GitHub state means nothing is lost |
| Task too big for one PR | Engineer reports it → Orchestrator splits the issue into sub-issues. Engineers never split work themselves |
| Requirements ambiguity mid-build | Agent flags it → Orchestrator answers from the PRD, or asks the customer if genuinely new, then updates the PRD. Agents never guess |
| Customer changes mind mid-build | Change request → Orchestrator writes an impact assessment (what's affected, what gets redone) → customer approves → PRD and affected issues revised. Never silently absorbed |
| Orchestrator crashes | A new Orchestrator boots, reads PRD + architecture docs + GitHub state (open issues = remaining work, open PRs = work in review), resumes |
| Budget | Hard token/cost cap per project; at 80% consumption the Orchestrator must report status to the customer before continuing |

## 8. Testing strategy (of the product being built)

- **Unit tests** — written by the Engineer alongside each task; reviewed with the PR.
- **CI** — automated test run on every PR; a red build blocks merge.
- **Integration/acceptance tests** — QA Agent in Phase 4, mapped 1:1 to the
  PRD's acceptance criteria.
- **Security review** — one whole-codebase pass in Phase 4.
- **UAT** — the customer, at Gate 3, against the spec they approved at Gate 1.

## 9. Success criteria for the orchestration itself

1. A customer with no technical background can go from conversation to a
   deployed product with exactly three approval interactions (plus answering
   clarifying questions).
2. Every line of production code was reviewed by an agent that didn't write it.
3. The full project state is reconstructible from the repository alone.
4. No critical QA or Security finding is open at ship time.
5. Cost stays within the per-project budget cap, with the 80% checkpoint honored.

## 10. Future extensions (explicitly deferred)

- Additional deployment profiles: mobile (app-store pipeline, device testing),
  desktop, CLI tools.
- Multiple concurrent customer projects sharing an agent pool.
- Post-ship maintenance mode (monitoring alerts → auto-filed bug issues → the
  same Build/Verify loop).
- Metrics dashboard: cost per project, review rejection rates, escalation rates.
