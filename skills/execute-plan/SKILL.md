---
name: execute-plan
description: Use when the user runs /skill:execute-plan, has a committed plan in docs/plans/ and wants it built, is resuming a half-finished plan after an interruption, or asks whether plan tasks can run in parallel.
---

# Execute Plan

Execute a committed plan task by task. The plan is the authority; you are not redesigning
it.

## The one hard rule

Implement only what the plan says. If the plan is wrong, stop and report — do not fix it in
flight. A plan corrected during execution is a plan nobody reviewed, and the divergence
compounds silently across every task after it.

## Step 1 — Read the plan and locate yourself

Read the plan in full, including Global Constraints, before touching anything. Given no
path, `ls docs/plans/` and ask which; never guess from recency when several exist.

If `.pi/skills/_recon.md` exists and its named config file hasn't changed since, take
the verify command and stack facts from it rather than re-deriving them by exploring the
repo — the plan itself should already carry everything task-specific.

Then find where you are: `git log --oneline -20`. The plan commits once per task, so the
last task commit is your resume point. No commits yet means start at Task 1. Say which task
you're starting from and why before you begin.

Open questions still unresolved in the plan? Stop. Planning around an undecided requirement
produces work that gets thrown away, and executing it wastes the throw-away twice.

## Step 2 — Decide sequential, parallel, or waves

Build a dependency graph from what the plan already gives you: the file structure table,
each task's Files line, and each Gate 2 prediction.

Two tasks may share a wave only when **all** hold:

- **Disjoint files.** No overlap in files created or modified.
- **No output dependency.** Neither task's Gate 2 prediction references something the other
  produces.
- **No shared exclusive resource.** Same Terraform state, same migration sequence, same
  port, same fixture database, same lockfile — any one of these forces sequence.
- **Neither is in the other's blast radius**, where the plan carries that section.

Then:

- **Sequential** — the common case, and the correct default when anything is uncertain.
  Ordering is cheap; a mid-wave collision is not.
- **Parallel** — one wave, when every task passes all four tests against every other.
- **Waves** — the mixed case, and usually what a real plan produces. Topologically sort into
  waves; parallel within a wave, sequential between them. Tasks 2 and 3 build independent
  modules, Task 4 wires them together: wave one is `{2,3}`, wave two is `{4}`.

Cap width at three. Beyond that the review burden on you exceeds the time saved, and
contention starts producing failures that look like code bugs.

State the plan — "Sequential; tasks 3 and 4 touch the same state file" or "Two waves:
{2,3} then {4}" — with the reason, before you start. One line.

Dispatch mechanics and the worktree case: read `references/dispatch.md`.

## Step 3 — Run each task

Follow the task block exactly: steps in order, then all three gates in order, then the
commit the plan specifies. Never skip a gate because the previous one passed, and never
reorder them — Gate 1 exists to make Gate 2's failure legible.

Gate 2 is the one that matters. Run it **before** implementing and confirm you get the
predicted failure. A test failing for the wrong reason has told you nothing, and a `plan`
that errors instead of showing an empty diff is not the same as a clean prediction.

After the commit, dispatch `code-reviewer` with scope `task`, the spec path, the plan path,
and this task's commit alone. Read its report before moving to the next task — a **Must
fix** finding is treated exactly like a Gate 2 divergence: stop and surface it rather than
starting the next task on top of it. This is bounded review (one commit, both passes in
full) rather than the single end-of-plan review over everything at once, which is what
keeps review context proportional to one task instead of the whole plan.

## Step 4 — Autonomy

Solve it yourself, without asking, when the uncertainty is about **how to implement**:

- Read the repo first. Existing code answers "how do we do this here" better than any doc.
- Then the actual documentation — the library's, the provider's, the API reference. Not
  your memory of it.
- Then established practice, weighted toward what this repo already does. A pattern the
  repo uses consistently beats a better pattern it doesn't.

Budget: three attempts, or roughly ten minutes of research per blocker. Then escalate with
what you tried and what each attempt produced. Spinning is not autonomy.

**Escalate immediately, regardless of severity:**

- **Gate 2 diverges from its prediction.** This is the plan being wrong, and documentation
  cannot fix a wrong plan. Report the predicted output, the actual output, and stop.
- **A destructive operation the plan didn't authorize** — an unpredicted `destroy` or
  `replace`, a dropped table, a force-push, deleted data, a state move.
- **Credentials, secrets, or access you don't have.**
- **Anything touching production that the plan does not name.**
- A plan requirement that is impossible or self-contradictory.

**Log and continue, report at the end:** ambiguous wording you resolved one way, a lint rule
you satisfied in a debatable manner, a name the plan used inconsistently but obviously.
These are noise mid-run and useful in aggregate.

The distinction is not severity. It is whether the uncertainty is about your implementation
or about the plan itself. You can research your way out of the first and never out of the
second.

## Step 5 — Report and stop

Per task, one line: task number, gate results, commit SHA. At the end: tasks completed,
tasks remaining, the logged ambiguities, and anything escalated.

Run the plan's feature-level verification if every task finished. If tasks remain, say which
and why — a fresh session resumes from the last commit.

If every task finished and got its per-task review, dispatch `code-reviewer` once more with
scope `closing`, spec path, plan path, and the full commit range. This pass only checks
coherence across tasks and fit with the repo — the things per-task review structurally
cannot see — not a re-review of what each `task`-scope pass already covered. Include its
findings in the final report alongside the feature-level verification.

Do not mark work complete on a gate you skipped or a prediction you rationalized. An
honest partial execution is recoverable; a false green is not.

## Failure modes in yourself

**Fixing the plan in flight.** The strongest pull in this skill. A divergent gate looks like
a small problem you could just handle. It is the plan telling you something nobody checked.

**Parallelizing on optimism.** "These probably don't conflict" is how two agents write the
same file. All four tests, every pair, or sequential.

**Skipping Gate 2's pre-check.** Running it only after implementing turns a prediction into
a confirmation, which proves nothing.

**Silent scope growth.** A refactor the plan didn't ask for, a dependency it didn't name, a
file it didn't list. All three are escalations, not initiative.