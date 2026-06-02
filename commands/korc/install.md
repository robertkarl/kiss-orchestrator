Install kiss-orchestrator (orc workflow for Claude Code).

Run the following install commands:

```bash
# Find the plugin root (where this command file lives)
PLUGIN_ROOT="$(dirname "$(dirname "$(dirname "$0")")")"

# If PLUGIN_ROOT doesn't resolve (slash command context), find it via the repo
if [[ ! -d "$PLUGIN_ROOT/bin" ]]; then
    # Try common locations
    for candidate in \
        "$HOME/Code/kiss-orchestrator" \
        "$(git -C "$(dirname "${BASH_SOURCE[0]:-$0}")" rev-parse --show-toplevel 2>/dev/null)" \
    ; do
        if [[ -d "$candidate/bin" ]]; then
            PLUGIN_ROOT="$candidate"
            break
        fi
    done
fi
```

Actually — this is a slash command, not a shell script. Here's what to do:

## Procedure

Run these commands via bash:

```bash
# Determine where the kiss-orchestrator repo is cloned
REPO=""
for candidate in "$HOME/Code/kiss-orchestrator" "$HOME/Code/subtask"; do
    if [[ -d "$candidate/bin" && -f "$candidate/commands/orc.md" ]]; then
        REPO="$candidate"
        break
    fi
done

if [[ -z "$REPO" ]]; then
    echo "ERROR: kiss-orchestrator repo not found. Clone it first:"
    echo "  git clone https://github.com/robertkarl/kiss-orchestrator ~/Code/kiss-orchestrator"
    exit 1
fi

mkdir -p ~/.local/bin ~/.claude/commands ~/.claude/agents

install -m755 "$REPO/bin/orc-launch"                  ~/.local/bin/orc-launch
install -m755 "$REPO/bin/newclaude"                    ~/.local/bin/newclaude
install -m755 "$REPO/bin/kiss-orchestrator-uninstall"  ~/.local/bin/kiss-orchestrator-uninstall
install -m644 "$REPO/commands/orc.md"                  ~/.claude/commands/orc.md
install -m644 "$REPO/agents/orc_agent.md"              ~/.claude/agents/orc_agent.md

echo "kiss-orchestrator installed. Use /orc from any Claude session in a git repo."
echo "To uninstall: kiss-orchestrator-uninstall"
```

Tell the user what was installed and that `/orc` is now available.
