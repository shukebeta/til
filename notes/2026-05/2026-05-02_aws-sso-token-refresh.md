# AWS SSO CLI tokens don't expire as fast as you think

There are three independent expiration clocks when using AWS CLI with IAM Identity Center:

- **SSO web session** (~4h default) — your browser session on the AWS portal
- **SSO access token** (up to 8h) — what the CLI actually uses; obtained via `aws sso login`
- **Role session** (up to 12h) — the `AssumeRole` credential your profile gets

The CLI auto-refreshes the access token as long as the SSO session is still valid. So if your CLI breaks within the 8h window, it's usually the SSO session that expired, not the access token. Re-run `aws sso login` and you're back.

One thing worth knowing: the web console and CLI use separate token chains. Your browser can be logged out while CLI keeps working fine, and vice versa.

All three durations are configurable by your AWS admin in IAM Identity Center → Settings → Session management.
