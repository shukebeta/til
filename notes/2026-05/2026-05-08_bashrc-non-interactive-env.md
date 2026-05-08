# Your .bashrc bails out before agents see PATH — fix it with BASH_ENV

The first non-comment line in most `.bashrc` files is this:

```bash
[ -z "$PS1" ] && return
```

It means: if this shell isn't interactive, do nothing. Innocent-looking, but it silently breaks any agent that shells out via `bash -c '...'` — the child shell gets no PATH additions, no `cargo env`, no API keys you carefully exported in `.bashrc.custom`. Your terminal works fine, your CLI agents don't, and the failure mode is some bare command name "not found" that has nothing to do with the actual problem.

The fix is to split the file by *purpose* (env vs interactive), not by *file* (`.bashrc` vs `.bashrc.custom`), and use `BASH_ENV` as the loader for non-interactive shells.

## Split env from interactive

Pull every `export`, every `PATH=` mutation, every `. ~/.cargo/env` and `. ~/.bashrc.secret` out of `.bashrc.custom` and put them in a new file. Call it `~/.bashrc.env`. It must be idempotent and produce no output (no `echo`, no `which`).

```bash
# ~/.bashrc.env — env-only, safe for both interactive and non-interactive shells.
export FNM_DIR="$HOME/.local/share/fnm"
export PNPM_HOME="$HOME/.local/share/pnpm"

case ":$PATH:" in
  *":$FNM_DIR:"*) ;;
  *) PATH="$FNM_DIR:$PATH" ;;
esac
# ... more PATH cases ...
export PATH

[ -f "$HOME/.cargo/env" ]      && . "$HOME/.cargo/env"
[ -f "$HOME/.bashrc.secret" ]  && . "$HOME/.bashrc.secret"

# This is the lever — child non-interactive bashes will source this file.
export BASH_ENV="$HOME/.bashrc.env"
```

Then source it from `.bashrc` **before** the PS1 guard:

```bash
# ~/.bashrc
[ -f "$HOME/.bashrc.env" ] && . "$HOME/.bashrc.env"
[ -z "$PS1" ] && return
# ... interactive stuff: prompt, aliases, autojump, fnm chpwd hook ...
```

Now interactive shells get env first, then everything else. Non-interactive children of an interactive shell inherit the env via normal process inheritance — they don't even need to re-source anything.

## Why BASH_ENV matters

Bash invoked as `bash -c 'cmd'` is non-interactive non-login. By default it skips `.bashrc` entirely. The one hook bash *does* honor in that mode is the `BASH_ENV` environment variable: if set, bash sources whatever file it points to before running the command.

So once your interactive shell has run `.bashrc.env` and exported `BASH_ENV=$HOME/.bashrc.env`, any descendant `bash -c` will source the same file and get the same PATH and secrets. Verify:

```bash
env -i HOME=$HOME PATH=/usr/bin:/bin BASH_ENV=$HOME/.bashrc.env \
  bash -c 'which fnm; which cargo; echo $SOME_SECRET_VAR'
```

`env -i` strips the environment to nothing — if the three lookups still resolve, your non-interactive path works.

## The systemd edge case

The trick above only works if the agent is descended from your interactive login shell. Anything launched by `systemd --user` (user services, `.desktop` apps, `systemd-cron`) starts from an empty environment with no `BASH_ENV` set. To cover those too, drop a one-line file:

```
# ~/.config/environment.d/bash_env.conf
BASH_ENV=%h/.bashrc.env
```

`%h` is systemd's `$HOME` placeholder. After the next login, `systemctl --user show-environment | grep BASH_ENV` will show it, and every descendant of the user session — including `bash -c` from a desktop launcher — gets it.

## What stays in .bashrc.custom

Aliases, prompt customization, autojump, fnm's `--use-on-cd` hook, anything that prints or relies on a TTY — all interactive-only. Keep them after the PS1 guard. Agents don't want a colored prompt and definitely don't want a chpwd hook that calls a command that may not be on their PATH.

A useful split test: if removing a line from `.bashrc.custom` would *break a script you call from cron*, that line belongs in `.bashrc.env`. If removing it only changes how your terminal feels, it stays.
