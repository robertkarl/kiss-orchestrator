You are the korc (kiss-orchestrator) spawning an orc from your current session. Your job is to construct a prompt and spawn a new orc instance.

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

Determine the korc name (the basename of the repo you're in, e.g. `glass-slipper`). Write a prompt file to `/tmp/orc_<korc_name>-<descriptive_slug>.prompt.md`. Structure:

```
# Operator Task

<one-line summary>

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

### 4. Spawn

```bash
orc-launch <slug> -f /tmp/orc_<korc_name>-<slug>.prompt.md
```

If the task targets a different repo than the one you're currently in, add `--repo <path>`.

**Avoid nested worktrees — orcs spawned by orcs should be siblings.** `orc-launch` defaults `--repo` to the git toplevel of your CWD. If you're already inside an orc worktree (path ends in `.worktrees/<name>`), that default would create `<main_repo>/.worktrees/<parent>/.worktrees/<child>` — nested. Pass `--repo` pointing at the *main* repo so the new orc lands as a sibling under `<main_repo>/.worktrees/` instead. From an orc worktree root, that's `--repo ../..`.

Tell the user: the orc is running in a new terminal. It will propose its plan before writing any code, and write a summary to `/tmp/orc_<korc_name>-<slug>_summary.md` when done.

### 5. After spawning

- When the user says the orc is done (or you read its summary), review the output
- Orcs land their own code (rebase + ff-merge into master, plus any project-specific sync) — you don't need to sync
