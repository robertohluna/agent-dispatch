# Agent Dispatch Operator's Guide

> How a human developer runs multi-agent development sprints with AI coding agents
> Universal version — works with any codebase and any AI coding agent

---

## Who This Is For

You are a **human developer** (the "operator") who will coordinate multiple AI coding agents working simultaneously on a codebase. Each agent runs in its own terminal, on its own git branch, in its own copy of the repo. You are the brain. They are the hands.

This guide works with **any AI coding agent**: Claude Code, Qwen Coder, OpenCode, Cursor, Windsurf, Aider, Continue — anything that can read files, edit code, and run commands.

---

## Table of Contents

1. [The Mental Model](#1-the-mental-model)
2. [How to Decide What to Work On](#2-how-to-decide-what-to-work-on)
3. [Sprint Planning Walkthrough](#3-sprint-planning-walkthrough)
4. [Setting Up the Environment](#4-setting-up-the-environment)
5. [Writing Activation Prompts](#5-writing-activation-prompts)
6. [Dispatching Agents (Wave by Wave)](#6-dispatching-agents-wave-by-wave)
7. [Monitoring and Steering](#7-monitoring-and-steering)
8. [Merge and Ship](#8-merge-and-ship)
9. [Agent-Specific Tips](#9-agent-specific-tips)
10. [Troubleshooting](#10-troubleshooting)
11. [Advanced: Agents Running Agent Teams](#11-advanced-agents-running-agent-teams)
12. [Execution Methodology Quick Reference](#12-signal-methodology-quick-reference)

---

## 1. The Mental Model

```
YOU (Operator)
 │
 ├─ PLAN what to work on (read your tracker, pick goals)
 ├─ DISPATCH agents (one terminal per agent, one branch per agent)
 ├─ MONITOR progress (check completion reports, answer questions)
 ├─ MERGE results (or delegate to LEAD agent)
 └─ SHIP (validate, tag, deploy)

Each Agent:
 ├─ Gets its own git worktree (isolated copy of the repo)
 ├─ Gets its own branch (sprint-XX/agent-name)
 ├─ Has a defined territory (files it can touch)
 ├─ Has a task document (what to do, acceptance criteria)
 └─ Produces a completion report when done
```

**Key principle:** Agents work in **isolation** (separate branches, separate directories). They never step on each other's toes. Conflicts are resolved at merge time, in a deterministic order.

---

## 2. How to Decide What to Work On

### Step 1: Read Your Progress Tracker

Every project should have a living document that tracks: what's done, what's broken, what's next. Look for:

- **Current Sprint Focus** — what's urgent right now
- **Bugs** — things that are broken
- **Technical Debt** — things that need cleanup
- **Feature Work** — things to build next
- **Audit Items** — security, performance, code quality

### Step 2: Write Execution Traces

Before assigning agents, **trace the signal** for each bug/task through the codebase. An execution trace is the path a signal takes from entry point to root cause:

```
BUG-001: Video preview broken
  Vector: GET /api/content/:id → contentHandler.Get()
  → contentService.GetByID() → contentStore.Read() → JSON parse
  Signal: Response returns raw JSON path instead of base64 thumbnail
  Root cause: contentStore.Read() doesn't expand relative paths
```

This trace tells you:
- **Where the bug actually lives** (store layer, not handler)
- **Which agent should fix it** (DATA, not BACKEND)
- **How to verify the fix** (reverse the trace — does GET /api/content/:id now return valid data?)

> See [METHODOLOGY.md](../core/methodology.md) for the complete methodology.

### Step 3: Choose a Sprint Theme + Assign Priorities

A sprint theme groups related work. Assign P0-P3 priorities:

| Priority | Name | Behavior |
|----------|------|----------|
| **P0** | CRITICAL ESCALATION | Stop everything. Fix first. Escalate immediately. |
| **P1** | CRITICAL | Fix before any P2/P3 work. No exceptions. |
| **P2** | IMPORTANT | Fix this sprint. Can be bumped if P0/P1 appears. |
| **P3** | HOUSEKEEPING | If time permits. Carry to next sprint if not. |

Good sprint themes:

| Theme | When to Use |
|-------|-------------|
| Bug Fix Sprint | 3+ critical bugs, users affected |
| Code Audit | After big feature push, before demo/launch |
| Security Pen Test | Before going live, before investors see it |
| Feature Sprint | Building new capability |
| Performance Sprint | Things are slow, need profiling |

### Step 4: Map Goals to Agent Domains

| Your Goal | Agent(s) |
|-----------|----------|
| Fix backend bugs | BACKEND (handlers) + DATA (data layer) |
| Fix frontend bugs | FRONTEND |
| Add tests | QA |
| Security audit | QA |
| Optimize integrations | SERVICES |
| Fix race conditions / data issues | DATA |
| Set up CI/CD / Docker | INFRA |
| Update docs, merge, ship | LEAD |
| Design system, tokens, a11y | DESIGN |
| Adversarial review before merge | RED TEAM |
| New UI component | DESIGN (spec) + FRONTEND (code) |
| New API endpoint | BACKEND + DATA (model) + FRONTEND (UI) |

### Step 5: Determine Wave Order

Ask: "Which agents can work independently? Which depend on others?"

```
Wave 1: Agents with NO dependencies on other agents' output
         (usually DATA, QA, INFRA, DESIGN)

Wave 2: Agents that need Wave 1 stable first
         (usually BACKEND, SERVICES)

Wave 3: Agents that need Wave 1/2 stable first
         (usually FRONTEND — needs DESIGN specs + stable backend)

Wave 4: RED TEAM — adversarial review of all agent branches
         (needs finished code to review)

Wave 5: LEAD always goes last (merge + docs, informed by RED TEAM findings)
```

**Rule of thumb:** Data layer first, backend second, frontend third, infra/tests anytime, docs last.

### Step 6: Write the Dispatch Doc

Copy `../templates/dispatch.md` → `sprint-XX/DISPATCH.md`. For each agent task, write **execution traces** instead of vague directory assignments:

```markdown
### Chain 1: Fix webhook timeout (P1)
Vector: POST /webhooks/stripe → webhookHandler.ProcessEvent()
→ paymentService.HandleInvoicePaid() → subscriptionStore.Activate()
Signal: 504 after 30s. subscriptionStore holds mutex during network I/O.
```

See the [examples/](../examples/) for complete sprint dispatches with execution traces.

---

## 3. Sprint Planning Walkthrough

### Example: "Fix payment bugs in an e-commerce API"

**1. Check your tracker, find the bugs:**
```
BUG-001: Stripe webhook timing out (P1 — money)
BUG-002: Double charges on retry (P1 — money)
BUG-003: Refund stuck in "pending" forever (P1 — money)
```

**2. Write execution traces for each:**
```
BUG-001: POST /webhooks/stripe → webhookHandler → paymentService.HandleInvoicePaid()
         → subscriptionStore.Activate() holds mutex during notificationService.Send()
         Signal: 504 timeout. Mutex + network I/O = deadlock risk.

BUG-002: POST /checkout → orderHandler.Create() → paymentService.Charge()
         → No idempotency key → Stripe charges twice on retry.
         Signal: Customer charged $200 instead of $100.

BUG-003: POST /refunds → refundHandler → refundService.Process()
         → Updates status to "processing" but never transitions to "completed."
         Signal: refundStore.UpdateStatus() called before Stripe confirms.
```

**3. Map to agents + set priorities (all P1 — money is involved):**
```
BUG-001 → mutex under network I/O  → DATA (data layer — fix store locking)
BUG-002 → no idempotency keys      → SERVICES (Stripe integration)
BUG-003 → premature status update   → BACKEND (handler orchestration)
```

**4. Determine waves:**
```
Wave 1: DATA (fix store mutex — foundational, other fixes depend on it)
         QA (write payment flow tests — read-only on app code)
Wave 2: BACKEND (fix handler orchestration — needs stable stores)
         SERVICES (fix Stripe integration — independent of BACKEND)
Wave 3: FRONTEND (add loading states, prevent double-click — needs stable backend)
Wave 4: RED TEAM (adversarial review of all branches — needs finished code)
Wave 5: LEAD (merge everything, informed by RED TEAM findings)
```

**5. Write DISPATCH.md with execution traces and agent task docs**

**6. Set up worktrees and dispatch**

> See [examples/ecommerce-api/](../examples/ecommerce-api/) for this complete sprint dispatch.

---

## 4. Setting Up the Environment

### 4.1 Create Worktrees

Before dispatching ANY agents, create isolated copies of the repo:

```bash
SPRINT="sprint-01"
PROJECT_DIR="$(pwd)"  # Your project root
PARENT_DIR="$(dirname $PROJECT_DIR)"
PROJECT_NAME="$(basename $PROJECT_DIR)"

# Create a branch and worktree for each agent
for agent in backend frontend infra services qa data lead design red-team; do
  git branch $SPRINT/$agent main 2>/dev/null || true
  git worktree add "$PARENT_DIR/${PROJECT_NAME}-${agent}" $SPRINT/$agent
done

echo "Worktrees ready:"
ls -d "$PARENT_DIR/${PROJECT_NAME}-"*
```

After this, your filesystem looks like:

```
parent-directory/
├── your-project/           ← Main repo (don't touch during sprint)
├── your-project-backend/     ← BACKEND's workspace
├── your-project-frontend/     ← FRONTEND's workspace
├── your-project-infra/   ← INFRA's workspace
├── your-project-services/     ← SERVICES's workspace
├── your-project-qa/      ← QA's workspace
├── your-project-data/   ← DATA's workspace
└── your-project-lead/      ← LEAD's workspace
```

### 4.2 Install Dependencies Per Worktree

If your project needs dependency installation (npm install, pip install, etc.):

```bash
# Example: Node.js project
for agent in backend frontend infra services qa data; do
  (cd "$PARENT_DIR/${PROJECT_NAME}-${agent}" && npm install)
done

# Example: Python project
for agent in backend frontend infra services qa data; do
  (cd "$PARENT_DIR/${PROJECT_NAME}-${agent}" && pip install -r requirements.txt)
done

# Example: Go project (modules auto-download, no action needed)
```

### 4.3 Open Terminals and Start Agents

One terminal per agent. Each terminal's working directory is the agent's worktree:

```bash
# Terminal 1
cd /path/to/your-project-data
claude  # or qwen-coder, opencode, cursor, etc.

# Terminal 2
cd /path/to/your-project-qa
claude

# ... etc
```

---

## 5. Writing Activation Prompts

The activation prompt is what you paste into each agent's terminal. This is the **most important part** — a good prompt means the agent works autonomously. A bad prompt means it wanders.

### The 6-Part Formula

Every activation prompt has 6 parts:

```
1. IDENTITY     — Who are you? What's your codename?
2. CONTEXT      — Read these docs for full context
3. CHAINS       — Your prioritized execution traces (trace → fix → verify)
4. TERRITORY    — What can you touch? What's off-limits?
5. PROTOCOLS    — How to work (chain execution, critical escalation)
6. COMPLETION   — What do you produce when done?
```

### Universal Template

```
You are [CODENAME] agent for [PROJECT] Sprint [XX].
Your branch is sprint-[XX]/[agent-name].

CONTEXT: Read these files first for full project understanding:
- docs/agent-dispatch/sprint-[XX]/agent-[X]-[domain].md (your chain assignments with execution traces)
- docs/agent-dispatch/agents/ (individual agent role definitions)
- [PROJECT_CONTEXT_FILE] (project overview — e.g., CLAUDE.md, README.md)

CHAINS (execute in priority order, complete each before starting the next):

Chain 1 [P1]: [title]
  Vector: [entry point] → [handler] → [service] → [store/root cause]
  Signal: [what's broken and how you know]
  Fix: [where the change needs to happen]
  Verify: [how to confirm it works]

Chain 2 [P1]: [title]
  Vector: [trace path]
  Signal: [what's broken]
  Fix: [fix site]
  Verify: [verification]

Chain 3 [P2]: [title]
  ...

TERRITORY:
- CAN modify: [list directories/file patterns]
- CANNOT modify: [list off-limits areas]
- Read anything you need for context

EXECUTION PROTOCOL:
- Complete one chain fully (trace → fix → verify → document) before starting the next
- P1 chains before P2. P2 before P3. Never skip priority order.
- All changes must compile/build: [your build command]
- Run tests after each chain: [your test command]
- Do not add new dependencies without justification
- Do not modify files outside your territory

CRITICAL ESCALATION PROTOCOL: If you discover a critical issue not in your assigned chains
(data corruption, security hole, race condition causing data loss), STOP
immediately. Document it in your completion report under "P0 DISCOVERIES".
Commit what you have. Do not attempt to fix P0 issues outside your territory.

WHEN DONE:
- Write completion report to docs/agent-dispatch/sprint-[XX]/agent-[X]-completion.md
- Commit all changes to your branch with message: "sprint-[XX]/[agent]: [summary]"
- Report: chains completed, files modified, P0 discoveries, blockers for other agents
```

### Tips for Better Prompts

| Do | Don't |
|----|-------|
| Be specific about file paths | Say "fix the backend" (too vague) |
| Include build/test commands | Assume agent knows your toolchain |
| Define territory explicitly | Let agent decide what to touch |
| State acceptance criteria | Say "make it better" (undefined) |
| Reference task doc by path | Inline all tasks (prompt too long) |
| Include quality protocol | Trust agent to self-govern |

---

## 6. Dispatching Agents (Wave by Wave)

### Wave 1: Foundation (No Dependencies)

**Dispatch simultaneously** — these agents all work in parallel:

```
Terminal 1 (DATA): Paste activation prompt → let it run
Terminal 2 (QA):    Paste activation prompt → let it run
Terminal 3 (INFRA): Paste activation prompt → let it run
Terminal 4 (DESIGN):   Paste activation prompt → let it run
```

**Wait for all Wave 1 agents to complete.**

### Wave 2: Backend (Depends on Wave 1)

**Only start after Wave 1 is complete.**

Option A — Agents work from main (simple, no Wave 1 dependency):
```
Terminal 5 (BACKEND): Paste activation prompt → let it run
Terminal 6 (SERVICES): Paste activation prompt → let it run
```

Option B — If Wave 1 changes are critical, merge DATA first:
```bash
cd /path/to/your-project
git checkout main
git merge sprint-01/data --no-ff
# Rebase Wave 2 branches onto updated main
git checkout sprint-01/backend && git rebase main
git checkout sprint-01/services && git rebase main
```

### Wave 3: Frontend (Needs DESIGN specs + stable backend)

```
Terminal 7 (FRONTEND): Paste activation prompt → let it run
```

### Wave 4: Adversarial Review

```
Terminal 8 (RED TEAM): Paste activation prompt → let it run
```

**Wait for RED TEAM to complete and deliver findings report.**

### Wave 5: Orchestrator

```
Terminal 9 (LEAD): Paste activation prompt → let it run
```

LEAD receives RED TEAM findings and decides whether each branch is safe to merge.

---

## 7. Monitoring and Steering

### What to Watch For

| Signal | Meaning | Action |
|--------|---------|--------|
| Agent reports P0 DISCOVERY | Critical issue found outside assigned chains | **Stop. Read it. Decide: fix now or park.** |
| Agent asks a question | Needs human decision | Answer it |
| Agent stuck in a loop | Confused about trace path | Intervene with specific file/line guidance |
| Agent modifies wrong files | Territory violation | Tell it to revert, stay in territory |
| Agent starts Chain 2 before finishing Chain 1 | Context-switching | Tell it to complete current chain first |
| Agent's build fails | Broke something | Let it fix, or intervene |
| Agent finishes too fast | Might have skipped verification | Check completion report — were all chains traced? |

### How to Intervene

Type directly in the agent's terminal:

```
HOLD. You're modifying files outside your territory.
Revert changes to [file path].
Your territory is [directories] only.
Continue with your remaining tasks.
```

### How to Check Progress

Ask the agent:
```
What's your progress? List completed tasks and remaining work.
```

Or read partial completion report:
```bash
cat /path/to/your-project-data/docs/agent-dispatch/sprint-01/agent-data-completion.md
```

### Advanced Monitoring

For sprints with 5+ agents, use the full monitoring toolkit:

**Status Board:** Copy [TEMPLATE-STATUS.md](../templates/status.md) into your sprint directory. Track every agent's status, current chain, blockers, and last check time. Update every 15-30 minutes. See [STATUS-TRACKING.md](../runtime/status-tracking.md) for the full methodology.

**Sprint Health:**

| Health | Criteria | Response |
|--------|----------|----------|
| GREEN | All agents active or complete, no blockers | Monitor every 30 min |
| YELLOW | 1+ agent blocked, minor scope creep | Investigate, check every 15 min |
| RED | P0 discovered, 2+ agents blocked, build broken | Stop. Assess. Intervene. |

**Escalation Timers:**

| Timer | Trigger | Action |
|-------|---------|--------|
| 5 min | Unusual behavior | Note it. Keep watching. |
| 15 min | No progress or issue persists | Intervene with correction message |
| 30 min | Still stuck after intervention | Reassign chains or restart agent |

**Quick Worktree Status:**
```bash
for agent in data design backend services frontend infra qa lead; do
  DIR="../$(basename $(pwd))-${agent}"
  [ -d "$DIR" ] && echo "[$agent] commits: $(git -C "$DIR" log main..HEAD --oneline 2>/dev/null | wc -l | tr -d ' ') uncommitted: $(git -C "$DIR" status --short 2>/dev/null | wc -l | tr -d ' ')" || echo "[$agent] NOT DISPATCHED"
done
```

**When something goes wrong:** See [REACTIONS.md](../runtime/reactions.md) for decision trees covering 12 runtime events (CI failures, stuck agents, territory violations, P0 discoveries, scope changes). See [INTERVENTIONS.md](../runtime/interventions.md) for copy-paste correction messages.

### Scaling Beyond 9 Agents

When your sprint needs more than 9 agents, roles split into specialized sub-roles or become team leads with sub-agents. See [SCALING.md](../scaling/scaling.md) for:

- Team size guide (solo through 30+ agents)
- Nested team architecture (team leads spawning sub-agents)
- Role splitting patterns (when to split BACKEND, FRONTEND, QA)
- Wave coordination at scale (max 6-8 agents per wave)
- Merge strategy (team leads pre-merge internally before merging to main)

---

## 8. Merge and Ship

### Manual Merge (Recommended)

```bash
cd /path/to/your-project
git checkout main

# Merge in dependency order
git merge sprint-01/data --no-ff -m "Sprint 01: DATA — [summary]"
# Run build + tests
[your-build-command]
[your-test-command]

git merge sprint-01/backend --no-ff -m "Sprint 01: BACKEND — [summary]"
[your-build-command]
[your-test-command]

# Continue for each agent in merge order...
# DESIGN → SERVICES → FRONTEND → INFRA → QA → LEAD
```

### Conflict Resolution

```bash
# See conflicting files
git diff --name-only --diff-filter=U

# Rule: Earlier merge order wins (DATA > DESIGN > BACKEND > SERVICES > FRONTEND > etc.)
# Resolve conflicts, then:
git add .
git merge --continue
```

### Post-Sprint Cleanup

```bash
PROJECT_DIR="$(pwd)"
PARENT_DIR="$(dirname $PROJECT_DIR)"
PROJECT_NAME="$(basename $PROJECT_DIR)"

for agent in backend frontend infra services qa data lead design red-team; do
  git worktree remove "$PARENT_DIR/${PROJECT_NAME}-${agent}" 2>/dev/null
  git branch -d sprint-01/$agent 2>/dev/null
done
echo "Sprint cleanup complete."
```

### Ship Decision Checklist

- [ ] All success criteria from DISPATCH.md met?
- [ ] Build passes?
- [ ] Tests pass?
- [ ] No CRITICAL issues in completion reports?
- [ ] Progress tracker updated?

If yes: `git tag sprint-01-complete && git push origin main --tags`

---

## 9. Agent-Specific Tips

### Claude Code

```bash
# Best experience — native agent teams support
claude --dangerously-skip-permissions  # For autonomous operation

# Each agent can spawn sub-agents via Task tool:
# "Use the Task tool to spawn a helper for this subtask"
```

### Qwen Coder

Best with explicit full file paths in prompts:
```
Read these files first:
1. /full/absolute/path/to/CLAUDE.md
2. /full/absolute/path/to/agent-task.md
```

### OpenCode

Good multi-file editing. Emphasize:
```
You may edit multiple files simultaneously.
Prefer editing all related files in a single operation.
```

### Cursor / Windsurf

Open the worktree folder as a separate project/workspace. Use Composer mode for multi-file edits.

### Aider

```bash
# Autonomous mode
aider --yes --auto-commits

# Pre-load context files
aider --read docs/agent-dispatch/sprint-01/agent-backend.md
```

### General Tips (All Agents)

1. **Front-load context.** The first thing the agent reads shapes everything.
2. **Be explicit about what NOT to do.** Territory boundaries prevent chaos.
3. **Include build commands.** Agents should verify their work.
4. **Request completion reports.** Without them, you can't merge confidently.
5. **Quality protocol in every prompt.** Read → understand → fix → verify → document.

---

## 10. Troubleshooting

### Agent Modifies Wrong Files

```
STOP. You modified [file] which is outside your territory.
Revert: git checkout -- [file]
Your territory is [directories] only. Continue within territory.
```

### Agent Adds Unnecessary Code

```
HOLD. You added [dependency/abstraction]. This sprint doesn't require it.
Remove it and use [existing tool] instead. Keep changes minimal.
```

### Merge Conflict

```bash
# Check which agents touched the file
git log --oneline sprint-01/backend -- path/to/file
git log --oneline sprint-01/services -- path/to/file
# Earlier merge order wins unless later version is clearly better
```

### Agent Gets Stuck

```
STOP iterating. Here's what I need:
1. Revert your last 3 changes
2. Read [specific file] lines [X-Y]
3. The fix is: [specific instruction]
4. Make ONLY that change, verify it compiles, move on
```

### Worktree Issues

```bash
git worktree list                              # See all worktrees
git worktree remove ../project-backend --force   # Force remove
git branch -D sprint-01/backend                  # Delete branch
git branch sprint-01/backend main                # Recreate
git worktree add ../project-backend sprint-01/backend  # Recreate worktree
```

---

## 11. Advanced: Agents Running Agent Teams

The power move. Each dispatched agent can spawn sub-agents:

```
YOU (Operator)
 ├─ BACKEND agent
 │   ├─ handler-fixer sub-agent
 │   ├─ error-auditor sub-agent
 │   └─ synthesizes results → completion report
 ├─ FRONTEND agent
 │   ├─ network-auditor sub-agent
 │   ├─ code-cleaner sub-agent
 │   └─ synthesizes results → completion report
 └─ ...
```

### Enabling This in Activation Prompts

Add to any agent's prompt:

```
You may spawn sub-agents for independent subtasks. For example:
- Spawn a "[subtask-name]" agent for [specific work]
- Spawn a "[subtask-name]" agent for [specific work]
- Run them in parallel, collect results
- Synthesize into your completion report

Sub-agent rules:
- Each gets a specific, well-defined subtask
- Each produces clear output (files modified, findings)
- None modify files outside YOUR territory
- You validate combined result compiles and passes tests
```

### For Non-Team-Capable Agents

If your agent can't spawn sub-agents natively, **you** act as dispatcher:
1. Agent says "I need to do X and Y, they're independent"
2. You open 2 more terminals in that agent's worktree
3. Give each terminal a focused subtask
4. Results merge into the same branch

### The Chain Execution Protocol

Add to every activation prompt for maximum quality:

```
FOR EACH CHAIN:
  1. TRACE — Follow the execution traces through the codebase (entry → root cause)
  2. DIAGNOSE — Is this the actual root cause, or a symptom of something deeper?
  3. FIX — Smallest correct change at the root cause site
  4. VERIFY — Build passes, tests pass, reverse-trace confirms the signal succeeds
  5. DOCUMENT — Chain completed: what changed, why, files touched, line counts
  6. NEXT — Move to next chain. Never start a new chain until current one is COMPLETE.

IF P0 DISCOVERED DURING CHAIN:
  → STOP current chain
  → Document P0 in completion report
  → Commit what you have
  → Flag for operator
```

---

## Quick Reference Card

```
SPRINT LIFECYCLE:
  1. Read progress tracker     → Find what needs work
  2. Write DISPATCH.md          → Plan the sprint
  3. Write agent task docs      → Tasks per agent
  4. Run worktree setup         → Isolated branches
  5. Open terminals             → One per agent
  6. Paste activation prompts   → Boot each agent
  7. Monitor                    → Steer when needed
  8. Collect completion reports → Know what each did
  9. Merge in order             → Dependency sequence
  10. Validate                  → Build + test after each merge
  11. Ship or rollback          → Tag if criteria met
  12. Cleanup worktrees         → Remove branches + dirs

MERGE ORDER (default):
  DATA → DESIGN → BACKEND → SERVICES → FRONTEND → INFRA → QA → LEAD
  (data → design → backend → services → frontend → infra → tests → docs)
  RED TEAM does not merge — produces findings report that informs LEAD's merge decisions.

WAVE ORDER (default):
  Wave 1: DATA + QA + INFRA + DESIGN  (no deps)
  Wave 2: BACKEND + SERVICES                     (need Wave 1)
  Wave 3: FRONTEND                             (needs DESIGN specs + stable backend)
  Wave 4: RED TEAM                             (adversarial review of all branches)
  Wave 5: LEAD                              (merge + ship, informed by RED TEAM findings)
```

---

## 12. Execution Methodology Quick Reference

The full theory is in [METHODOLOGY.md](../core/methodology.md). Here's the operator cheat sheet:

### Execution Traces (How to Write Agent Tasks)

Don't say: "BACKEND: fix bugs in handlers/"

Do say:
```
Chain 1 [P1]: Fix webhook timeout
  Vector: POST /webhooks/stripe → webhookHandler.ProcessEvent()
  → paymentService.HandleInvoicePaid() → subscriptionStore.Activate()
  Signal: 504 after 30s. Mutex held during network I/O.
  Fix: Release lock before notification call.
  Verify: Webhook responds <2s. No race on concurrent activations.
```

### Priority Levels (What Order Agents Work In)

```
P0 CRITICAL ESCALATION  → Stop everything. Fix now. Alert operator.
P1 CRITICAL   → Fix before any P2/P3 work.
P2 IMPORTANT  → Fix this sprint. Can be bumped.
P3 HOUSEKEEPING → If time permits.
```

### Chain Execution (How Agents Work)

```
Complete Chain 1 fully → then Chain 2 → then Chain 3
Never context-switch mid-chain.
Exception: Chain blocked → park it → start next → return when unblocked.
```

### Context Density (Where to Focus)

80% of bugs live in 20% of files. Find the high-density files and focus agents there. Better to deeply fix 3 files than shallowly touch 20.

### Merge Validation (When Agents' Work Meets)

Every merge is a merge validation point. Build + test after EVERY merge:
```
Merge DATA → build + test ✓
Merge BACKEND   → build + test ✓
Merge SERVICES   → build + test ✓  ← This is where DATA + BACKEND + SERVICES meet
```

### Execution Pace

| Agent | Speed | Reason |
|-------|-------|--------|
| DATA | Slow, careful | Data corruption risk |
| DESIGN | Deliberate | Design decisions cascade through every component |
| BACKEND | Moderate | Needs thorough tracing |
| SERVICES | Moderate | External side effects |
| FRONTEND | Fast, iterative | Visual feedback loop |
| INFRA | Fast | Mechanical changes |
| QA | Broad, scanning | Coverage over depth |
| RED TEAM | Thorough | Value is in what it catches, not speed |
| LEAD | Deliberate | Merge decisions are irreversible |

---

**Related Documents:**
- [METHODOLOGY.md](../core/methodology.md) — Full methodology (execution traces, chain execution, priority levels)
- [WORKFLOW.md](../core/workflow.md) — Technical workflow details
- [agents/](../agents/) — Individual agent role definitions
- [CUSTOMIZATION.md](customization.md) — How to adapt for your project
- [TEMPLATE-DISPATCH.md](../templates/dispatch.md) — Sprint template
- [TEMPLATE-AGENT.md](../templates/agent.md) — Agent task template
- [TEMPLATE-COMPLETION.md](../templates/completion.md) — Completion report template
- [TEMPLATE-ACTIVATION.md](../templates/activation.md) — Activation prompt templates
- [examples/](../examples/) — Complete sprint dispatch examples across 6 stacks
