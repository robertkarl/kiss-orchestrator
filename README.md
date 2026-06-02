# subtask — isolated side-branch agents for Claude Code

A workflow for spawning "subtasks": side branches of work from a primary Claude
Code session. Each subtask runs as a separate Claude instance in its own git
**worktree** (on a `subtask/<slug>` branch) and its own terminal window, so the
primary session keeps working on `master` uninterrupted. The subtask proposes a
plan, implements after approval, lands locally (rebase + ff-merge), and writes a
summary file the primary session reads back.

## Pieces

- `commands/subtask.md` — a slash command (`/subtask`) for the **primary**
  session. It classifies the request (new subtask vs. continuation), writes a
  prompt file, and spawns the subtask via `subtask-launch`. Also produces
  "continuation blurbs" to paste into an already-running subtask.
- `agents/subtask_agent.md` — the **subtask agent** definition: its rules
  (propose-before-implementing, the local-only landing sequence, never push),
  project-convention deference, persistent agent-memory, and tool hygiene.
- `bin/subtask-launch` — creates `.worktrees/<slug>` on branch
  `subtask/<slug>`, optionally warms it (`scripts/setup-worktree.sh` if the repo
  ships one), then launches a new Claude with `--agent subtask_agent`.
- `bin/newclaude` — opens a new terminal running Claude Code (Linux: Alacritty,
  macOS: Ghostty, Windows: Alacritty/Git Bash). `subtask-launch` calls this.

## How it fits together

```
primary session  ──/subtask──►  subtask-launch ──►  newclaude ──►  new terminal
                                 (git worktree)      (spawn)        Claude --agent subtask_agent
```

## Requirements

- `git` (worktrees), the `claude` CLI on PATH, and a supported terminal
  (Alacritty on Linux/Windows, Ghostty on macOS — edit `newclaude` to use yours).
- These are personal scripts; paths/conventions assume a `master` default branch
  and Claude Code's `~/.claude/{commands,agents}` layout. Adapt to taste.

## Install

```sh
install -m755 bin/subtask-launch ~/.local/bin/subtask-launch
install -m755 bin/newclaude      ~/.local/bin/newclaude
install -m644 commands/subtask.md      ~/.claude/commands/subtask.md
install -m644 agents/subtask_agent.md  ~/.claude/agents/subtask_agent.md
```

Then `/subtask spawn a subtask for …` from any Claude session inside a git repo.

## Notes / things to adapt

- **Landing is local-only and never pushes** — by design (see the agent's Git
  workflow rules). The human pushes by hand.
- `subtask-launch --warm` runs `scripts/setup-worktree.sh` if the target repo
  ships one (no-op otherwise) — that's where you'd put `npm install`, etc.
- Subtasks launch with `CLAUDE_CODE_ENABLE_TASKS=false` to silence the TODO
  system-reminder; drop that env if you want the Task tools.
- `newclaude --fork` resumes the current session forked into a new window — not
  used by the subtask flow, but handy on its own.
