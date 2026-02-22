# Sprint [XX] Status Board

> Last updated: [YYYY-MM-DD HH:MM]
> Sprint health: [🟢 GREEN / 🟡 YELLOW / 🔴 RED]

---

## Sprint Overview

- **Theme:** [sprint theme — e.g., "Payment Bug Fix Sprint"]
- **Started:** [date/time]
- **Target completion:** [date/time]
- **Total chains:** [N]
- **Agents dispatched:** [N]
- **Dispatch doc:** `docs/agent-dispatch/sprint-[XX]/DISPATCH.md`

---

## Agent Status Board

| Agent    | Status | Current Chain | Done | Left | Blockers | Last Check |
|----------|--------|--------------|------|------|----------|------------|
| DATA     |        |              | /    |      |          |            |
| DESIGN   |        |              | /    |      |          |            |
| BACKEND  |        |              | /    |      |          |            |
| SERVICES |        |              | /    |      |          |            |
| FRONTEND |        |              | /    |      |          |            |
| INFRA    |        |              | /    |      |          |            |
| QA       |        |              | /    |      |          |            |
| RED TEAM |        |              | /    |      |          |            |
| LEAD     |        |              | /    |      |          |            |

**Status values:** IDLE ⏸ | ACTIVE ▶ | BLOCKED 🔴 | REVIEW 👀 | COMPLETE ✅ | FAILED ❌

**Current Chain format:** `Chain N (STATE)` — e.g., `Chain 2 (FIXING)`

**Chain states:** QUEUED | TRACING | FIXING | VERIFYING | COMPLETE | BLOCKED | PARKED

---

## Wave Progress

- **Wave 1:** [STATUS] — [agents + progress, e.g., DATA(2/4) QA(1/3) INFRA(2/2 ✅) DESIGN(1/1 ✅)]
- **Wave 2:** [STATUS] — [agents + progress]
- **Wave 3:** [STATUS] — [agents + progress]
- **Wave 4:** [STATUS] — [agents + progress]
- **Wave 5:** [STATUS] — [agents + progress]

**Wave status values:** ⏸ WAITING | ▶ ACTIVE | ✅ COMPLETE

**Transition criteria:** Every agent in the wave must be COMPLETE (completion report reviewed, build passes) before the next wave starts.

---

## Blocking Issues

| # | Agent | Description | Impact | Resolution |
|---|-------|-------------|--------|------------|
| 1 |       |             |        |            |
| 2 |       |             |        |            |

---

## P0 Discoveries

Critical issues found during sprint work. Each must be triaged before the sprint can continue.

| # | Found By | Description | Assigned To | Status |
|---|----------|-------------|-------------|--------|
| 1 |          |             |             |        |
| 2 |          |             |             |        |

**P0 status values:** OPEN | ASSIGNED | IN PROGRESS | RESOLVED | DEFERRED

---

## Merge Queue

Merge in this order. Build + test after each merge. Do not skip ahead.

| Order | Agent    | Branch               | Ready? | Merged? | Build OK? | Tests OK? |
|-------|----------|----------------------|--------|---------|-----------|-----------|
| 1     | DATA     | sprint-[XX]/data     |        |         |           |           |
| 2     | DESIGN   | sprint-[XX]/design   |        |         |           |           |
| 3     | BACKEND  | sprint-[XX]/backend  |        |         |           |           |
| 4     | SERVICES | sprint-[XX]/services |        |         |           |           |
| 5     | FRONTEND | sprint-[XX]/frontend |        |         |           |           |
| 6     | INFRA    | sprint-[XX]/infra    |        |         |           |           |
| 7     | QA       | sprint-[XX]/qa       |        |         |           |           |
| 8     | LEAD     | sprint-[XX]/lead     |        |         |           |           |

**Ready** = completion report reviewed, all chains verified, no outstanding P0s, no uncommitted changes.

---

## Ship Decision

Complete this before tagging the sprint:

- [ ] All success criteria from DISPATCH.md met
- [ ] Build passes on main
- [ ] Tests pass on main
- [ ] No unresolved CRITICAL issues in any completion report
- [ ] Progress tracker updated with sprint outcomes
- [ ] Worktrees cleaned up

If all checked: `git tag sprint-[XX]-complete && git push origin main --tags`

---

## Notes

[Free-form notes, decisions, observations, interventions made during the sprint]
