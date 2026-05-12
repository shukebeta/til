# whoseport: Find What's Listening on a Port (With Working Directory)

`ss -tlnp` and `lsof -i :PORT` tell you the PID and command name, but for Node.js or Python processes, "node" or "python" alone doesn't tell you *which project* is running. The working directory is what you actually need — and it's sitting right there in `/proc/$pid/cwd`.

Put this in `~/.bashrc`:

```bash
whoseport() {
  if [ -z "$1" ]; then
    sudo ss -tlnp | tail -n +2 | while read -r line; do
      port=$(echo "$line" | grep -oP ':\K[0-9]+(?=\s)')
      pid=$(echo "$line" | grep -oP 'pid=\K[0-9]+' | head -1 | tr -dc '0-9')
      [ -n "$pid" ] && echo "PORT: $port | PID: $pid | CMD: $(ps -p $pid -o comm=) | CWD: $(readlink /proc/$pid/cwd)"
    done
  else
    sudo ss -tlnp | grep ":$1 " | grep -oP 'pid=\K[0-9]+' | head -1 | tr -dc '0-9' | while read pid; do
      echo "PORT: $1 | PID: $pid | CMD: $(ps -p $pid -o comm=) | CWD: $(readlink /proc/$pid/cwd)"
    done
  fi
}
```

Usage:

```
$ whoseport 6173
PORT: 6173 | PID: 1326886 | CMD: node | CWD: /home/davidw/Projects/ccode_viewer/server

$ whoseport          # list all listening ports
PORT: 61217 | PID: 1356 | CMD: tailscaled | CWD: /
PORT: 22     | PID: 1385  | CMD: sshd      | CWD: /
```

A few things that went wrong before arriving at this version:

Don't use an alias for this — `$1` in a single-quoted alias gets swallowed by inner `sh -c` calls, and `local` variables don't survive `xargs` boundaries. A function avoids both problems. The `tr -dc '0-9'` on the PID is not cosmetic — `ss` output can carry trailing whitespace or newlines that break `ps -p` with a "process ID list syntax error".
