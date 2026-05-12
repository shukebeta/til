# WezTerm stealing Alt+Enter? Here's the fix

WezTerm binds `Alt+Enter` to toggle fullscreen by default. That hijacks `Alt+Enter` newline in Claude Code, GitHub Copilot, and other terminal-based tools.

Drop this in `~/.wezterm.lua` to disable the default and remap fullscreen to `Ctrl+Shift+F11`:

```lua
local wezterm = require 'wezterm'
local config = wezterm.config_builder()

config.keys = {
  {
    key = 'Enter',
    mods = 'ALT',
    action = wezterm.action.DisableDefaultAssignment,
  },
  {
    key = 'F11',
    mods = 'CTRL|SHIFT',
    action = wezterm.action.ToggleFullScreen,
  },
}

return config
```

Config takes effect immediately on save — no restart needed.
