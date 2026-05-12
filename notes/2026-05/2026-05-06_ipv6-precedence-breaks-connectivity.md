# Linux suddenly preferring IPv6 and breaking connectivity? Fix gai.conf

Your Linux box resolves a host to its AAAA (IPv6) record, connects, but the remote isn't actually listening on IPv6. You see `telnet` hang on an IPv6 address. This happens when your system starts preferring IPv6 over IPv4.

One-off fix — force IPv4 for a single command:

```bash
telnet -4 api.z.ai 80
```

Permanent fix — edit `/etc/gai.conf` and uncomment this line:

```
precedence ::ffff:0:0/96  100
```

This tells `getaddrinfo()` to return IPv4-mapped addresses with higher precedence than IPv6. Changes take effect immediately — no restart needed. `gai.conf` is re-read on every `getaddrinfo()` call, so the next DNS lookup picks up the new rule. Existing connections are unaffected.

If `/etc/gai.conf` doesn't exist, just create it with that single line.
