---
description: Spawn an orc (parallel Claude worker) in its own git worktree and terminal
---

You are the korc (kiss-orchestrator) spawning an orc from your current session. Your job is to construct a prompt, set up a git worktree, and launch a new Claude instance in a new terminal window.

## Task Request

$ARGUMENTS

## Procedure

### 1. Understand the task

Parse what the user wants. Identify:
- What needs to be built or modified
- Which existing files are relevant context
- Whether this is a new script, a modification, or analysis work
- Any session-specific details to pass along (current values, file paths, measurements, error messages)

### 2. List session-known context files

Include only files you've **already referenced in this session**, or that the user explicitly mentioned. The orc reads project CLAUDE.md automatically and can discover the rest of its context via grep/glob.

**Do NOT read files just to write the prompt.** Only read a file if YOU need information from it that isn't already in your session context.

### 3. Construct the prompt

Determine the korc name (the basename of the repo you're in, e.g. `glass-slipper`). Choose a short descriptive slug for the task (e.g. `fix_auth`, `add_logging`). Write a prompt file to `/tmp/orc_<korc_name>-<slug>.prompt.md`. Structure:

```
# Operator Task

<one-line summary>

## Rules

You are an orc. Follow these rules strictly:

1. **Propose before implementing.** Do NOT write code yet. Use AskUserQuestion to interview the user one question at a time. Grill them until you and the user have agreed on 5-7 tenets or axioms for the approach. Only after achieving consensus on these tenets should you present a concrete plan and wait for explicit approval before writing any code.
2. **Adversarial review before building.** After reaching consensus on tenets and drafting a plan, get an adversarial review of the plan before implementing. Use a subagent (or ask the user to get a review from Codex/Claude) to stress-test the plan for gaps, wrong assumptions, over-engineering, and missing edge cases. Incorporate feedback, then proceed.
3. **Everything is local.** NEVER `git push`. The human pushes by hand.
4. **NEVER commit without explicit go-ahead.** After implementing, ask "commit/land?"
5. **Landing sequence (local only):**
   - Commit on your branch
   - `git rebase <base-branch>`
   - `git -C ../.. merge --ff-only orc/<slug>`
   - Stop. Do NOT push.
6. **Summary file.** After landing, write `/tmp/orc_<korc_name>-<slug>_summary.md` (2-4 sentences: what changed, why, any new flags/tools).

## Context

<2-3 sentences of what this task is about and why>

## Read these files first

<bulleted list of context files to read — only files you've already referenced or that the user mentioned>

## Requirements

<what the user asked for — pass through their requirements, not your interpretation of the implementation>
```

**Requirements, not implementation.** Describe *what* is needed and *why*, not *how* to build it. The orc reads the context files and figures out the approach — that's its job. Don't specify CLI flag names, function signatures, data formats, or step-by-step implementation plans unless the user explicitly dictated them.

Include session-specific values when relevant (file paths, measurements, error messages) — these are requirements context, not implementation detail. Also include test plans where applicable.

**NEVER include "commit", "push", or sync instructions in the prompt.** The orc's default behavior is to propose a plan and wait for approval. Don't override that — the user reviews before anything is committed.

### 4. Create worktree and launch orc

Substitute `SLUG` and `PROMPT_FILE` into the following script and run it as a single bash command. Do not split it up or modify the logic.

**Avoid nested worktrees.** If your CWD contains `.worktrees/`, you are inside an orc worktree. In that case, change `REPO_DIR` to the main repo root (two levels up) so the new orc is a sibling, not nested.

```bash
set -euo pipefail
SLUG="<slug>"
PROMPT_FILE="/tmp/orc_<korc_name>-<slug>.prompt.md"

REPO_DIR="$(git rev-parse --show-toplevel)"
WORKTREE="${REPO_DIR}/.worktrees/${SLUG}"
BRANCH="orc/${SLUG}"

# Prune stale worktrees
git worktree prune 2>/dev/null

# Clean up if this slug was used before
if git worktree list --porcelain | grep -q "worktree ${WORKTREE}$"; then
    git worktree remove --force "${WORKTREE}" 2>/dev/null || true
fi
if git show-ref --verify --quiet "refs/heads/${BRANCH}"; then
    git branch -D "${BRANCH}" 2>/dev/null || true
fi

# Create worktree
mkdir -p "${REPO_DIR}/.worktrees"
git worktree add "${WORKTREE}" -b "${BRANCH}" HEAD

# Launch: pass the file path, NOT the file contents.
# Embedding multi-line markdown in a shell command causes escaping disasters.
CMD="cd $(printf '%q' "${WORKTREE}") && env CLAUDE_CODE_ENABLE_TASKS=false claude --agent orc_agent 'Read ${PROMPT_FILE} for your task instructions.'"

osascript \
    -e 'tell application "iTerm2" to create window with default profile' \
    -e "tell application \"iTerm2\" to tell current session of current window to write text \"${CMD}\""

echo "Orc '${SLUG}' launched in new iTerm2 window (branch: ${BRANCH})"
```

Tell the user: the orc is running in a new terminal. It will propose its plan before writing any code, and write a summary to `/tmp/orc_<korc_name>-<slug>_summary.md` when done.

### 5. After spawning

- When the user says the orc is done (or you read its summary), review the output
- Orcs land their own code (rebase + ff-merge into master, plus any project-specific sync) — you don't need to sync
