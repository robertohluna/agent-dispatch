# Sprint Registry

> Central index of all sprints. Updated after every sprint by ORCHESTRATOR.
> Copy this template to your project at `docs/agent-dispatch/sprints/REGISTRY.md`

---

## Active Sprint

| Field | Value |
|-------|-------|
| Sprint | [XX] |
| Theme | [theme] |
| Status | [PLANNING / DISPATCHED / IN PROGRESS / MERGING / ASSESSING / COMPLETE] |
| Started | YYYY-MM-DD |
| Agents | [list] |

---

## Sprint History

| Sprint | Theme | Date | Agents | Chains | Completed | Result | Carry-Forward |
|--------|-------|------|--------|--------|-----------|--------|---------------|
| 01 | [theme] | YYYY-MM-DD | [N] | [N] | [N]/[N] | [ACHIEVED/PARTIAL/NOT] | [N] items |
| 02 | [theme] | YYYY-MM-DD | [N] | [N] | [N]/[N] | [ACHIEVED/PARTIAL/NOT] | [N] items |
| 03 | [theme] | YYYY-MM-DD | [N] | [N] | [N]/[N] | [ACHIEVED/PARTIAL/NOT] | [N] items |

---

## Carry-Forward Tracker

Items that were deferred from one sprint and must be picked up in a future sprint.

| Item | From Sprint | Priority | Assigned Sprint | Status | Age (sprints deferred) |
|------|-------------|----------|-----------------|--------|----------------------|
| [description] | 01 | P1 | 02 | RESOLVED | 1 |
| [description] | 01 | P2 | 03 | PENDING | 2 |
| [description] | 02 | P1 | 03 | IN PROGRESS | 1 |
| [description] | 02 | P3 | — | UNASSIGNED | 2+ |

**Rules:**
- P1 carry-forward items MUST be assigned to the next sprint
- P2 items can defer once. If deferred twice, escalate to P1
- P3 items can defer indefinitely but review quarterly
- Items deferred 3+ sprints → flag for operator review (is this still relevant?)

---

## Agent Performance Across Sprints

| Agent | Sprint 01 | Sprint 02 | Sprint 03 | Trend |
|-------|-----------|-----------|-----------|-------|
| BACKEND | PASS | PASS | PARTIAL | ↓ |
| FRONTEND | PASS | PASS | PASS | → |
| DATA | PARTIAL | PASS | PASS | ↑ |
| QA | PASS | PASS | PASS | → |

Track patterns:
- Agent consistently PARTIAL → scope might be too large, consider splitting
- Agent consistently PASS → reliable, can handle more chains
- Agent FAIL → investigate root cause (bad traces? wrong territory? tool issue?)

---

## Success Criteria History

| Sprint | Criteria | Result |
|--------|----------|--------|
| 01 | [criterion] | PASS |
| 01 | [criterion] | FAIL |
| 02 | [criterion] | PASS |
| 02 | [criterion] | PASS |
| 03 | [criterion] | PASS |
| 03 | [criterion] | PARTIAL |

---

## Pattern Log

Observations that apply across sprints. Update this as you learn.

| Pattern | Discovered | Impact | Action |
|---------|-----------|--------|--------|
| [e.g., "BACKEND gets too many chains when both API and service work exists"] | Sprint 02 | Agent overload → PARTIAL grade | Split into BACKEND-API and BACKEND-SERVICE for large sprints |
| [e.g., "RED TEAM finds auth issues every sprint"] | Sprint 01-03 | Recurring security gaps | Add auth review to QA's standard chains |
| [e.g., "Merge conflicts always happen in package.json"] | Sprint 01-03 | Merge friction | INFRA owns package.json exclusively |

---

## How to Use This Registry

**Before planning a new sprint:**
1. Check carry-forward tracker — P1 items from last sprint MUST be included
2. Check agent performance — adjust agent scope based on trends
3. Check pattern log — avoid repeating known problems
4. Check success criteria history — are we meeting our goals across sprints?

**After completing a sprint:**
1. Add sprint to history table
2. Update carry-forward tracker (resolve completed items, add new deferrals)
3. Update agent performance table
4. Add success criteria results
5. Add any new patterns to the pattern log
