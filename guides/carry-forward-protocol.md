# Carry-Forward Protocol

> How deferred work from Sprint N becomes chains in Sprint N+1.
> Prevents lost work, forgotten P0s, and perpetual technical debt.

---

## The Problem

Sprints produce carry-forward items: unfinished chains, parked work, P0 discoveries, RED TEAM findings not resolved. Without a protocol, these items get lost between sprints. The ORCHESTRATOR for Sprint N+1 doesn't know what Sprint N left behind.

## The Solution

Every sprint produces carry-forward items. Every next sprint consumes them. The sprint registry tracks them across sprints. Nothing falls through the cracks.

---

## Step 1: Extract Carry-Forward (End of Sprint N)

ORCHESTRATOR extracts carry-forward items from three sources:

### Source 1: Agent Completion Reports

Read every `agent-X-completion.md`. Look for:
- **Parked chains** — chains the agent started but couldn't finish
- **Blocked chains** — chains waiting on another agent's output
- **P0 discoveries** — critical issues found during work
- **Suggested follow-up** — work the agent recommends for next sprint

### Source 2: RED TEAM Findings

Read `red-team-findings.md`. Look for:
- **Unresolved CRITICAL/HIGH findings** — these MUST be carry-forward P1
- **MEDIUM findings not addressed** — carry-forward as P2
- **Patterns detected** — recurring issues that need structural fixes

### Source 3: Sprint Assessment

Read `SPRINT-ASSESSMENT.md` (your own output). Look for:
- **Failed success criteria** — goals not met this sprint
- **Agent FAIL grades** — work that wasn't merged
- **Recommendations** — structural improvements for next sprint

---

## Step 2: Classify Each Item

For every carry-forward item, assign:

| Field | Value |
|-------|-------|
| **Description** | What the work is (specific, not vague) |
| **Source** | Which agent/report it came from |
| **Original Priority** | What priority it had in Sprint N |
| **Carry-Forward Priority** | What priority it should have in Sprint N+1 |
| **Reason Deferred** | Why it wasn't completed (blocker, scope, time, P0 preempted) |
| **Age** | How many sprints it has been deferred (1 if first time) |

### Priority Escalation Rules

| Situation | Action |
|-----------|--------|
| P1 item deferred once | Stays P1. MUST be in next sprint. Flag to operator. |
| P1 item deferred twice | Escalate to P0. Operator must address before any new work. |
| P2 item deferred once | Stays P2. Should be in next sprint. |
| P2 item deferred twice | Escalate to P1. Must be in next sprint. |
| P3 item deferred once | Stays P3. Include if capacity allows. |
| P3 item deferred 3+ times | Review: is this still relevant? If yes, escalate to P2. If no, remove. |
| RED TEAM CRITICAL/HIGH | Always P1 carry-forward. No exceptions. |
| P0 discovery not fixed | Always P1 carry-forward. Flag to operator. |

---

## Step 3: Update the Sprint Registry

Add carry-forward items to the registry at `docs/agent-dispatch/sprints/REGISTRY.md`:

1. Update the carry-forward tracker table
2. Mark items from previous sprints as RESOLVED if completed
3. Increment the "Age" counter for items still pending
4. Flag items with age 3+ for operator review

---

## Step 4: Feed Into Next Sprint (Start of Sprint N+1)

When ORCHESTRATOR begins planning Sprint N+1:

**Before Phase 1 (Codebase Analysis), read:**
1. `sprints/REGISTRY.md` — carry-forward tracker
2. `sprint-[N]/SPRINT-ASSESSMENT.md` — previous sprint's assessment
3. `sprint-[N]/red-team-findings.md` — unresolved findings

**During Phase 2 (Work Discovery):**
- All P1 carry-forward items become chains in Sprint N+1 automatically
- All P2 carry-forward items are candidates (include if scope allows)
- P3 carry-forward items only if capacity exists after P1/P2 assigned

**During Phase 5 (Sprint Proposal):**
- Present carry-forward items separately: "These [N] items are carried from Sprint [N]"
- Operator can defer again (with documented reason) but cannot silently drop P1 items

---

## Step 5: Document the Bridge

In Sprint N+1's `DISPATCH.md`, add a section:

```markdown
## Carry-Forward from Sprint [N]

| Item | Original Sprint | Priority | Assigned To | Chain # |
|------|----------------|----------|-------------|---------|
| [description] | Sprint [N] | P1 | BACKEND | Chain 3 |
| [description] | Sprint [N] | P2 | QA | Chain 5 |
| [description] | Sprint [N-1] → [N] | P1 (escalated) | DATA | Chain 1 |
```

This makes carry-forward visible. Anyone reading Sprint N+1's dispatch knows exactly what came from previous sprints.

---

## Anti-Patterns

**Silent drop:** Carry-forward item exists in Sprint N's assessment but doesn't appear in Sprint N+1's dispatch or registry. This means work was lost.

**Perpetual defer:** Same item deferred sprint after sprint. If an item has been deferred 3+ times, it's either not important (remove it) or being avoided (escalate it).

**Priority inflation:** Everything carried forward as P1. Defeats the purpose. Only escalate per the rules above.

**No registry:** Planning Sprint N+1 without reading Sprint N's carry-forward. ORCHESTRATOR MUST read the registry before planning.

---

## Quick Reference

```
End of Sprint N:
  1. Extract carry-forward from completion reports, RED TEAM findings, assessment
  2. Classify each item (description, source, priority, reason, age)
  3. Update sprint registry

Start of Sprint N+1:
  4. ORCHESTRATOR reads registry FIRST
  5. P1 carry-forward → automatic chains
  6. Document carry-forward in DISPATCH.md
```
