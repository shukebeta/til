# AWS SSO login on a headless server? Use `--use-device-code`

AWS CLI v2.22.0+ switched the default SSO login flow to PKCE, which requires a browser on the same machine. If you're SSH'd into a server with no GUI, `aws sso login` just hangs waiting for a browser that doesn't exist.

The fix is two flags:

```bash
aws sso login --profile prod --no-browser --use-device-code
```

It prints a URL and a one-time code. Open the URL on any other device (phone, laptop, tablet), enter the code, authenticate, and you're done. The CLI polls in the background and picks up the token automatically.
