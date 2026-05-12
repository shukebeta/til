# Stop opening admin cmd just to mklink — git-bash can do real symlinks

If you've been alt-tabbing to an admin cmd window to run `mklink` every time you wanted a symlink on Windows, there's a much cleaner way nobody seems to mention. Two settings, set them once, forget about it.

The reason `ln -s` in git-bash silently turns into `cp` by default is two unrelated barriers stacked together:

1. Windows non-admins can't create symlinks unless **Developer Mode** is on.
2. MSYS (git-bash's runtime) doesn't even *try* to create native symlinks without the **`MSYS`** env var set.

Fix the first once with admin, fix the second in your shell profile.

```powershell
# Run once in admin PowerShell, or use Settings → Privacy & Security → For developers → Developer Mode
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\AppModelUnlock" `
  -Name "AllowDevelopmentWithoutDevLicense" -Value 1 -Type DWord
```

```bash
# In ~/.bashrc
export MSYS=winsymlinks:nativestrict
```

`nativestrict` makes failures loud — `ln -s` errors out instead of silently copying when symlinks aren't supported. Avoid `winsymlinks:native` (the lenient sibling) — its silent fallback is exactly the trap you're trying to escape. Avoid `winsymlinks:lnk` entirely; .lnk shortcuts aren't symlinks anything else recognizes.

Open a new git-bash window and verify:

```bash
ln -s ~/.bashrc /tmp/test-link
ls -la /tmp/test-link
```

If the line starts with `l`, it's a real symlink that git, Linux tools, GNU stow, and your dotfiles `install.sh` all treat consistently across platforms. If it starts with `-`, either the env var didn't reach the new shell or Developer Mode didn't actually toggle.

Bonus: Developer Mode also unlocks Windows 11's native `sudo` command, so you can stop alt-tabbing to admin terminals for quick one-offs entirely.
