# kiss-orchestrator — parallel Claude Code workers via git worktrees

## Terminology

- **korc.** The main Claude task. It's the kiss-orchestrator instance. You can call him Gothmog if you want. Korcs have a name: the name of the repo you're in.
- **orc.** An orc is just a Claude subtask + minor bookkeeping. It is 1) a new terminal window running Claude. 2) a git worktree initialized by the korc. 3) a prompt written by the korc in `/tmp`.

## Overview

A workflow for spawning orcs: side branches of work from a primary Claude Code
session. Each orc runs as a separate Claude instance in its own git **worktree**
(on an `orc/<slug>` branch) and its own terminal window, so the korc keeps
working on `master` uninterrupted. The orc proposes a plan, implements after
approval, lands locally (rebase + ff-merge), and writes a summary file the korc
reads back.

## Pieces

- `commands/orc.md` — a slash command (`/orc`) for the **korc** session.
  Writes a prompt file and spawns the orc via `orc-launch`.
- `agents/orc_agent.md` — the **orc agent** definition: its rules
  (propose-before-implementing, the local-only landing sequence, never push),
  project-convention deference, persistent agent-memory, and tool hygiene.
- `bin/orc-launch` — creates `.worktrees/<slug>` on branch `orc/<slug>`,
  optionally warms it (`scripts/setup-worktree.sh` if the repo ships one),
  then launches a new Claude with `--agent orc_agent`.
- `bin/newclaude` — opens a new terminal running Claude Code (Linux: Alacritty,
  macOS: iTerm2, Windows: Alacritty/Git Bash). `orc-launch` calls this.

## How it fits together

```
korc session  ──/orc──►  orc-launch ──►  newclaude ──►  new terminal
                          (git worktree)   (spawn)        Claude --agent orc_agent
```

## Requirements

- `git` (worktrees), the `claude` CLI on PATH, and a supported terminal
  (Alacritty on Linux/Windows, iTerm2 on macOS — edit `newclaude` to use yours).

## Install

```sh
install -m755 bin/orc-launch ~/.local/bin/orc-launch
install -m755 bin/newclaude  ~/.local/bin/newclaude
install -m644 commands/orc.md        ~/.claude/commands/orc.md
install -m644 agents/orc_agent.md    ~/.claude/agents/orc_agent.md
```

Then `/orc spawn an orc for ...` from any Claude session inside a git repo.

## Uninstall

```sh
kiss-orchestrator-uninstall
```

Or manually remove: `~/.local/bin/{orc-launch,newclaude,kiss-orchestrator-uninstall}`, `~/.claude/commands/orc.md`, `~/.claude/agents/orc_agent.md`.

## Notes / things to adapt

- **Landing is local-only and never pushes** — by design (see the agent's Git
  workflow rules). The human pushes by hand.
- `orc-launch --warm` runs `scripts/setup-worktree.sh` if the target repo
  ships one (no-op otherwise) — that's where you'd put `npm install`, etc.
- Orcs launch with `CLAUDE_CODE_ENABLE_TASKS=false` to silence the TODO
  system-reminder; drop that env if you want the Task tools.
- `newclaude --fork` resumes the current session forked into a new window — not
  used by the orc flow, but handy on its own.
