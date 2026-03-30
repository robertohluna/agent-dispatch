# Post-Sprint Checklist

> Run through this after LEAD merges and before closing the sprint.
> Every item must be YES before the sprint is marked COMPLETE.
> Mirror of preflight-checklist.md — this is the teardown side.

---

## After Merge

### Validation
- [ ] Final build passes on main (after all merges)
- [ ] Final test suite passes on main
- [ ] Lint passes on main
- [ ] No regressions introduced (compare test count before/after)

### Documentation
- [ ] LEAD's sprint summary exists (`sprint-XX/SPRINT-SUMMARY.md`)
- [ ] All agent completion reports collected (`sprint-XX/agent-X-completion.md`)
- [ ] RED TEAM findings report exists (`sprint-XX/red-team-findings.md`)

---

## ORCHESTRATOR Assessment (Wave 6)

- [ ] ORCHESTRATOR has read all completion reports
- [ ] ORCHESTRATOR has read RED TEAM findings
- [ ] ORCHESTRATOR has graded every dispatched agent
- [ ] `sprint-XX/SPRINT-ASSESSMENT.md` exists and is complete
- [ ] Per-agent grades assigned with justification
- [ ] Mission result stated (ACHIEVED / PARTIALLY ACHIEVED / NOT ACHIEVED)
- [ ] Merge recommendation documented
- [ ] Carry-forward items documented

---

## Carry-Forward

- [ ] Carry-forward items extracted from:
  - [ ] Agent completion reports (parked chains, blockers, P0 discoveries)
  - [ ] RED TEAM findings (unresolved CRITICAL/HIGH)
  - [ ] Sprint assessment (failed criteria, recommendations)
- [ ] Each carry-forward item has: description, source, priority, reason deferred
- [ ] Priority escalation rules applied (P1 deferred twice → P0, etc.)
- [ ] Sprint registry updated (`sprints/REGISTRY.md`)
  - [ ] Sprint added to history table
  - [ ] Carry-forward tracker updated
  - [ ] Agent performance table updated
  - [ ] Success criteria results added

---

## Success Criteria

- [ ] Every success criterion from DISPATCH.md evaluated
- [ ] Each criterion marked PASS or FAIL with evidence
- [ ] Results recorded in sprint assessment and registry

---

## Cleanup

### Worktrees
```bash
PROJECT_DIR="$(pwd)"
PARENT_DIR="$(dirname $PROJECT_DIR)"
PROJECT_NAME="$(basename $PROJECT_DIR)"
SPRINT="sprint-XX"

for agent in [list all dispatched agents]; do
  git worktree remove "$PARENT_DIR/${PROJECT_NAME}-${agent}" 2>/dev/null
done
```
- [ ] All worktrees removed

### Branches
```bash
for agent in [list all dispatched agents]; do
  git branch -d $SPRINT/$agent 2>/dev/null
done
```
- [ ] All sprint branches deleted (or archived if you keep them)

### Archive
- [ ] Sprint directory (`sprint-XX/`) committed to main
- [ ] All sprint docs (DISPATCH, task docs, completion reports, assessment) preserved

---

## Release (if shipping)

- [ ] Version bumped (if applicable)
- [ ] CHANGELOG updated
- [ ] Git tag created (`git tag -a vX.Y.Z -m "Sprint XX: [theme]"`)
- [ ] Deployed to staging/production (if applicable)
- [ ] Smoke test passed in target environment

---

## Sprint Close

- [ ] All items above are YES
- [ ] Sprint status in registry updated to COMPLETE
- [ ] Operator has signed off

**Sprint [XX] is now COMPLETE.**

---

## File Inventory — Complete Sprint

When a sprint is fully closed, these files should exist:

```
sprint-XX/
├── codebase-analysis.md              ← ORCHESTRATOR Phase 1
├── DISPATCH.md                       ← ORCHESTRATOR Phase 6
├── DISPATCH-PROMPTS.md               ← Archived activation prompts (optional)
│
├── agent-A-backend.md                ← Task docs (one per agent)
├── agent-B-frontend.md
├── agent-[X]-[domain].md
│
├── agent-A-completion.md             ← Completion reports (one per agent)
├── agent-B-completion.md
├── agent-[X]-completion.md
│
├── red-team-findings.md              ← RED TEAM
├── SPRINT-SUMMARY.md                 ← LEAD
└── SPRINT-ASSESSMENT.md              ← ORCHESTRATOR Wave 6
```

Missing file = incomplete sprint record. Go back and produce it.
