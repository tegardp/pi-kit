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

To scaffold either into a project:

```bash
mkdir -p .pi/extensions/pi-permission-system
# paste the example above into:
$EDITOR .pi/extensions/pi-permission-system/config.json
```
