# Agent Roster

> 9 specialized agents for coordinated development sprints

## Roster

| Agent | Codename | Domain | Default Territory |
|-------|----------|--------|-------------------|
| **A** | [BACKEND](backend.md) | Backend Logic | Handlers, services, API routes, middleware |
| **B** | [FRONTEND](frontend.md) | Frontend UI | Routes, components, stores, styling |
| **C** | [INFRA](infra.md) | Infrastructure | Docker, CI/CD, builds, env config |
| **D** | [SERVICES](services.md) | Specialized Services | Integrations, workers, external APIs, ML |
| **E** | [QA](qa.md) | QA / Security | Tests, security audits, dependency scanning |
| **F** | [DATA](data.md) | Data Layer | Models, storage, migrations, data integrity |
| **G** | [LEAD](lead.md) | Orchestrator | Merge authority, docs, ship decisions |
| **H** | [DESIGN](design.md) | Design & Creative | Design system, tokens, a11y, visual specs |
| **R** | [RED TEAM](red-team.md) | Adversarial Review | Read-only on all code, write to findings + tests |

## Wave Structure

```
Wave 1: DATA, QA, INFRA, DESIGN       (foundation)
Wave 2: BACKEND, SERVICES              (backend logic)
Wave 3: FRONTEND                       (frontend)
Wave 4: RED TEAM                       (adversarial review of all branches)
Wave 5: LEAD                           (merge + ship, informed by RED TEAM findings)
```

## Merge Order

```
1. DATA       — migrations and models first
2. INFRA      — build/CI next
3. QA         — test infrastructure
4. DESIGN     — design tokens and specs
5. SERVICES   — integrations and workers
6. BACKEND    — API and business logic
7. FRONTEND   — UI (depends on backend APIs + design tokens)
8. RED TEAM   — does not merge; produces findings report
9. LEAD       — executes the merge sequence, validates after each
```

## Scaling

| Size | Agents | Use When |
|------|--------|----------|
| **Small** (3) | BACKEND, FRONTEND, LEAD | Simple sprints, few chains |
| **Medium** (5) | BACKEND, FRONTEND, QA, DATA, LEAD | Most sprints |
| **Full** (9) | All agents | Complex sprints, security-sensitive work |
| **Beyond** (10+) | Split roles into sub-agents | 15+ chains across subsystems |

For nested team architecture, sub-agent spawning rules, and merge strategy at 30+ agents, see [scaling.md](../scaling/scaling.md).

## Communication Protocol

Agents communicate through completion reports — no direct agent-to-agent messaging.

```
Agent completes work
  -> Writes completion report (sprint-XX/agent-X-completion.md)
  -> LEAD reads all reports
  -> LEAD executes merge order
  -> LEAD validates after each merge
  -> LEAD writes sprint summary
```

LEAD is the single point of coordination.

## Related Documents

- [operators-guide.md](../guides/operators-guide.md) — Full tutorial
- [methodology.md](../core/methodology.md) — How agents execute chains
- [legacy-codebases.md](../guides/legacy-codebases.md) — Adapted roles for legacy codebases
- [customization.md](../guides/customization.md) — Adapt territories for your project
- [workflow.md](../core/workflow.md) — Technical workflow details
