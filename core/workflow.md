# Agent Dispatch Workflow

> How multi-agent development sprints work — universal version
> Last Updated: 2026-02-22

## Overview

Agent Dispatch coordinates multiple AI coding agents working simultaneously on a codebase using git worktrees for isolation. Each agent operates in its own branch, on its own copy of the code, with a defined territory and clear merge order.

Agents work using the **Execution Methodology** — they follow execution traces through the codebase (not scan directories), complete one chain at a time, and follow P0-P3 priority levels. See [methodology.md](methodology.md) for the full theory. For legacy codebases with no tests or docs, see [legacy-codebases.md](../guides/legacy-codebases.md).

## How It Works

### 1. Sprint Planning

1. Read your progress tracker — identify goals
2. Create `sprint-XX/DISPATCH.md` — wave assignments, merge order, success criteria
3. Create `sprint-XX/agent-X-*.md` — per-agent task documents
4. Use [activation.md](../templates/activation.md) to write copy-paste activation prompts

### 2. Worktree Setup

```bash
SPRINT="sprint-01"
PROJECT_DIR="$(pwd)"
PARENT_DIR="$(dirname $PROJECT_DIR)"
PROJECT_NAME="$(basename $PROJECT_DIR)"

# Create branches and worktrees
for agent in backend frontend infra services qa data lead design red-team; do
  git branch $SPRINT/$agent main 2>/dev/null || true
  git worktree add "$PARENT_DIR/${PROJECT_NAME}-${agent}" $SPRINT/$agent
done

# Install dependencies per worktree (customize for your stack)
# Node.js: for agent in ...; do (cd ../${PROJECT_NAME}-${agent} && npm install); done
# Python:  for agent in ...; do (cd ../${PROJECT_NAME}-${agent} && pip install -r requirements.txt); done
# Go:      No action needed (modules auto-download)
```

### 3. Wave Execution

Agents dispatch in waves based on dependencies:

| Wave | Agents | Rationale |
|------|--------|-----------|
| Wave 1 | DATA, QA, INFRA, DESIGN | Foundation: data layer, tests, infra, design specs (no code deps) |
| Wave 2 | BACKEND, SERVICES | Backend: handlers + services (depend on stable data layer) |
| Wave 3 | FRONTEND | Frontend (needs DESIGN design specs + stable backend API) |
| Wave 4 | RED TEAM | Adversarial review of all agent branches (needs finished code to review) |
| Wave 5 | LEAD | Orchestrator: merge, docs, ship (informed by RED TEAM findings) |

Customize wave assignments based on your specific dependencies.

### 4. Merge Order

Merges happen sequentially in dependency order:

```
1. DATA  → main  (data layer — foundation everything depends on)
2. DESIGN    → main  (design system/tokens — FRONTEND depends on these)
3. BACKEND    → main  (backend handlers — depends on data layer)
4. SERVICES    → main  (services/integrations — depends on handlers)
5. FRONTEND    → main  (frontend — depends on DESIGN specs + backend)
6. INFRA  → main  (infrastructure — wraps everything)
7. QA     → main  (tests — validates everything)
8. LEAD     → main  (docs — last, after all code merged and RED TEAM reviewed)
```

### 5. Post-Merge Validation

After each merge, run your full validation suite:

```bash
# Customize these for your stack:
# Go:     go build ./... && go test -race ./...
# Node:   npm run build && npm test
# Python: python -m pytest && mypy .
# Rust:   cargo build && cargo test
```

### 6. Conflict Resolution

- LEAD (orchestrator) handles all merge conflicts
- Earlier agents in merge order win conflicts (DATA > DESIGN > BACKEND > SERVICES > etc.)
- Non-trivial semantic conflicts get flagged for human review

## Territory Rules

Each agent has a defined territory (directories they own):

- Agents can **read** any file in the repo
- Agents can **write** only to their territory
- Cross-territory changes require orchestrator (LEAD) approval
- Shared files (package.json, go.mod, requirements.txt) coordinated by LEAD

## Completion Protocol

Each agent produces a completion report containing:

1. Files modified (with line counts)
2. Tests added/modified
3. Issues discovered during work
4. Blockers for other agents
5. Suggested follow-up work

## Sprint Lifecycle

```
PLAN → DISPATCH → EXECUTE → MONITOR → MERGE → VALIDATE → SHIP
  │        │          │         │         │        │         │
  │        │          │         │         │        │         └─ Tag release
  │        │          │         │         │        └─ Full test suite
  │        │          │         │         └─ Sequential merge per order
  │        │          │         └─ Track status, react to events, intervene
  │        │          └─ Agents work in parallel (waves)
  │        └─ Create worktrees + paste prompts
  └─ Sprint planning from progress tracker
```

### Runtime Monitoring

The MONITOR phase runs continuously while agents execute. The operator:

1. **Tracks status** — Maintain an agent status board ([status.md](../templates/status.md)) showing each agent's state, current chain, and blockers. Update every 15-30 minutes.

2. **Reacts to events** — When something goes wrong (CI failure, stuck agent, territory violation, P0 discovery), follow the decision trees in [reactions.md](../runtime/reactions.md).

3. **Intervenes when needed** — Use copy-paste correction messages from [interventions.md](../runtime/interventions.md) to steer agents back on track.

4. **Manages escalation timers** — 5 min observe, 15 min intervene, 30 min reassign. Not every issue needs immediate action.

5. **Tracks sprint health** — GREEN (on track), YELLOW (minor issues), RED (stop and assess). See [status-tracking.md](../runtime/status-tracking.md) for the full methodology.

**Wave transitions:** Don't start Wave N+1 until all Wave N agents are COMPLETE. Check the merge readiness checklist for each agent before proceeding.

---

**Related Documents:**
- [operators-guide.md](../guides/operators-guide.md) — Full tutorial for human operators
- [methodology.md](methodology.md) — Execution traces, chain execution, priority levels
- [legacy-codebases.md](../guides/legacy-codebases.md) — Adapted workflow for legacy codebases
- [agents.md](agents.md) — Agent role definitions
- [customization.md](../guides/customization.md) — Adapt for your project
