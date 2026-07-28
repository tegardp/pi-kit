# Dispatch

Read when running anything other than plain sequential.

## The subagent brief

Plan tasks are written to be read in isolation — that is the property the plan skill
enforces, and it is what makes parallel dispatch possible at all. Give each subagent
exactly its own task block plus the Global Constraints, and nothing else. Adding "context
from the conversation" defeats the isolation the plan was written for, and if a task
genuinely cannot be executed from its own block, that is a plan defect worth reporting.

```
Execute one task from an implementation plan. Do not read other tasks.

GLOBAL CONSTRAINTS
<verbatim from the plan>

YOUR TASK
<verbatim task block: description, files, steps, all three gates, commit line>

RULES
- Follow the steps in order. Run all three gates in order.
- Gate 2: run it BEFORE implementing and confirm the predicted failure.
  If actual output differs from either prediction, stop and report both. Do not fix it.
- Touch only the files named in your task.
- Do not commit. Report your diff and gate output.
- Stuck on how to implement: read the repo, then the docs, three attempts, then report.
```

**Subagents do not commit.** The main session commits after each wave, in task order. Two
agents committing into one working tree interleaves changes and produces a history where no
commit corresponds to a task — which destroys the resume mechanism.

## Verifying a wave

After a wave returns, before committing: re-run each task's Gate 1 and Gate 3 in the main
session. Subagents can pass in isolation and still conflict — two disjoint file sets can
both compile alone and fail together on a shared type, a shared route table, a shared
lockfile.

A wave that fails re-verification collapses to sequential. Re-run its tasks one at a time
rather than debugging the interaction; the sequential run tells you which task actually
broke, and it costs less than the analysis.

## Worktrees

When tasks are file-disjoint but you want isolation stronger than trust, give each a
worktree:

```bash
git worktree add ../wt-task-3 -b exec/task-3
# ... run, verify, then from the main tree:
git merge --no-ff exec/task-3
git worktree remove ../wt-task-3
```

This is usually not worth it. It costs a full environment per worktree — dependencies,
build cache, generated files — and the merge can fail in ways the isolation was supposed to
prevent. Reach for it when tasks are long and genuinely independent, or when a task needs a
dev server or a port that a sibling also needs. For most plans, sequential is faster than
worktrees once setup is counted.

## When parallelism is not worth it

- Fewer than three parallelizable tasks. The dispatch and re-verification overhead exceeds
  the saving.
- Tasks under about five minutes each.
- Any shared migration, state file, or lockfile anywhere in the wave.
- A plan whose tasks were sliced by layer rather than by deliverable — those are never
  independent, whatever the file lists suggest. Report the slicing as a plan defect.