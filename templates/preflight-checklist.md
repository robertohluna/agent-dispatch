# Pre-Flight Checklist

> Run through this before pasting activation prompts into terminals.
> Every item must be YES before you dispatch.

---

## Before Dispatching

### Sprint Plan
- [ ] DISPATCH.md reviewed and approved
- [ ] Agent count matches the work (not dispatching 9 agents for 3 chains)
- [ ] Wave assignments make sense (dependencies respected)
- [ ] Merge order defined
- [ ] Success criteria clear and measurable

### Agent Docs
- [ ] Per-agent task doc exists for every dispatched agent
- [ ] Each task doc has execution traces with vectors, signals, fixes, verification
- [ ] Territory boundaries defined (CAN modify / DO NOT touch)
- [ ] Cross-agent context included (what other agents are building)
- [ ] Context reading list includes `config/code-standards.md`

### Activation Prompts
- [ ] One prompt per agent, self-contained
- [ ] Each prompt follows the 11-step assembly order from `config/dispatch-styles.md`
- [ ] Intensity protocol appended to every prompt
- [ ] Optimal path block (BEFORE/WHILE/AFTER) appended to every prompt
- [ ] Mesh mode triggers included
- [ ] Build + test commands specified for the project's stack
- [ ] Completion report requirement stated

### Environment
- [ ] Git worktrees created (one per agent)

```bash
SPRINT="sprint-XX"
PROJECT_DIR="$(pwd)"
PARENT_DIR="$(dirname $PROJECT_DIR)"
PROJECT_NAME="$(basename $PROJECT_DIR)"

for agent in [list your agents]; do
  git branch $SPRINT/$agent main 2>/dev/null || true
  git worktree add "$PARENT_DIR/${PROJECT_NAME}-${agent}" $SPRINT/$agent
done
```

- [ ] Dependencies installed in each worktree
- [ ] Terminals open (one per agent)
- [ ] Each terminal is in the correct worktree directory

### Wave Discipline
- [ ] Wave 1 agents ready to dispatch immediately
- [ ] Wave 2+ agents queued — DO NOT paste until prior wave completes
- [ ] Status board created (`templates/status.md`)

---

## Dispatch Order

```
1. Paste Wave 1 prompts (DATA, QA, INFRA, DESIGN)     → agents start working
2. Monitor Wave 1 progress                              → update status board
3. ALL Wave 1 agents report COMPLETE                    → verify completion docs
4. Paste Wave 2 prompts (BACKEND, SERVICES)             → agents start working
5. Monitor Wave 2 progress                              → update status board
6. ALL Wave 2 agents report COMPLETE                    → verify completion docs
7. Paste Wave 3 prompt (FRONTEND)                       → agent starts working
8. Wave 3 COMPLETE                                      → verify completion doc
9. Paste Wave 4 prompt (RED TEAM)                       → adversarial review
10. Wave 4 COMPLETE                                     → review findings
11. Paste Wave 5 prompt (LEAD)                          → merge + ship
12. SPRINT COMPLETE                                     → cleanup worktrees
```

---

## After Sprint

- [ ] All completion reports collected
- [ ] RED TEAM findings reviewed — blocking issues resolved
- [ ] LEAD merged all branches in dependency order
- [ ] Build + test passed after every merge
- [ ] Sprint summary written
- [ ] Worktrees cleaned up: `git worktree remove ...`
- [ ] Sprint branches cleaned up: `git branch -d ...`
