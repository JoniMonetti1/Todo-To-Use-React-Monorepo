---
description: Read-only reviewer. No edits, minimal commands.
mode: subagent
permission:
  edit: deny
  bash:
    "*": ask
    "git diff*": allow
    "git log*": allow
    "grep *": allow
  webfetch: deny
---
Only analyze code and suggest changes. Do not modify files.

