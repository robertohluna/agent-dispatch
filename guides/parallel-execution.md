# Parallel Execution Guide

> How Agent Dispatch scales across multiple terminals, sub-agents, and concurrent sessions.
> This is the operational model for running 30-100+ agents simultaneously.

---

## The Model

Each agent in a sprint gets its **own terminal session** — a separate Claude Code instance, Cursor window, Aider process, or whatever AI coding tool you're using. Each terminal is an isolated execution environment with its own git worktree (branch + working directory copy).

Inside each terminal, the agent can spawn **sub-agents** for parallel chain execution. Each sub-agent handles one chain or one well-defined subtask.

```
┌─────────────────────────────────────────────────────────────┐
│                    OPERATOR (you)                            │
│                                                             │
│  Pastes activation prompts into separate terminals          │
│  Monitors progress across all agents                        │
│  Manages wave transitions                                   │
└──────────┬──────────┬──────────┬──────────┬────────────────┘
           │          │          │          │
     ┌─────▼────┐ ┌──▼─────┐ ┌─▼──────┐ ┌▼────────┐
     │Terminal 1 │ │Terminal│ │Terminal│ │Terminal │    ...
     │BACKEND   │ │FRONTEND│ │DATA    │ │SERVICES │
     │          │ │        │ │        │ │         │
     │┌────────┐│ │┌──────┐│ │┌──────┐│ │┌───────┐│
     ││Agent A ││ ││Agent ││ ││Agent ││ ││Agent  ││
     ││        ││ ││B     ││ ││F     ││ ││D      ││
     ││ sub-1  ││ ││ sub-1││ ││ sub-1││ ││ sub-1 ││
     ││ sub-2  ││ ││ sub-2││ ││ sub-2││ ││ sub-2 ││
     ││ sub-3  ││ ││ sub-3││ ││      ││ ││ sub-3 ││
     │└────────┘│ │└──────┘│ │└──────┘│ │└───────┘│
     └──────────┘ └────────┘ └────────┘ └─────────┘
          │            │          │           │
          ▼            ▼          ▼           ▼
     sprint-XX/   sprint-XX/  sprint-XX/  sprint-XX/
     backend      frontend    data        services
     (branch)     (branch)    (branch)    (branch)
```

**Every terminal is isolated.** Different branch, different working directory, different agent context. They cannot interfere with each other during execution. Conflicts are resolved at merge time by LEAD, not during execution.

---

## Scaling Math

### Sub-Agents Per Terminal

Each agent terminal spawns sub-agents based on its chain count and chain independence:

| Chain Count | Independent Chains | Sub-Agents | Why |
|-------------|-------------------|------------|-----|
| 1-2 | Any | 0 (agent handles directly) | Overhead of sub-agents > benefit |
| 3-4 | 2+ independent | 2-3 | One sub-agent per independent chain |
| 5-8 | 3+ independent | 3-6 | Parallel chain execution |
| 8-15 | 5+ independent | 5-10 | Maximum practical parallelism |
| 15+ | Split the role | N/A | This agent should become a team lead |

**Rule:** Sub-agents only help when chains are **independent** (no data dependency between them). If Chain 2 depends on Chain 1's output, they must run sequentially regardless of sub-agent count.

### Total Agent Count

| Terminals | Agents/Terminal | Total Agents | Use Case |
|-----------|----------------|--------------|----------|
| 2 | 1 (no sub-agents) | 2 | Quick bug fix, 2 domains |
| 3 | 3-5 | 9-15 | Standard sprint, 3 waves |
| 5 | 5-8 | 25-40 | Full sprint, all 9 roles |
| 6 | 5-8 | 30-48 | Full sprint with deep chains |
| 9 | 5-15 | 45-135 | Large sprint, nested teams |
| 12+ | 10-15 | 120-180+ | Enterprise, multi-repo |

**Example:** You have 6 Claude Code terminals open. Each runs one agent role. Each agent spawns 5-6 sub-agents for parallel chain execution. That's **30-36 agents running simultaneously** on isolated branches, all coordinated by the dispatch plan.

---

## How Each Prompt Goes to a Separate Terminal

This is the critical mental model. **Each activation prompt is a separate session.**

### Setup

```bash
# 1. Create branches and worktrees (run once from main repo)
SPRINT="sprint-05"
PROJECT_DIR="$(pwd)"
PARENT_DIR="$(dirname $PROJECT_DIR)"
PROJECT_NAME="$(basename $PROJECT_DIR)"

for agent in data qa infra backend services frontend red-team lead; do
  git branch $SPRINT/$agent main 2>/dev/null || true
  git worktree add "$PARENT_DIR/${PROJECT_NAME}-${agent}" $SPRINT/$agent
done
```

### Dispatch

