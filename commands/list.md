---
description: List active orc worktrees
---

List all orc worktrees for the current repo.

## Procedure

Run:

```bash
git worktree list
```

Filter the output to show only worktrees on `orc/` branches. For each one, show:
- The slug (branch name minus the `orc/` prefix)
- The worktree path
- The branch
- The HEAD commit (short hash + subject)

If there are no orc worktrees, say so.
