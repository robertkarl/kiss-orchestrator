# kiss-orchestrator. stupid simple subagent orchestrator for Claude
- This was ripped off wholesale from Zack Gomez.

  
## Terminology


- **korc.** Your main Claude thread. Call him Gothmog if you want.
- **orc.** An orc is just a Claude subtask + minor bookkeeping. It is 1) a new terminal window running Claude. 2) a git worktree initialized by the korc. 3) a prompt written by the korc.

## Overview

A workflow for spawning orcs: side branches of work from a primary Claude Code
session. Each orc runs as a separate Claude instance in its own git **worktree**
(on an `orc/<slug>` branch) and its own terminal window, so the korc keeps
working on `master` uninterrupted. The orc proposes a plan, implements after
approval, lands locally (rebase + ff-merge), and writes a summary file the korc
reads back.

Orc workflows are opinionated: no pushing. Must get approval before implementing. The korc needs to handle pushing.

## Install

```sh
claude plugin marketplace add robertkarl/kiss-orchestrator
claude plugin install korc@kiss
```

Then `/orc spawn an orc for ...` from any Claude session inside a git repo.

## Uninstall

```sh
claude plugin uninstall korc
```

## Pieces

- `commands/orc.md` — the `/orc` slash command for the korc session.
  Writes a prompt, creates the worktree, and launches the orc in a new terminal.
- `agents/orc_agent.md` — the orc agent definition: propose-before-implementing,
  local-only landing sequence, never push, project-convention deference.
- `bin/newclaude` — standalone utility for opening a new terminal running Claude
  Code (Linux: Alacritty, macOS: iTerm2, Windows: Alacritty/Git Bash). Optional.

## How it fits together

```
korc session  ──/orc──►  git worktree add  ──►  new terminal
                          write prompt            Claude --agent orc_agent
```

## Requirements

- `git` (worktrees), the `claude` CLI on PATH, and a supported terminal
  (Alacritty on Linux/Windows, iTerm2 on macOS).

## Notes

- **Landing is local-only and never pushes** — by design. The human pushes.
- Orcs launch with `CLAUDE_CODE_ENABLE_TASKS=false` to silence the TODO
  system-reminder; drop that env if you want the Task tools.
- `bin/newclaude` is a standalone utility not required by the plugin but useful
  on its own (`newclaude --fork` forks the current session into a new window).