```bash
# 2. Open separate terminals — one per agent

# Terminal 1: DATA agent
cd ../project-data
# Paste Agent F (DATA) activation prompt here

# Terminal 2: QA agent
cd ../project-qa
# Paste Agent E (QA) activation prompt here

# Terminal 3: INFRA agent
cd ../project-infra
# Paste Agent C (INFRA) activation prompt here

# Terminal 4: BACKEND agent (wait for Wave 1 to complete)
cd ../project-backend
# Paste Agent A (BACKEND) activation prompt here

# Terminal 5: SERVICES agent (wait for Wave 1 to complete)
cd ../project-services
# Paste Agent D (SERVICES) activation prompt here

# Terminal 6: FRONTEND agent (wait for Waves 1+2 to complete)
cd ../project-frontend
# Paste Agent B (FRONTEND) activation prompt here

# Terminal 7: RED TEAM (wait for all code agents to finish)
cd ../project-red-team
# Paste Agent R (RED TEAM) activation prompt here

# Terminal 8: LEAD (after RED TEAM review)
cd ../project-lead
# Paste Agent G (LEAD) activation prompt here
```

### Key Points

- **Each terminal is a fresh AI session.** No shared context between agents.
- **Each prompt contains everything the agent needs** — identity, context files to read, tasks, chains, territory, methodology, completion requirements.
- **Wave discipline matters.** Don't paste Wave 2 prompts until Wave 1 agents are done. Wave 2 agents depend on Wave 1's stable data layer.
- **Sub-agents spawn inside each terminal.** When the activation prompt includes TEAM MODE, the agent spawns sub-agents (Claude Code Task tool, etc.) for parallel chain execution within its own branch.

---

## Sub-Agent Protocol

When an agent's activation prompt includes the TEAM MODE block, it spawns sub-agents:

```
TEAM MODE: You have sub-agents. Use them. Maximum parallel execution.

For each independent chain in your assignment, spawn a dedicated sub-agent:
- "chain-1-webhook-fix" agent for Chain 1: Fix webhook timeout
- "chain-2-refund-handler" agent for Chain 2: Fix premature refund status
- "chain-3-rate-limit" agent for Chain 3: Add rate limiting to auth endpoints

Run ALL independent chains simultaneously. Do not serialize work that can
be parallelized.

Sub-agent rules:
- Each sub-agent gets ONE chain or ONE well-defined subtask
- Sub-agents inherit YOUR territory — they cannot expand it
- Sub-agents must document what files they changed and what they verified
- YOU validate the combined output
- After all sub-agents complete: run [build] && [test] on the combined result
```

### Sub-Agent Coordination

The parent agent (the one in the terminal) is responsible for:

1. **Spawning** — Creating sub-agents with clear, scoped assignments
2. **Territory enforcement** — Sub-agents stay within the parent's territory
3. **Synthesis** — Collecting results, checking for file conflicts between sub-agents
4. **Validation** — Running build + test on the combined output
5. **Reporting** — Writing one completion report covering all chains

Sub-agents DO NOT:
- Modify files outside the parent agent's territory
- Spawn their own sub-agents (one level deep only)
- Write completion reports (the parent writes the report)
- Make merge decisions

---

## Practical Scaling Examples

### Small Sprint: 3 Terminals, 9-12 Total Agents

Bug fix sprint — 3 execution traces across backend, data, and QA.

```
Terminal 1: DATA agent
  └─ sub-1: Chain 1 (fix race condition in store)
  └─ sub-2: Chain 2 (add missing index)

Terminal 2: BACKEND agent
  └─ sub-1: Chain 1 (fix webhook handler timeout)
  └─ sub-2: Chain 2 (fix refund status update)
  └─ sub-3: Chain 3 (add rate limiting)

Terminal 3: QA agent
  └─ sub-1: Write regression tests for DATA fixes
  └─ sub-2: Write tests for BACKEND fixes
  └─ sub-3: Security audit on new rate limiter
```

**Total: 3 terminals, 8 sub-agents + 3 parent agents = 11 agents**

### Standard Sprint: 6 Terminals, 30-40 Total Agents

Feature sprint — new payment system across all layers.

```
Terminal 1: DATA (Wave 1)
  └─ 3 sub-agents: schema migration, store methods, seed data

Terminal 2: INFRA (Wave 1)
  └─ 2 sub-agents: Docker config, CI pipeline

Terminal 3: QA (Wave 1)
  └─ 4 sub-agents: test infra, unit tests, integration tests, security audit

Terminal 4: BACKEND (Wave 2)
  └─ 5 sub-agents: handlers, services, middleware, validation, error handling

Terminal 5: SERVICES (Wave 2)
  └─ 4 sub-agents: Stripe integration, webhook processor, notification worker, retry logic

Terminal 6: FRONTEND (Wave 3)
  └─ 5 sub-agents: payment form, checkout flow, receipt page, error states, loading states
```

