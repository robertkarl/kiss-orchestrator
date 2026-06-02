You are spawning or continuing a subtask from your current session. Your job is to either construct a prompt and spawn a new subtask instance, or write a continuation blurb for an existing one.

## Task Request

$ARGUMENTS

## Procedure

### 1. Classify the request

Determine whether this is:

- **New subtask**: User says "new subtask", "spawn a subtask for...", or there's no existing subtask that fits the work.
- **Continuation**: User says "send this to the X subtask", "tell the [name] subtask to also...", or references a recently-used subtask by name or topic.

If ambiguous, ask.

---

## New Subtask Flow

### 2. Understand the task

Parse what the user wants. Identify:
- What needs to be built or modified
- Which existing files are relevant context
- Whether this is a new script, a modification, or analysis work
- Any session-specific details to pass along (current values, file paths, measurements, error messages)

### 3. List session-known context files

Include only files you've **already referenced in this session**, or that the user explicitly mentioned. The subtask reads project CLAUDE.md automatically and can discover the rest of its context via grep/glob — don't compile a speculative menu.

**Do NOT read files just to write the prompt.** Only read a file if YOU need information from it that isn't already in your session context.

### 4. Construct the prompt

Write a prompt file to `/tmp/subtask_<descriptive_slug>.prompt.md`. Structure:

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

**Requirements, not implementation.** Describe *what* is needed and *why*, not *how* to build it. The subtask reads the context files and figures out the approach — that's its job. Don't specify CLI flag names, function signatures, data formats, or step-by-step implementation plans unless the user explicitly dictated them.

Include session-specific values when relevant (file paths, measurements, error messages) — these are requirements context, not implementation detail. Also include test plans where applicable.

**NEVER include "commit", "push", or sync instructions in the prompt.** The subtask's default behavior is to propose a plan and wait for approval. Don't override that — the user reviews before anything is committed.

### 5. Spawn

```bash
subtask-launch <slug> -f /tmp/subtask_<slug>.prompt.md
```

If the task targets a different repo than the one you're currently in, add `--repo <path>`.

**Avoid nested worktrees — subtasks spawned by subtasks should be siblings.** `subtask-launch` defaults `--repo` to the git toplevel of your CWD. If you're already inside a subtask worktree (path ends in `.worktrees/<name>`), that default would create `<main_repo>/.worktrees/<parent>/.worktrees/<child>` — nested. Pass `--repo` pointing at the *main* repo so the new subtask lands as a sibling under `<main_repo>/.worktrees/` instead. From a subtask worktree root, that's `--repo ../..`.

Tell the user: the subtask is running in a new terminal. It will propose its plan before writing any code, and write a summary to `/tmp/<name>_summary.md` when done.

### 6. After spawning

- When the user says the subtask is done (or you read its summary), review the output
- Subtasks land their own code (rebase + ff-merge into master, push, plus any project-specific sync) — you don't need to sync

---

## Continuation Flow

For sending a follow-up task to an already-running subtask.

### 2c. Pick a slug for the continuation

Choose a short descriptive slug for this follow-up task (e.g. `fix_threshold`, `add_settle_param`). This determines the new summary file name: `/tmp/subtask_<slug>_summary.md`.

### 3c. Write the continuation blurb

Write a blurb that the user will paste into the existing subtask's terminal.

**Do NOT read source files to write the blurb.** You already have session context. The subtask has the code open. Only include problem description and session-specific values (measurements, file paths, error messages).

Format:

```
# Operator Task

Summary file: `/tmp/subtask_<slug>_summary.md`

<problem description — what's wrong or what's needed, with any relevant values/context from the current session>

Before proceeding, recite your subtask_agent Rules.
```

Rules for the blurb:
- **Problem description only** — don't propose solutions. The subtask agent will figure out the approach.
- Include session-specific values (file paths, error messages, etc.) that the subtask needs.
- Keep it concise — 1-2 paragraphs max.

### 4c. Present to the user

Show the blurb and tell the user which subtask terminal to paste it in. Example:

> Paste this into the [name] subtask terminal:

### 5c. Log it (if applicable)

If the parent session is keeping a notebook/log, append a one-liner: what continuation was sent, to which subtask.
