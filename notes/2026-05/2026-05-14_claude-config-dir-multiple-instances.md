# Run Multiple Claude Code Instances with Separate Configs

Use `CLAUDE_CONFIG_DIR` to point Claude Code at a different config directory — credentials, settings, sessions all go there instead of `~/.claude`.

```bash
mkdir -p ~/.claude-work
CLAUDE_CONFIG_DIR=$HOME/.claude-work claude
```

Set up aliases in your shell config for quick switching:

```bash
alias claude-work='CLAUDE_CONFIG_DIR=$HOME/.claude-work claude'
```

First launch in the new directory triggers the login flow — that's how you get account isolation. Both instances can run simultaneously without interference.

## Share session history between instances

`CLAUDE_CONFIG_DIR` is all-or-nothing — no built-in way to share just sessions. Symlink the `projects/` directory back to the original to get shared session history with separate settings:

```bash
mkdir -p ~/.claude-work
cp ~/.claude/settings.json ~/.claude-work/settings.json
ln -s ~/.claude/projects ~/.claude-work/projects
```

This gives you independent config/credentials but a unified session list. New sessions written by either instance appear in both (bidirectional). No read-only option exists — if you need isolation, don't symlink.
