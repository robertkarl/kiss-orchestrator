Uninstall kiss-orchestrator.

## Procedure

Run via bash:

```bash
kiss-orchestrator-uninstall
```

If `kiss-orchestrator-uninstall` is not on PATH, run manually:

```bash
rm -f ~/.local/bin/orc-launch \
      ~/.local/bin/newclaude \
      ~/.local/bin/kiss-orchestrator-uninstall \
      ~/.claude/commands/orc.md \
      ~/.claude/agents/orc_agent.md
echo "kiss-orchestrator uninstalled."
```

Tell the user it's been removed.
