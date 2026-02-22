# Execution Methodology

> How agents focus, chain, escalate, and merge — the theory behind Agent Dispatch

---

## Core Concept: Execution Traces

Agents don't work on "directories." They work on **execution traces** — a traced chain through the codebase that follows a specific signal (bug, feature, debt) from entry point to root cause and back.

```
Traditional (weak):
  "BACKEND: fix bugs in handlers/"
  → Agent scans randomly, makes scattered changes, misses root causes

Execution Trace (strong):
  "BACKEND: Trace BUG-001 from HTTP request entry (router) → handler parsing
  → service call → store mutation → response serialization. Fix at root cause.
  Then trace BUG-002 along the same path."
  → Agent follows the signal, finds the exact failure point, fixes surgically
```

An execution trace has:
- **Entry point** — where the signal enters the system (HTTP request, event, cron)
- **Trace path** — the chain of files/functions the signal flows through
- **Root cause location** — where the actual bug/issue lives
- **Fix site** — where the change needs to happen (may differ from root cause)
- **Verification path** — how to confirm the fix works (reverse trace)

### Writing Execution Traces in Task Docs

```markdown
### Task: Fix payment webhook timeout

**Vector:** POST /webhooks/stripe → webhookHandler.ProcessEvent()
→ paymentService.HandleInvoicePaid() → subscriptionStore.Activate()
→ notificationService.Send()

**Signal:** Webhook returns 504 after 30s. Stripe retries 3x, all fail.

**Hypothesis:** subscriptionStore.Activate() holds a write lock while
calling notificationService.Send() (network I/O under mutex).

**Fix site:** subscriptionStore.Activate() — release lock before notification.

**Verify:** Send test webhook, confirm <2s response. Check no race condition
with concurrent activations.
```

---

## Chain Execution

Agents don't context-switch randomly. They complete one **chain** before moving to the next. A chain is a full execution trace from trace to fix to verify.

```
Chain 1: Trace BUG-001 → Find root cause → Fix → Verify → Document
Chain 2: Trace BUG-002 → Find root cause → Fix → Verify → Document
Chain 3: Trace BUG-003 → Find root cause → Fix → Verify → Document
```

**Never:** Start Chain 2 before Chain 1 is complete.
**Exception:** If Chain 1 is blocked (needs info from another agent), park it and start Chain 2. Return to Chain 1 when unblocked.

### Chain States

```
TRACING    → Agent is reading code, following the signal path
DIAGNOSING → Agent has found the failure point, analyzing root cause
FIXING     → Agent is implementing the change
VERIFYING  → Agent is testing the fix (build, test, manual check)
DOCUMENTING → Agent is writing the change into completion report
COMPLETE   → Chain is done, move to next chain
PARKED     → Chain is blocked, waiting on external input
```

---

## Priority Levels (P0-P3)

Not all tasks are equal. Assign priority levels that determine execution order and escalation behavior.

### Priority Levels

| Priority | Name | Behavior | Example |
|----------|------|----------|---------|
| **P0** | CRITICAL ESCALATION | **Stop everything.** Fix this first. Alert operator immediately if discovered during work. | Data corruption, security vulnerability, production crash |
| **P1** | CRITICAL | Fix before any P2/P3 work. No exceptions. | User-facing bug, broken API endpoint, test failure on critical path |
| **P2** | IMPORTANT | Fix during this sprint. Can be deprioritized if P0/P1 appears. | Code duplication, missing error handling, performance issue |
| **P3** | HOUSEKEEPING | Fix if time permits. Carry over to next sprint if not. | Dead code removal, naming improvements, comment cleanup |

### Critical Escalation

