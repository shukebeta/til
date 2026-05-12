# Ctrl+J is the newline shortcut in Codex CLI's TUI

If you're using Codex CLI and want to insert a newline in the prompt box without submitting, `Shift+Enter` probably won't work. Use `Ctrl+J` instead.

`Ctrl+J` is the ASCII control character for line feed (`\n`). Many terminal TUI libraries interpret it as "insert newline" rather than "submit," which is exactly what Codex CLI's input component does.

`Shift+Enter` *looks* like the right key (it works in browser-based chat UIs), but whether it inserts a newline or submits depends entirely on how the TUI library handles raw key events — and Codex CLI's library doesn't treat it as a newline.

So: `Ctrl+J` to insert a newline, `Enter` to submit.
