# Sprint Status Tracking

> How to monitor what every agent is doing — and catch problems before they compound

---

## Why Track Status

Running multiple agents simultaneously creates cognitive overload without a system. Without status tracking, you lose visibility into:

- Which agents are blocked (and holding up the whole wave)
- Which agents have wandered outside their territory
- Which chains are done vs. "done but not verified"
- Which merges are ready vs. which need more work
- Whether you're on track to complete the sprint goal

Status tracking is the operator's **situational awareness**. It transforms parallel workstreams into a single picture you can reason about.

Without it, you will:
- Start Wave 2 before Wave 1 is actually complete
- Merge work that hasn't been verified
- Miss P0 discoveries sitting in completion reports
- Discover mid-merge that two agents touched the same file
- Lose track of which blockers were resolved

The cost of 5 minutes checking status every 30 minutes is far lower than untangling a broken merge.

---

## Agent Status States

| State | Symbol | Meaning |
|-------|--------|---------|
| IDLE | ⏸ | Not yet dispatched or between waves |
| ACTIVE | ▶ | Working on assigned chains |
| BLOCKED | 🔴 | Waiting on another agent or external input |
| REVIEW | 👀 | Work complete, awaiting operator review |
| COMPLETE | ✅ | All chains done, completion report filed |
| FAILED | ❌ | Crashed, disconnected, or unrecoverable |

**Important distinctions:**

