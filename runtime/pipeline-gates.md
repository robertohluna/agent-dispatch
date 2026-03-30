# Pipeline Gates

> Hard checkpoints in the sprint pipeline. Each gate must pass before proceeding.
> Gates are enforced by the ORCHESTRATOR. The operator can override with explicit approval.

---

## Gate Overview

```
WAVE 0                    WAVE 1-3              WAVE 4-5              WAVE 6
│                         │                     │                     │
├─ GATE 0: Plan Approved  ├─ GATE 2: Wave N+1   ├─ GATE 4: Merge     ├─ GATE 5: Close
├─ GATE 1: Pre-Dispatch   │  Transition          │  Readiness          │  Sprint
│                         ├─ GATE 3: Agent       │                     │
│                         │  Completion          │                     │
```

---

## GATE 0: Plan Approved

**When:** After ORCHESTRATOR presents sprint proposal (Phase 5)
**Who checks:** Operator
**Blocks:** Document generation (Phases 6-8)

| Check | Required |
|-------|----------|
| Sprint goal is clear and scoped | YES |
| Chain count matches the scope (not too many, not too few) | YES |
| Agent count matches the work | YES |
| Wave dependencies are correct | YES |
| Success criteria are measurable (pass/fail, not vague) | YES |
| Risk assessment addresses high-risk chains | YES |
| Operator has explicitly approved | YES |

**If gate fails:** ORCHESTRATOR adjusts proposal per operator feedback. Re-present.

---

## GATE 1: Pre-Dispatch

**When:** After ORCHESTRATOR generates all documents (Phase 8.5)
**Who checks:** ORCHESTRATOR (verified by operator)
**Blocks:** Agent dispatch (pasting prompts into terminals)

### Required Files

| File | Exists? |
|------|---------|
| `sprint-XX/codebase-analysis.md` | [ ] |
| `sprint-XX/DISPATCH.md` | [ ] |
| `sprint-XX/agent-[X]-[domain].md` (one per dispatched agent) | [ ] |
| Activation prompts (one per agent) | [ ] |

### DISPATCH.md Completeness

| Section | Complete? |
|---------|----------|
| Sprint goals | [ ] |
| Codebase conventions (build/test/lint commands) | [ ] |
| All execution traces with vectors, signals, fixes, verification | [ ] |
| Territory map (every directory assigned to one agent) | [ ] |
| Wave assignments (agents to waves) | [ ] |
| Merge order | [ ] |
| Success criteria (measurable) | [ ] |
| Worktree setup script | [ ] |

### Per-Agent Doc Completeness (check every doc)

| Section | Complete? |
|---------|----------|
| Role description with cross-agent context | [ ] |
| Context reading list (numbered, with reasons) | [ ] |
| Files owned (explicit paths) | [ ] |
| Tasks with IDs, current state, required changes, key details | [ ] |
| Wave organization | [ ] |
| Territory with agent attribution on every "DO NOT touch" | [ ] |
| Verification checklist with exact commands + expected output | [ ] |
| Commit strategy | [ ] |
| Completion instructions | [ ] |

### Activation Prompt Completeness (check every prompt)

| Part | Complete? |
|------|----------|
| Part 1: Identity (codename, branch, working directory) | [ ] |
| Part 2: Context reading list | [ ] |
| Part 3: Domain + cross-agent context | [ ] |
| Part 4: Task summary + chains with traces | [ ] |
| Part 5: Territory (CAN/DO NOT) | [ ] |
| Part 6: Execution protocol + style blocks | [ ] |
| Part 7: Completion requirements | [ ] |
| Intensity protocol appended | [ ] |
| Optimal path appended | [ ] |
| Mesh mode trigger included | [ ] |

### Environment

| Check | Ready? |
|-------|--------|
| Git worktrees created (one per agent) | [ ] |
| Dependencies installed in each worktree | [ ] |
| Terminals open (one per agent) | [ ] |
| Each terminal in correct worktree directory | [ ] |
| Status board created | [ ] |

**If gate fails:** Identify which file or section is missing. Go back to the appropriate phase and complete it. Do not dispatch with incomplete documents.

---

## GATE 2: Wave Transition

**When:** Before starting Wave N+1
**Who checks:** ORCHESTRATOR + operator
**Blocks:** Next wave dispatch

| Check | Required |
|-------|----------|
| ALL Wave N agents have status COMPLETE or FAILED | YES |
| ALL Wave N completion reports exist | YES |
| Build passes on each Wave N agent's branch | YES |
| No unresolved P0 discoveries from Wave N | YES |
| Operator has acknowledged Wave N results | YES |

**Additional checks for specific transitions:**

**Wave 1 → Wave 2:**
| Check | Required |
|-------|----------|
| DATA's migrations are complete and tested | YES (if DATA dispatched) |
| INFRA's build pipeline works | YES (if INFRA dispatched) |
| DESIGN's tokens/specs are defined | YES (if DESIGN dispatched) |

