# Slots

Interview targets, not a script.

## Descriptions — read this first

`{{DESCRIPTION}}` and `{{PLAN_DESCRIPTION}}` state **triggering conditions only**. Never
summarize the workflow.

This is empirical, not stylistic. A description that names the steps becomes a shortcut the
agent follows instead of reading the body — a skill described as doing "review between
tasks" produced one review where the body specified two. The generated plan skill has three
gates whose entire value is the predict-before-implementing detail. An agent that reads
"with verification gates" can produce something gate-shaped without ever loading it.

```
BAD   Interrogate the user about a Terraform change until the design is unambiguous,
      then write a committed spec to docs/specs/.
BAD   Turn an approved spec into a task-by-task plan with verification gates,
      blast radius, and rollback commands.

GOOD  Use when the user is about to change Terraform or GCP IAM and the design isn't
      settled — new modules, state moves, provider upgrades, IAM changes — or runs
      /brainstorm, or asks to be grilled before applying.
GOOD  Use when the user has an approved spec and needs it turned into executable work,
      runs /plan, or asks how to sequence an infrastructure change.
```

Third person. Concrete symptoms the user would actually type. Pushy on triggers, silent on
process.

**The name is generic, so the description carries all the disambiguation.** `brainstorm`
tells the agent nothing about scope. If the user also has a personal `~/.pi/skills/
brainstorm`, these two compete on description alone — so lead with concrete domain symptoms
(`terraform plan`, `for_each`, `useEffect`, `dbt run`) rather than generic design language
that would match anything.

## Brainstorm slots

| Slot | Question | Notes |
| --- | --- | --- |
| `{{DOMAIN_NAME}}` | What domain? | "Terraform on GCP", "React frontends" |
| `{{DOMAIN_NOUN}}` | derived | the unit built: module, model, component, route |
| `{{DESCRIPTION}}` | derived | triggers only — see above |
| `{{PLUGIN_VERSION}}` | read `~/.pi/agent/git/github.com/tegardp/pi-kit/package.json`'s `version` field | not a question — stamps which template generation produced this skill, so a later `/skill:project-init` re-run can tell whether it predates the current templates |
| `{{RECON_STEPS}}` | "What do you open first when changing something here?" | 3–5 bullets, real commands and paths. Good recon kills half the questions before they're asked. |
| `{{DECOMP_SIGNALS}}` | "What requests here are secretly three projects?" | 3–4 bullets |
| `{{TIER_LIGHT_CRITERIA}}` | "What would you just do?" | 1–2 sentences |
| `{{TIER_MEDIUM_CRITERIA}}` | "What makes you slow down?" | 1–2 sentences |
| `{{TIER_DEEP_CRITERIA}}` | "What has bitten you?" | The tier that saves real money |
| `{{QUESTION_AXES}}` | built across the interview | 4–6 numbered axes |
| `{{DOMAIN_TRAPS}}` | "What has someone got wrong more than once?" | 2–4 bullets. Highest-signal question — ask it even if you skip others. |
| `{{DESIGN_SECTIONS}}` | "What must a design cover before you'd trust it?" | 4–6 items |
| `{{DOMAIN_SPECIFIC_SECTIONS}}` | same | 1–3 extra spec sections, heading plus one sentence |
| `{{VERIFICATION_NOTE}}` | "How do you know it worked?" | 1–2 sentences |
| `{{SPEC_DIR}}` | check the repo first | default `docs/specs/` |
| `{{COMMIT_PREFIX}}` | read `git log` | default `docs(spec):` |

## Plan slots

Two fork questions first — they change what everything else can be.

> Does this domain let you see what a change *will* do before it does it — a plan, dry run,
> compile step, migration preview? Or is the prediction a failing test?

Both preserve the principle: state the expected outcome, stop when reality diverges. Only
the mechanism differs. Never force dry-run vocabulary onto a domain without one — a gate
that can't be run gets silently skipped.

> When a change goes wrong, does `git revert` plus a rebuild fix it, or does something stay
> broken until someone fixes it by hand?

If reverting is the whole story, set `{{RISK_SECTION}}`, `{{RISK_FIELDS}}` and
`{{FEATURE_ROLLBACK}}` empty. Blast-radius blocks in a fully reversible domain are ceremony,
and ceremony teaches the reader to skim.

| Slot | Question |
| --- | --- |
| `{{PLAN_DESCRIPTION}}` | derived — triggers only |
| `{{GATE_STATIC}}` | "Fastest check that catches a typo before anything runs?" |
| `{{GATE_PREDICT}}` | fork question one |
| `{{GATE_CONVERGE}}` | "How do you prove it works from outside, and that you broke no neighbour?" |
| `{{TASK_BOUNDARY_RULE}}` | "Smallest change you'd send for review alone?" — the line that stops layer-shaped tasks |
| `{{RISK_SECTION}}` `{{RISK_FIELDS}}` `{{FEATURE_ROLLBACK}}` | fork question two, or empty |
| `{{PLAN_DIR}}` `{{PLAN_COMMIT_PREFIX}}` | defaults `docs/plans/`, `docs(plan):` |
| `{{WORKED_EXAMPLE_NOTE}}` | you write it — one or two fully worked task blocks with real commands and expected output. A template with abstract gates produces plans with abstract gates. |

## Rationalization slots

Both generated skills enforce discipline: *don't implement until approved*. Discipline
skills need explicit loophole closure, because an agent under pressure will find one and
the reasoning will sound good.

Ask: **"When has an agent — or you — talked yourself out of planning here?"** Then fill
`{{RATIONALIZATIONS}}` as an excuse/reality table and `{{RED_FLAGS}}` as a short list of
phrases meaning stop. Use real ones; a strawman row teaches nothing.

```
| "The user said just build it"         | They asked for a result, not a shortcut. Offer
                                          Light tier: three questions, then build.
| "It's Light tier so this is optional" | Light is a shorter process, not no process.
| "I'll write the spec after"           | A spec derived from an implementation documents
                                          what you did, not what was decided.
| "The design is obvious"               | Then presenting it costs one turn.

RED_FLAGS
- Writing any file that isn't the spec
- "I'll just scaffold while we talk"
- Two questions in one message
- Answering your own question and moving on
```

## Eval slot

`{{EVAL_PROMPTS}}` — three prompts for `evals/pressure-tests.md`, one per failure mode:

- **Pressure** — urgency plus a plausible reason to skip the process
- **Scope** — a request that is secretly two or three projects
- **Application** — an ordinary request, checking the skill produces something *useful*
  rather than merely compliant

Each gets a one-line pass criterion.

## Worked contrast — same slots, two domains

```
                 React app ("vibecode")            Terraform / Ansible
TASK_BOUNDARY    One user-visible behaviour end     One working service. A role that
                 to end. "All the types" is not     provisions storage and configures
                 a task.                            DNS is two roles.

GATE_PREDICT     No dry run. Write the test first   terraform plan / --check --diff with
                 and name the exact failure —       the diff predicted numerically: "2 to
                 which assertion, which received    add, 0 to change, 0 to destroy — the
                 value. A test failing on a bad     CNAME and the tunnel route." An
                 import has told you nothing.       unpredicted destroy is a stop.

GATE_CONVERGE    Run it for real outside the test,  Apply, re-run, require changed=0 or
                 then the full suite: prior pass    "No changes." Then one external
                 count plus the new tests.          assertion: dig, systemctl, curl.

RISK_SECTION     (empty)                            DNS, remote access, shared storage
                                                    are self-blocking. One node first
                                                    via --limit.
```

Same three gates, same principle, different commands. That difference is why these are
slots and not prose.
