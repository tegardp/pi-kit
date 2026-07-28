# pi-permission-system config

`config.json` in this directory is the **global** policy — symlinked to
`~/.pi/agent/extensions/pi-permission-system/config.json` and applied to every
project unless a project overrides it.

## Adding a project-specific override

Project config lives at `<project>/.pi/extensions/pi-permission-system/config.json`
and only loads once the project is trusted. It fully replaces the global file
for that project (not a merge), so copy the parts you still want.

Example: a throwaway/sandbox project where bash friction isn't worth it,
but secrets are still off-limits:

```json
{
  "permission": {
    "*": "allow",
    "path": {
      "*": "allow",
      "*.env": "deny",
      "*.env.*": "deny",
      "**/.ssh/*": "deny",
      "**/.aws/*": "deny"
    },
    "bash": {
      "*": "allow",
      "rm -rf *": "deny",
      "sudo *": "ask"
    },
    "external_directory": "ask"
  }
}
```

Example: a stricter project (production infra, client work) — tighten
`external_directory` and `bash` further:

```json
{
  "permission": {
    "*": "ask",
    "path": {
      "*": "allow",
      "*.env": "deny",
      "*.env.*": "deny",
      "**/.ssh/*": "deny",
      "**/.aws/*": "deny",
      "**/*.pem": "deny"
    },
    "bash": {
      "*": "ask",
      "git status": "allow",
      "git diff*": "allow",
      "git log*": "allow",
      "ls *": "allow",
      "cat *": "allow"
    },
    "external_directory": "deny"
  }
}
```

Example: **safe yolo** — no prompts for anything routine, but the guardrails that actually
matter (secrets, destructive bash) stay hard `deny`, immune to `yoloMode`:

```json
{
  "yoloMode": true,
  "permission": {
    "*": "allow",
    "path": {
      "*": "allow",
      "*.env": "deny",
      "*.env.*": "deny",
      "**/.ssh/*": "deny",
      "**/.aws/*": "deny",
      "**/*.pem": "deny",
      "**/*_rsa": "deny",
      "**/*_ed25519": "deny"
    },
    "bash": {
      "*": "allow",
      "git push --force*": "ask",
      "git push -f*": "ask",
      "git reset --hard*": "ask",
      "git clean -f*": "ask",
      "rm -rf *": "deny",
      "sudo *": "ask",
      "chmod 777 *": "deny",
      "curl * | sh": "deny",
      "curl * | bash": "deny",
      "wget * | sh": "deny"
    },
    "external_directory": "ask"
  }
}
```

`yoloMode` only auto-approves what would otherwise `ask` — every `deny` above still blocks
unconditionally. This is the difference from Claude Code's `--dangerously-skip-permissions`:
there is no config that skips a `deny`, only ones that don't set the `deny` in the first
place (see "total yolo" below).

Example: **total yolo** — the actual equivalent of `--dangerously-skip-permissions`. No
`deny` anywhere, so `yoloMode` has nothing left to auto-approve around. Nothing is
protected, including `.env`, `~/.ssh`, and `rm -rf`:

```json
{
  "yoloMode": true,
  "permission": {
    "*": "allow",
    "path": { "*": "allow" },
    "bash": { "*": "allow" },
    "external_directory": "allow"
  }
}
```

Only use this in a container, VM, or genuinely disposable sandbox — never in a project with
real credentials, an SSH agent, or write access to anything you'd miss.

To scaffold either into a project:

```bash
mkdir -p .pi/extensions/pi-permission-system
# paste the example above into:
$EDITOR .pi/extensions/pi-permission-system/config.json
```
