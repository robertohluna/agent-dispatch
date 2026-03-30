# Agent Roster

> 9 specialized agents for coordinated development sprints

## Roster

| Agent | Codename | Domain | Default Territory |
|-------|----------|--------|-------------------|
| **O** | [ORCHESTRATOR](orchestrator.md) | Sprint Command | Sprint planning, dispatch, monitoring, grading |
| **A** | [BACKEND](backend.md) | Backend Logic | Handlers, services, API routes, middleware |
| **B** | [FRONTEND](frontend.md) | Frontend UI | Routes, components, stores, styling |
| **C** | [INFRA](infra.md) | Infrastructure | Docker, CI/CD, builds, env config |
| **D** | [SERVICES](services.md) | Specialized Services | Integrations, workers, external APIs, ML |
| **E** | [QA](qa.md) | QA / Security | Tests, security audits, dependency scanning |
| **F** | [DATA](data.md) | Data Layer | Models, storage, migrations, data integrity |
| **G** | [LEAD](lead.md) | Merge Authority | Sequential merge, post-merge validation, ship decisions |
| **H** | [DESIGN](design.md) | Design & Creative | Design system, tokens, a11y, visual specs |
| **R** | [RED TEAM](red-team.md) | Adversarial Review | Read-only on all code, write to findings + tests |

## Wave Structure

```
Wave 0: ORCHESTRATOR                   (plan, generate docs, dispatch)
Wave 1: DATA, QA, INFRA, DESIGN       (foundation)
Wave 2: BACKEND, SERVICES              (backend logic)
Wave 3: FRONTEND                       (frontend)
Wave 4: RED TEAM                       (adversarial review of all branches)
Wave 5: LEAD                           (merge + ship, informed by RED TEAM findings)
Wave 6: ORCHESTRATOR                   (grade agents, assess sprint, close)
```

## Merge Order

```
0. ORCHESTRATOR — does not merge; produces dispatch plan (before) and sprint assessment (after)
1. DATA         — migrations and models first
2. INFRA        — build/CI next
3. QA           — test infrastructure
4. DESIGN       — design tokens and specs
5. SERVICES     — integrations and workers
6. BACKEND      — API and business logic
7. FRONTEND     — UI (depends on backend APIs + design tokens)
8. RED TEAM     — does not merge; produces findings report
9. LEAD         — executes the merge sequence, validates after each
```

## Scaling

| Size | Agents | Use When |
|------|--------|----------|
| **Minimal** (2) | ORCHESTRATOR + 1 agent | Single-domain fix, ORCHESTRATOR plans and grades |
| **Small** (4) | ORCHESTRATOR, BACKEND, FRONTEND, LEAD | Simple sprints, few chains |
| **Medium** (6) | ORCHESTRATOR, BACKEND, FRONTEND, QA, DATA, LEAD | Most sprints |
| **Full** (10) | All agents including ORCHESTRATOR | Complex sprints, security-sensitive work |
| **Beyond** (11+) | Split roles into sub-agents | 15+ chains across subsystems |

ORCHESTRATOR is always present. Even in small sprints, it creates the plan and grades the output.

For nested team architecture, sub-agent spawning rules, and merge strategy at 30+ agents, see [scaling.md](../scaling/scaling.md).

## Communication Protocol

Agents communicate through completion reports — no direct agent-to-agent messaging.

```
ORCHESTRATOR creates sprint plan + agent task docs
  -> Operator dispatches agents
  -> Agents execute chains
  -> Agents write completion reports (sprint-XX/agent-X-completion.md)
  -> RED TEAM reviews all branches, writes findings report
  -> ORCHESTRATOR reads all reports, grades each agent against mission
  -> ORCHESTRATOR hands graded assessment + merge recommendation to LEAD
  -> LEAD executes merge order, validates after each merge
  -> LEAD writes sprint summary
  -> ORCHESTRATOR writes final sprint assessment
```

ORCHESTRATOR is the planning and grading authority. LEAD is the merge authority.

## Related Documents

- [operators-guide.md](../guides/operators-guide.md) — Full tutorial
- [methodology.md](../core/methodology.md) — How agents execute chains
- [legacy-codebases.md](../guides/legacy-codebases.md) — Adapted roles for legacy codebases
- [customization.md](../guides/customization.md) — Adapt territories for your project
- [workflow.md](../core/workflow.md) — Technical workflow details
