# Newline in OpenAI Codex CLI: It's Ctrl-J, Not Shift+Enter

Shift+Enter doesn't insert a newline in the OpenAI Codex CLI. This isn't a terminal or environment issue — you'll see the same behavior whether you're using iTerm2, the macOS Terminal app, or anything else. It's built into Codex itself.

The shortcut for a new line is **Ctrl-J**.

This catches people off guard because Shift+Enter is the standard multiline shortcut in virtually every chat and code editor (Claude Code, ChatGPT web, VS Code, etc.). The muscle memory is strong and there's currently no way to remap it — Codex doesn't expose a keybinding config.