When an agent discovers a **P0 issue during work** (even if it wasn't in their task list):

1. **STOP** current chain
2. **DOCUMENT** the P0 finding immediately in completion report
3. **ALERT** the operator (flag in commit message, write to a known alert file)
4. **WAIT** for operator decision (fix now, or park and continue)

Add this to activation prompts:

```
CRITICAL ESCALATION PROTOCOL: If you discover a critical issue not in your assigned chains
(data corruption, security hole, race condition causing data loss), STOP
immediately. Document it in your completion report under "P0 DISCOVERIES".
Commit what you have. Do not attempt to fix P0 issues outside your territory
— flag them for the appropriate agent.
```

### Priority in Task Docs

```markdown
## Tasks (ordered by priority)

### P0: None assigned (critical escalation only — discovered during work)

### P1: Fix webhook timeout [Chain 1]
**Vector:** POST /webhooks → handler → service → store
...

### P1: Fix auth token validation [Chain 2]
**Vector:** middleware.Authenticate() → tokenService.Validate()
...

### P2: Improve error messages [Chain 3]
**Vector:** All handlers → errorResponse() utility
...

### P3: Remove deprecated imports [Chain 4]
**Vector:** grep across all files
...
```

---

## Merge Validation Pattern

After agents complete their chains independently, their work meets during merge. The merge order isn't arbitrary — it follows the dependency graph of the codebase:

```
DATA (data layer)
    ↓ depends on
DESIGN (design system/tokens)
    ↓ depends on
BACKEND (backend logic)
    ↓ depends on
SERVICES (services/integrations)
    ↓ depends on
FRONTEND (frontend — uses DESIGN specs + backend APIs)
    ↓ wraps
INFRA (infrastructure)
    ↓ validates
QA (tests)
    ↓ documents
LEAD (ship)
```

Each merge is a **merge validation point** — the moment two independent chains of work meet and must be validated together. This is where bugs hide. That's why you build + test after EVERY merge, not just at the end.

---

## Context Density

Not all files deserve equal agent attention. **Context density** measures how much of the problem lives in a given file.

```
High context density:  payment_service.go (core business logic, 8 bugs traced here)
Medium context density: user_handler.go (3 bugs, standard CRUD)
Low context density:   health_check.go (0 bugs, trivial endpoint)
```

### Applying Context Density

1. **Focus agents on high-density files first.** If 80% of bugs trace to 3 files, those 3 files ARE the sprint.
2. **Don't spread agents thin.** Better to have BACKEND deeply fix 3 high-density files than shallowly touch 20 files.
3. **Measure density per chain.** After tracing a bug, note which files it touches. Files that appear in multiple chains are high-density.

---

## Execution Pace

Different agents work at different speeds. Match pace to the work:

| Agent | Pace | Why |
|-------|------|-----|
| DATA | Slow, careful | Data layer changes can corrupt everything. Measure twice. |
| DESIGN | Deliberate | Design decisions cascade through every component. Wrong tokens propagate everywhere. |
| BACKEND | Moderate | Backend logic needs tracing. Don't rush root cause analysis. |
| SERVICES | Moderate | External integrations have side effects. Test thoroughly. |
| FRONTEND | Fast, iterative | Frontend changes are visible immediately. Iterate on UI. |
| INFRA | Fast | Infrastructure changes are mechanical. Either works or doesn't. |
| QA | Broad, scanning | Tests cover surface area. Write many, verify all pass. |
| LEAD | Deliberate | Merge decisions are irreversible. Review every conflict. |

---

## The Complete Agent Execution Loop

```
RECEIVE task doc with prioritized chains
  │
  FOR EACH chain (P1 first, then P2, then P3):
  │
  ├─ TRACE: Follow the execution trace through the codebase
  │   └─ Read entry point → follow function calls → find failure point
  │
  ├─ DIAGNOSE: Identify root cause
  │   └─ Is this the actual cause, or a symptom of something deeper?
  │   └─ If deeper: extend the trace. Don't fix symptoms.
  │
  ├─ FIX: Implement the smallest correct change
  │   └─ Change only what's necessary
  │   └─ If fix requires touching another agent's territory → PARK, document
  │
  ├─ VERIFY: Confirm the fix
  │   └─ Build passes
  │   └─ Existing tests pass
  │   └─ New test covers the fix (if QA territory, note for QA)
  │   └─ Reverse-trace: does the original signal now succeed?
  │
  ├─ DOCUMENT: Record in completion report
  │   └─ What changed, why, which files, line counts
  │   └─ Any P0 discoveries (critical escalation)
  │   └─ Any blockers for other agents
  │
  └─ NEXT chain
  │
  WHEN ALL chains complete:
  ├─ Final build + test
  ├─ Write completion report
  └─ Commit and signal done
```

---

## Runtime Operations

The methodology above covers planning and execution. Once agents are running, you need a system for monitoring and reacting to events in real time.

**Status Tracking:** Track agent states (IDLE → ACTIVE → BLOCKED → COMPLETE), chain progress (QUEUED → TRACING → FIXING → VERIFYING → COMPLETE), and sprint health (GREEN/YELLOW/RED). See [status-tracking.md](../runtime/status-tracking.md).

**Reactions:** When something goes wrong mid-sprint — CI fails, agent gets stuck, territory violation, P0 discovery — use the decision trees in [reactions.md](../runtime/reactions.md). Each follows SYMPTOM → DIAGNOSIS → ACTION → VERIFY.

**Interventions:** When you need to correct an agent, [interventions.md](../runtime/interventions.md) has 24 copy-paste message templates organized by category (territory violations, chain execution, stuck protocols, P0 routing, inter-agent conflict, crash recovery, output quality, scope management).

**Escalation Timers:** Not every issue needs immediate intervention. Default: 5 min observe → 15 min intervene → 30 min reassign.

---

## Summary

| Concept | What It Means |
|---------|---------------|
| **Execution Trace** | Trace a signal through the codebase, don't scatter across directories |
| **Chain Execution** | Complete one full trace-fix-verify cycle before starting the next |
| **Priority Levels** | P0 (stop everything) → P1 (fix first) → P2 (this sprint) → P3 (if time) |
| **Critical Escalation** | Critical discoveries during work get immediate escalation |
| **Merge Validation** | Merge order follows dependency graph; validate at every merge |
| **Context Density** | Focus on high-density files where most bugs concentrate |
| **Execution Pace** | Match execution speed to the risk profile of the work |

---

**Related Documents:**
- [operators-guide.md](../guides/operators-guide.md) — How to run the system
- [workflow.md](workflow.md) — Technical workflow
- [agents.md](agents.md) — Agent roles
- [legacy-codebases.md](../guides/legacy-codebases.md) — Adapted methodology for legacy codebases
- [reactions.md](../runtime/reactions.md) — Runtime reaction decision trees
- [interventions.md](../runtime/interventions.md) — Intervention catalog
- [status-tracking.md](../runtime/status-tracking.md) — Sprint monitoring