- **ACTIVE** means the agent is working. It does not mean the work is correct.
- **REVIEW** means the agent reports it's done. It does not mean it passed verification.
- **COMPLETE** means you (the operator) have reviewed the completion report and confirmed all chains are verified, not just claimed.
- **FAILED** means you need to intervene. See the [Troubleshooting](#troubleshooting) section in OPERATORS-GUIDE.md.

---

## Chain Progress States

Agents work through chains sequentially. Each chain has its own state:

| State | Meaning |
|-------|---------|
| QUEUED | Chain assigned but not started |
| TRACING | Agent is reading code, following the execution trace |
| FIXING | Agent is implementing the change |
| VERIFYING | Agent is testing the fix (build, test, manual check) |
| COMPLETE | Chain done — fix verified, documented |
| BLOCKED | Chain waiting on external input or another agent |
| PARKED | Chain temporarily set aside for a higher-priority chain |

Agents should never be in FIXING state for Chain 2 while Chain 1 is still TRACING. That's context-switching. If you see it, intervene.

---

## Agent Status Board

Maintain this table as a scratch file or shared doc. Update every 15–30 minutes during active sprints.

```
| Agent    | Status   | Current Chain         | Chains Done | Chains Left | Blockers       | Last Check |
|----------|----------|-----------------------|-------------|-------------|----------------|------------|
| DATA     | ACTIVE   | Chain 2 (FIXING)      | 1/4         | 3           | None           | 14:30      |
| QA       | ACTIVE   | Chain 1 (TRACING)     | 0/3         | 3           | None           | 14:25      |
| INFRA    | COMPLETE | —                     | 2/2         | 0           | None           | 14:20      |
| DESIGN   | REVIEW   | —                     | 1/1         | 0           | Awaiting check | 14:15      |
| BACKEND  | IDLE     | —                     | 0/3         | 3           | Wave 2         | —          |
| SERVICES | IDLE     | —                     | 0/2         | 2           | Wave 2         | —          |
| FRONTEND | IDLE     | —                     | 0/4         | 4           | Wave 3         | —          |
| RED TEAM | IDLE     | —                     | 0/4         | 4           | Wave 4         | —          |
| LEAD     | IDLE     | —                     | 0/3         | 3           | Wave 5         | —          |
```

**Columns explained:**

- **Chains Done** — Format is `completed/total`. "2/4" means 2 of 4 chains complete.
- **Blockers** — "Wave 2" means waiting for wave transition. "Needs DATA Chain 3" means a specific dependency. "None" means clear.
- **Last Check** — When you last verified this agent's actual state (not self-reported).

How to verify an agent's state without asking it:

```bash
# Read partial completion report
cat /path/to/your-project-data/docs/agent-dispatch/sprint-01/agent-data-completion.md

# Check recent commits
git -C /path/to/your-project-data log --oneline -5

# Check what's been modified
git -C /path/to/your-project-data diff --stat
```

---

## Wave Progress Tracker

Track wave status alongside agent status:

```
Wave 1: [▶ ACTIVE]   DATA(2/4) QA(0/3) INFRA(2/2 ✅) DESIGN(1/1 👀)
Wave 2: [⏸ WAITING]  BACKEND(0/3) SERVICES(0/2)
Wave 3: [⏸ WAITING]  FRONTEND(0/4)
Wave 4: [⏸ WAITING]  RED TEAM(0/4)
Wave 5: [⏸ WAITING]  LEAD(0/3)
```

### Wave Transition Rules

A wave is complete when **every agent in that wave** reaches COMPLETE status — not REVIEW, not "they said they're done."

```
Wave N COMPLETE criteria:
  ✅ Every agent in wave N has status COMPLETE
  ✅ Every agent's completion report has been reviewed by operator
  ✅ No unresolved P0 discoveries from wave N agents
  ✅ Build passes on each agent's branch (pre-merge check)
```

Only after all criteria are met do you dispatch Wave N+1.

**Why not start early?** Wave N+1 agents often need Wave N's output to be stable before they can trace their own chains accurately. Starting BACKEND before DATA completes means BACKEND may trace incorrect root causes.

### Transition Checklist (Wave N → Wave N+1)

Before dispatching the next wave:

- [ ] All Wave N agents show COMPLETE (not just REVIEW)
- [ ] All completion reports reviewed — no action items outstanding
- [ ] P0 discoveries triaged and assigned
- [ ] Wave N branches build cleanly
- [ ] Decide: merge Wave N into main before dispatching Wave N+1, or let Wave N+1 work from main?

---

## Merge Readiness Checklist

Per-agent checklist before merging their branch. Run this for each agent in merge order (DATA first, LEAD last):

- [ ] All chains COMPLETE — not just "done", but verified by you reading the completion report
- [ ] Completion report filed at `docs/agent-dispatch/sprint-XX/agent-X-completion.md`
- [ ] No unresolved P0 discoveries (triaged and assigned elsewhere is fine; ignored is not)
- [ ] Build passes on the agent's branch: `[your-build-command]`
- [ ] Tests pass on the agent's branch: `[your-test-command]`
- [ ] No files modified outside territory — run `git diff --name-only main..sprint-XX/agent` and compare to territory spec
- [ ] No uncommitted changes: `git -C /path/to/worktree status --short`

If any item fails, do not merge. Either fix the issue or document why you're overriding the check.

---

## Sprint Health Indicators

Check sprint health at every status check. This is your overall assessment:

| Health | Criteria | Action |
|--------|----------|--------|
| 🟢 GREEN | All active agents working. No blockers. No P0 discoveries. Wave transitions happening on schedule. | Continue monitoring every 30 min. |
| 🟡 YELLOW | 1+ agent blocked. Minor scope creep detected. Wave behind schedule. P2 discoveries needing triage. | Investigate blockers immediately. Consider reassigning chains. Check every 15 min. |
| 🔴 RED | P0 discovered. 2+ agents blocked simultaneously. Build broken on main. Agent crashed or disconnected. Wave transition failing. | Stop all work. Assess. Intervene directly. |

### What RED looks like in practice

- DATA agent discovers data corruption during Chain 2 — RED immediately
- BACKEND and SERVICES both blocked on the same missing interface — RED
- You merge DATA and the build breaks — RED (revert, fix, retry)
- An agent stops responding and you can't reach it — RED

When RED: stop dispatching new work. Resolve the immediate issue. Resume only when you're back to YELLOW or GREEN.

---

## Quick Status Check Script

Run this at the start of each status check to get a fast overview of all agent worktrees:

```bash
#!/bin/bash
# Quick status of all agent worktrees
# Usage: ./check-status.sh (run from your main project directory)

SPRINT="sprint-01"
PROJECT_NAME="$(basename $(pwd))"
PARENT_DIR="$(dirname $(pwd))"

echo "=== Agent Dispatch Status: $PROJECT_NAME ==="
echo "Sprint: $SPRINT | $(date '+%Y-%m-%d %H:%M')"
echo ""

for agent in data design backend services frontend infra qa lead; do
  DIR="$PARENT_DIR/${PROJECT_NAME}-${agent}"
  if [ -d "$DIR" ]; then
    BRANCH=$(git -C "$DIR" branch --show-current 2>/dev/null)
    COMMITS=$(git -C "$DIR" log main..$BRANCH --oneline 2>/dev/null | wc -l | tr -d ' ')
    UNCOMMITTED=$(git -C "$DIR" status --short 2>/dev/null | wc -l | tr -d ' ')
    REPORT="$DIR/docs/agent-dispatch/$SPRINT/agent-${agent}-completion.md"
    REPORT_STATUS=$([ -f "$REPORT" ] && echo "report:✅" || echo "report:—")
    echo "[$agent] branch: $BRANCH | commits ahead: $COMMITS | uncommitted: $UNCOMMITTED | $REPORT_STATUS"
  else
    echo "[$agent] NOT DISPATCHED — worktree missing at $DIR"
  fi
done

echo ""
echo "Check completion reports for REVIEW/COMPLETE agents."
echo "Run 'git -C <worktree> diff --name-only main..<branch>' to verify territory."
```

What each column means:

- **commits ahead** — how many commits this agent has made on its branch
- **uncommitted** — files modified but not committed (agent may still be working)
- **report** — whether a completion report has been written

---

## How Often to Check

Match check frequency to sprint phase:

| Sprint Phase | Check Frequency | What to Look For |
|-------------|----------------|-------------------|
| Wave active (first 30 min) | Every 10 min | Agents starting correctly. No immediate blockers. Territory being respected. |
| Wave active (steady state) | Every 30 min | Chain progress. Blocked agents. P0 discoveries filed. |
| Wave nearing completion | Every 15 min | Are agents actually finishing or just claiming done? |
| Wave transition | Immediately before dispatching next wave | All Wave N criteria met. No outstanding P0s. |
| Merge phase | After each merge | Build + tests pass. No regressions from the merge. |
| Post-sprint | Once, after LEAD completes | Worktree cleanup. Final completion reports archived. |

**Rule of thumb:** More frequent checking early when you're unsure how agents are performing. Less frequent once you trust they're on track. Always check before a wave transition.

---

## Status Tracking Without a Shared Doc

If you're running the sprint solo without a team tracking document, maintain the status board as a scratch file:

```bash
# Create a local scratch file at sprint start
touch ~/sprint-01-status.md

# Update it by hand every 30 min
# Or pipe the check script output to it:
./check-status.sh >> ~/sprint-01-status.md
echo "---" >> ~/sprint-01-status.md
```

The goal isn't ceremony — it's to have a single place you can look at to answer "where is each agent right now?" without having to check 8 terminals.

---

## Related Documents

- [TEMPLATE-STATUS.md](../templates/status.md) — Copy-paste sprint status board template
- [OPERATORS-GUIDE.md](../guides/operators-guide.md) — Full sprint lifecycle guide
- [METHODOLOGY.md](../core/methodology.md) — Chain execution, priority levels, escalation protocol
- [agents/](../agents/) — Agent role definitions and territories
- [TEMPLATE-COMPLETION.md](../templates/completion.md) — What a completion report should contain
