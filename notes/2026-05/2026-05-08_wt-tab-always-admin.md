# WT keeps opening as Administrator? Check settings.json before the registry

Your Windows Terminal default tab is opening with admin privileges and you don't remember asking for it. The intuitive guess is the AppCompat Layers registry — that's where ticking "Run as administrator" on a shortcut's compatibility tab gets stored, and it persists across reinstalls. Worth checking, but it's the second place to look for WT.

The first place is `settings.json`:

```json
{
    "commandline": "C:\\Program Files\\Git\\bin\\bash.exe",
    "elevate": true,
    "guid": "{17c3a2bf-...}",
    "name": "Git Bash"
}
```

WT 1.18+ added `elevate` as a per-profile flag. When it's `true` and that profile is also pointed to by `defaultProfile`, every new window silently opens elevated — no UAC prompt at the tab level because the elevation happened at WT launch. Delete the line, restart WT, done.

If you still want an admin tab on demand, add a second profile with a fresh GUID and a different name (e.g. "Git Bash (Admin)") that keeps `elevate: true`. You get a dropdown choice and a real UAC prompt when you actually need it, instead of unconditional elevation on every launch.

To rule out the registry side as well:

```powershell
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Layers"
Get-ItemProperty -Path "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Layers"
```

Each property name is an exe path; a value containing `RUNASADMIN` means that program is forced to elevate every launch. To clear one, prefer the GUI — right-click the exe → Properties → Compatibility → uncheck "Run this program as administrator" — over editing the registry by hand.

Two mechanisms, same symptom, different layers. For Windows Terminal specifically, the per-profile setting wins; check it first.
