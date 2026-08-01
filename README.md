# pi-kit

A [pi](https://github.com/earendil-works/pi) package that bundles skills, agents, themes, and extensions for structured development workflows.

## Installation

### Prerequisites

pi-kit is a package for [pi](https://github.com/earendil-works/pi), so pi itself must be
installed first:

```bash
npm install -g @earendil-works/pi-coding-agent
pi --version
```

### Install pi-kit

```bash
pi install git:github.com/tegardp/pi-kit
```

Then restart pi. The package auto-loads everything declared in `package.json` → `pi`:
skills, agents, themes, and extensions — no further configuration needed.

Verify it loaded by running `/skill:project-init` or checking that `pi-kit-night` appears
in the theme picker.

### Updating

Already installed pi-kit and want the latest version (new skills, agents, or bundled
extensions)?

```bash
pi update git:github.com/tegardp/pi-kit
```

or update everything you have installed at once:

```bash
pi update --all
```

Then restart pi.

### Local / development install

To work on pi-kit itself, clone it and point pi at the local path instead of the GitHub repo:

```bash
git clone https://github.com/tegardp/pi-kit.git
cd pi-kit
npm install
pi install .
```

Pull latest and run `npm install` again to pick up dependency changes, then restart pi.

## What's included

| Thing | Type | What it does |
|---|---|---|
| `execute-plan` | skill | Builds a committed plan task by task, with gates and per-task review |
| `project-init` | skill | Generates `brainstorm` and `plan` skills tuned to your repo's stack |
| `plan` / `spec` / `brainstorm` | skill templates | Fill-and-write templates produced by `project-init` |
| `code-reviewer` | agent | Reviews diffs against spec and plan; two-pass, severity-ranked report |
| `pi-kit-night` | theme | Dark Tokyo Night-derived theme |
| `pi-permission-system` | extension | Command/path permission rules with `yoloMode` support, configured per project or globally |
| `pi-simplify` | extension | — |
| `pi-web-access` | extension | — |
| `pi-subagents` | extension | Multi-agent dispatch |
| `pi-9router-ext` | extension | Connects to a [9router](https://github.com/irfansofyana/pi-9router-ext) AI routing proxy instance |

## Skills

### `/skill:execute-plan`

Execute a committed plan from `docs/plans/` task by task. Determines sequential vs parallel
execution, runs gates (including Gate 2 prediction verification), dispatches per-task code
review, and produces a final report.

- **Trigger:** `/skill:execute-plan`, resuming a half-finished plan, or asking whether
  tasks can run in parallel.

### `/skill:project-init`

Interview-driven generator that produces `brainstorm` and `plan` skills tuned to your
repo's stack, test command, commit convention, and domain-specific failure modes. Writes to
`.pi/skills/` so skills version alongside the repo.

- **Trigger:** `/skill:project-init`, or asking for a brainstorm/spec/plan skill for a
  specific stack.

## Agent: code-reviewer

A read-only reviewer agent that checks implemented code against its spec and plan. Run it
after a task finishes, or on any diff that needs correctness/scope/quality review before
trusting it.

- **Scope `task`:** Full two-pass review on a single task commit.
- **Scope `closing`:** Lighter pass across all task commits — coherence and repo fit only.

## Theme: pi-kit-night

Dark theme with Tokyo Night palette. Activate in pi's settings or via the theme picker.

## Permission system config

The bundled `pi-permission-system` ships a global config at `config/pi-permission-system/config.json`
with sensible defaults: secrets blocked (`.env`, `.ssh`, `.aws`, `.pem`, keys), destructive
bash denied or gated, and package manager commands allowed.

### Project-specific overrides

Create `<project>/.pi/extensions/pi-permission-system/config.json`. It fully replaces the
global config for that project (no merge). See `config/pi-permission-system/README.md` for
examples including safe-yolo, strict, and total-yolo profiles.

```bash
mkdir -p .pi/extensions/pi-permission-system
# copy and edit:
cp ~/.pi/agent/git/github.com/tegardp/pi-kit/config/pi-permission-system/config.json \
   .pi/extensions/pi-permission-system/config.json
```

## Project structure

```
pi-kit/
├── agents/                  # Agent definitions
│   └── code-reviewer.md
├── config/
│   └── pi-permission-system/
│       ├── config.json      # Global permission defaults
│       └── README.md        # Permission setup guide
├── skills/
│   ├── execute-plan/        # Plan execution skill
│   ├── project-init/        # Skill generator skill
│   └── templates/           # Templates for generated skills
├── themes/
│   └── pi-kit-night.json    # Dark theme
└── package.json             # Package manifest
```

## Requirements

- `@earendil-works/pi-coding-agent` (peer)
- `@earendil-works/pi-tui` (peer)
- `typebox` (peer)

## License

See [pi](https://github.com/earendil-works/pi).
