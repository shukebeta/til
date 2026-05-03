# SSH in a script? Use `-o BatchMode=yes` or it'll hang

When SSH runs in a script and the key isn't set up, it prompts for a password — and your script hangs forever. `BatchMode=yes` disables all interactive prompts (password, passphrase, host key confirmation) and fails immediately instead.

```bash
ssh -o BatchMode=yes user@host "echo ok"
```

Returns exit code 0 on success, non-zero immediately on failure. Perfect for connectivity checks in CI/CD or Ansible pre-tasks.