**Wave 2 → Wave 3:**
| Check | Required |
|-------|----------|
| BACKEND API endpoints exist and respond | YES (if BACKEND dispatched) |
| SERVICES integrations work | YES (if SERVICES dispatched) |

**Wave 3 → Wave 4:**
| Check | Required |
|-------|----------|
| All coding agents (Waves 1-3) are COMPLETE | YES |
| All branches have been pushed and are ready for review | YES |

**Wave 4 → Wave 5:**
| Check | Required |
|-------|----------|
| RED TEAM findings report exists | YES |
| CRITICAL findings are documented | YES |
| ORCHESTRATOR has reviewed RED TEAM findings | YES |

**If gate fails:** Do NOT dispatch next wave. Diagnose the blocker:
- Agent FAILED → operator decides: retry, reassign, or skip
- P0 discovered → route to appropriate agent or park for next sprint
- Completion report missing → agent isn't done, wait

---

## GATE 3: Agent Completion

**When:** An agent claims to be done
**Who checks:** ORCHESTRATOR
**Blocks:** Agent status changing to COMPLETE

| Check | Required |
|-------|----------|
| Completion report exists (`sprint-XX/agent-[X]-completion.md`) | YES |
| All P1 chains claimed as complete | YES |
| Build passes on the agent's branch | YES |
| Tests pass on the agent's branch | YES |
| Verification commands in task doc produce expected output | YES |
| Only files in "Files Owned" were modified (`git diff --name-only`) | YES |
| No unresolved P0 discoveries (documented, not necessarily fixed) | YES |

**If gate fails:**
- Completion report missing → agent isn't done
- Build/test failure → agent must fix before being marked COMPLETE
- Territory violation → agent must revert unauthorized changes
- P1 chain incomplete → agent must finish or document why it's parked

---

## GATE 4: Merge Readiness

**When:** Before LEAD begins merging
**Who checks:** ORCHESTRATOR (provides recommendation to LEAD)
**Blocks:** Merge sequence start

| Check | Required |
|-------|----------|
| ALL coding agents (Waves 1-3) are COMPLETE | YES |
| RED TEAM findings report exists and has been reviewed | YES |
| No unresolved CRITICAL RED TEAM findings | YES |
| HIGH findings either resolved or accepted with documented risk | YES |
| ORCHESTRATOR has produced merge recommendation | YES |
| Merge order matches DISPATCH.md (or documented adjustment) | YES |
| Operator has approved merge start | YES |

**Per-branch merge check (LEAD runs this for each branch):**

| Check | Required |
|-------|----------|
| Branch is up to date with its base | YES |
| Build passes on the branch | YES |
| Tests pass on the branch | YES |
| ORCHESTRATOR grade is PASS or PARTIAL (not FAIL) | YES |
| No CRITICAL RED TEAM findings on this branch | YES |

**If gate fails:**
- CRITICAL finding unresolved → that branch does NOT merge
- Agent grade FAIL → that branch does NOT merge without operator override
- Build/test failure → agent must fix before merge

---

## GATE 5: Sprint Close

**When:** After LEAD merges and ships
**Who checks:** ORCHESTRATOR
**Blocks:** Sprint being marked as COMPLETE

| Check | Required |
|-------|----------|
| LEAD's sprint summary exists | YES |
| Final build passes on main (after all merges) | YES |
| Final tests pass on main | YES |
| Success criteria evaluated (pass/fail for each) | YES |
| ORCHESTRATOR's SPRINT-ASSESSMENT.md exists | YES |
| Per-agent grades assigned | YES |
| Carry-forward items documented | YES |
| Worktrees cleaned up | YES |
| Sprint branches cleaned up (or archived) | YES |

**If gate fails:** Complete the missing item before closing the sprint.

---

## Gate Override

Any gate can be overridden by the operator with explicit approval. When overriding:

1. State which gate is being overridden
2. State the reason
3. Document the override in the sprint assessment under "Exceptions"
4. Accept the risk — overriding a gate means accepting that the gate's checks were not met

ORCHESTRATOR cannot override gates unilaterally. Only the operator can override.

---

## Quick Reference

```
Gate 0: Plan Approved         → Operator approves proposal → generate docs
Gate 1: Pre-Dispatch          → All files exist, all complete → dispatch agents
Gate 2: Wave Transition (×N)  → All Wave N agents COMPLETE → dispatch Wave N+1
Gate 3: Agent Completion (×N) → Agent's work verified → mark COMPLETE
Gate 4: Merge Readiness       → All agents done + RED TEAM reviewed → LEAD merges
Gate 5: Sprint Close          → Assessment complete, cleanup done → sprint COMPLETE
```
