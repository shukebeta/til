# autossh + SSH Keepalive: The -M 0 Trick

`autossh` has its own connection monitoring that opens an extra port for heartbeats. But this is redundant — SSH already has native keepalive via `ServerAliveInterval`. The extra port can even cause problems if it's not available on the remote side.

The modern approach: disable autossh's built-in monitoring with `-M 0` and let SSH's native heartbeats handle it. When SSH detects a dead connection (after `ServerAliveInterval * ServerAliveCountMax` seconds of no response), autossh automatically reconnects.

Put everything in `~/.ssh/config`:

```
Host gateway
    HostName your-gateway-ip
    User ec2-user
    ServerAliveInterval 30
    ServerAliveCountMax 3
    LocalForward 8080 internal-soap-ec2:8080
```

Then the command reduces to:

```bash
autossh -M 0 -Nf gateway
```

`-M 0` is the only autossh-specific flag you need. Everything else — host, user, keepalive, even the tunnel — lives in SSH config where it belongs. `ServerAliveInterval 30` sends a heartbeat every 30 seconds through the SSH port itself (no extra ports needed), and `ServerAliveCountMax 3` means 90 seconds of silence triggers a disconnect detection.
