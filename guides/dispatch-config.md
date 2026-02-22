# Dispatch Configuration

> Optional machine-readable config format for automated dispatch pipelines

---

## When to Use This

You don't need a config file for manual sprints. The markdown docs are the system.

Use `dispatch.yaml` when:
- You're building automation around Agent Dispatch (custom scripts)
- You want a single source of truth that scripts can parse
- You're running 10+ agents and need programmatic wave management

**If you're running sprints manually with 3-8 agents, skip this file entirely.** Use [TEMPLATE-DISPATCH.md](../templates/dispatch.md) instead.

---

## Reference Format

```yaml
# dispatch.yaml — Sprint configuration
# Place in: sprint-XX/dispatch.yaml

sprint:
  id: "sprint-01"
  theme: "Payment Bug Fix"
  base_branch: "main"
  build_command: "go build ./..."
  test_command: "go test -race ./..."

agents:
  - name: "data"
    role: "DATA"
    territory:
      include:
        - "internal/store/"
        - "internal/model/"
        - "internal/repository/"
        - "migrations/"
      exclude:
        - "internal/handler/"
        - "internal/service/"
    chains:
      - id: "chain-1"
        priority: "P1"
        title: "Fix race condition in subscriptionStore"
        vector: "webhookHandler → paymentService → subscriptionStore.Activate()"
        signal: "DATA RACE at store/subscription.go:88"
        fix_site: "store/subscription.go"
        verify: "go test -race ./internal/store/..."
      - id: "chain-2"
        priority: "P2"
        title: "Add missing index on payments table"
        vector: "paymentStore.ListByUser() → full table scan"
        signal: "Query takes 2.3s on 100K rows"
        fix_site: "migrations/"
        verify: "EXPLAIN shows index scan"

  - name: "backend"
    role: "BACKEND"
    territory:
      include:
        - "internal/handler/"
        - "internal/service/"
        - "cmd/"
      exclude:
        - "internal/store/"
        - "internal/model/"
    chains:
      - id: "chain-1"
        priority: "P1"
        title: "Fix premature status update in refund handler"
        vector: "POST /refunds → refundHandler → refundService.Process()"
        signal: "Status set to 'processing' before Stripe confirms"
        fix_site: "internal/handler/refund.go"
        verify: "Refund status transitions correctly after Stripe callback"

  # ... more agents

waves:
  - number: 1
    agents: ["data", "qa", "infra"]
    description: "Foundation — no dependencies"
  - number: 2
    agents: ["backend", "services"]
    depends_on: [1]
    description: "Backend — depends on stable data layer"
  - number: 3
    agents: ["frontend"]
    depends_on: [1, 2]
    description: "Frontend — needs design specs + stable API"
  - number: 4
    agents: ["lead"]
    depends_on: [1, 2, 3]
    description: "Merge + ship"

merge_order:
  - "data"
  - "design"
  - "backend"
  - "services"
  - "frontend"
  - "infra"
  - "qa"
  - "lead"

scaling:
  max_agents_per_wave: 6
  sub_agents_enabled: false
  nested_teams: false
```

---

## Field Reference

### sprint

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Sprint identifier (e.g., "sprint-01") |
| `theme` | string | Yes | Sprint theme or goal |
| `base_branch` | string | Yes | Branch agents fork from (usually "main") |
| `build_command` | string | Yes | Build command for validation |
| `test_command` | string | Yes | Test command for validation |

### agents[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Agent identifier (used for branch naming) |
| `role` | string | Yes | Role codename (BACKEND, FRONTEND, DATA, etc.) |
| `territory.include` | string[] | Yes | Directories the agent can modify |
| `territory.exclude` | string[] | No | Directories explicitly off-limits |
| `chains[]` | object[] | Yes | Ordered list of execution traces |

### agents[].chains[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Chain identifier |
| `priority` | string | Yes | P0, P1, P2, or P3 |
| `title` | string | Yes | Short description |
| `vector` | string | Yes | Execution trace path |
| `signal` | string | Yes | What's broken and evidence |
| `fix_site` | string | Yes | Where the change happens |
| `verify` | string | Yes | How to confirm the fix |

### waves[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `number` | integer | Yes | Wave execution order |
| `agents` | string[] | Yes | Agent names in this wave |
| `depends_on` | integer[] | No | Wave numbers that must complete first |
| `description` | string | No | Human-readable purpose |

### scaling

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `max_agents_per_wave` | integer | No | Cap on parallel agents (default: 8) |
| `sub_agents_enabled` | boolean | No | Allow agents to spawn sub-agents |
| `nested_teams` | boolean | No | Enable team lead → sub-agent hierarchy |

---

## Generating from Config

Scripts can read `dispatch.yaml` to automate:

```bash
# Create worktrees from config
yq '.agents[].name' dispatch.yaml | while read agent; do
  git branch sprint-01/$agent main 2>/dev/null || true
  git worktree add "../project-${agent}" sprint-01/$agent
done

# Generate activation prompts from config
yq '.agents[] | .name + ": " + (.chains | length | tostring) + " chains"' dispatch.yaml

# Check wave dependencies
yq '.waves[] | "Wave " + (.number | tostring) + ": " + (.agents | join(", "))' dispatch.yaml
```

## Automation Interop

### Custom Scripts
Parse `dispatch.yaml` with `yq` or any YAML library. The schema is intentionally simple — no nested objects beyond two levels.

### CI Integration
Add `dispatch.yaml` validation to CI:
```yaml
# .github/workflows/dispatch-validate.yml
- run: yq eval '.' sprint-*/dispatch.yaml
```

---

**Related Documents:**
- [TEMPLATE-DISPATCH.md](../templates/dispatch.md) — Markdown sprint template (manual workflow)
- [TOOL-GUIDE.md](tool-guide.md) — Agent tool setup and configuration
- [WORKFLOW.md](../core/workflow.md) — Sprint lifecycle
