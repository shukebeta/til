# Short-lived tokens don't belong in the same file as long-lived credentials

You protect `~/.bashrc.secret` with `chmod 444` so nothing accidentally overwrites your API keys and passwords. Then your token-refresh script needs to write a new session cookie somewhere — and hits `Permission denied`.

The fix isn't adding a second writable secrets file. It's **stop persisting short-lived tokens entirely**.

Refactor the refresh script to output `TOKEN HASH` on stdout and let the caller capture it:

```bash
# refresh_token — reads credentials, prints session token to stdout
CREDENTIALS=$(bash ~/bin/refresh_token 2>/dev/null)
TOKEN=$(echo "$CREDENTIALS" | awk '{print $1}')
HASH=$(echo "$CREDENTIALS" | awk '{print $2}')
```

No file writes. No second secrets layer. The token lives in the process environment for the duration of the operation and disappears when it's done.

This works because session tokens are cheap to re-obtain — one HTTP call with stored credentials. The only reason to persist them is avoiding that call, which is almost never worth the complexity of managing a writable token store alongside a read-only credential store.

The rule: if you can regenerate it from long-lived credentials in under a second, don't persist it.