**Total: 6 terminals, 23 sub-agents + 6 parent agents = 29 agents**
Plus RED TEAM and LEAD in Waves 4-5 = **~35 agents total**

### Large Sprint: 9 Terminals, 80-100+ Total Agents

Full-stack migration — moving from monolith to microservices.

```
Terminal 1: DATA (Wave 1) — 8 sub-agents
Terminal 2: DESIGN (Wave 1) — 5 sub-agents
Terminal 3: INFRA (Wave 1) — 6 sub-agents
Terminal 4: QA (Wave 1) — 10 sub-agents
Terminal 5: BACKEND (Wave 2) — 12 sub-agents
Terminal 6: SERVICES-AI (Wave 2) — 8 sub-agents
Terminal 7: SERVICES-RSS (Wave 2) — 6 sub-agents
Terminal 8: FRONTEND (Wave 3) — 15 sub-agents
Terminal 9: RED TEAM (Wave 4) — 8 sub-agents
+ LEAD (Wave 5) — 4 sub-agents
```

**Total: 10 terminals, 82 sub-agents + 10 parent agents = 92 agents**

At this scale, each role is effectively a **team lead** managing a squad of sub-agents. See [scaling/scaling.md](../scaling/scaling.md) for the nested team architecture.

---

## Terminal Management

### Using tmux (Recommended)

```bash
# Create a tmux session per agent
for agent in data qa infra backend services frontend red-team lead; do
  tmux new-session -d -s "sprint-$agent" -c "../project-${agent}"
done

# List all agent sessions
tmux ls

# Attach to specific agent
tmux attach -t sprint-backend

# Switch between agents
# Ctrl-b ) → next session
# Ctrl-b ( → previous session
# Ctrl-b s → session picker
```

### Using Multiple Terminal Windows

Open separate terminal windows or tabs. Each window runs one agent. Label them clearly.

### Using IDE Workspaces

For Cursor/Windsurf: open each worktree as a separate project/workspace.

### Monitoring All Agents

While agents work, the operator:

1. **Cycles through terminals** checking progress (every 15-30 min)
2. **Updates the status board** (`templates/status.md`)
3. **Reacts to events** using decision trees from `runtime/reactions.md`
4. **Manages wave transitions** — Wave 2 agents don't start until Wave 1 is done

---

## Agent Isolation Guarantees

Why parallel execution works without agents breaking each other:

| Isolation Layer | How It Works |
|----------------|-------------|
| **Git worktrees** | Each agent has its own directory with its own branch. File system isolation. |
| **Territory rules** | Each agent can only modify declared files/directories. Enforced by convention + completion report audit. |
| **Wave ordering** | Dependency-respecting execution order. DATA finishes before BACKEND starts. |
| **No shared state** | Agents don't communicate during execution. They work from their task doc. |
| **Merge validation** | Build + test after every merge catches integration issues. |
| **RED TEAM review** | Adversarial agent reviews all branches before merge for security, edge cases, territory violations. |

**The only time agents "interact" is at merge time.** And that's managed by LEAD using the dependency-ordered merge sequence with validation after every step.

---

## Troubleshooting

### Agent Spawns Too Many Sub-Agents
- **Symptom:** Context window overflow, hallucination, incomplete work
- **Fix:** Reduce to 3-5 sub-agents max. Quality > quantity. Better to serialize some chains than overwhelm the agent.

### Wave 2 Agent Starts Before Wave 1 Finishes
- **Symptom:** Build failures because data layer isn't stable
- **Fix:** Wait. Check Wave 1 completion reports before pasting Wave 2 prompts.

### Sub-Agents Modify Same File
- **Symptom:** One sub-agent's changes overwrite another's
- **Fix:** Parent agent must assign non-overlapping file scopes to sub-agents. If two chains touch the same file, serialize them.

### Terminal Runs Out of Context
- **Symptom:** Agent forgets earlier chains, produces incomplete work
- **Fix:** Split into multiple sessions. Agent completes chains 1-3 in session 1, chains 4-6 in session 2. Each session gets its own activation prompt covering only its chains.

### Agent Produces No Completion Report
- **Symptom:** Terminal closed or agent claimed "done" without report
- **Fix:** Re-activate the agent with: "Read your task doc. Write your completion report. Include all chains, files modified, and verification results."

---

**Related Documents:**
- [templates/activation.md](../templates/activation.md) — Activation prompt structure + TEAM MODE block
- [scaling/scaling.md](../scaling/scaling.md) — Nested teams for 20-30+ agents
- [runtime/status-tracking.md](../runtime/status-tracking.md) — Monitoring agent status
- [runtime/reactions.md](../runtime/reactions.md) — Decision trees for runtime events
- [config/dispatch-styles.md](../config/dispatch-styles.md) — Style blocks appended to every prompt
