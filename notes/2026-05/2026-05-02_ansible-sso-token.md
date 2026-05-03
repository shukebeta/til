# Ansible playbook fails after 1 hour? Your AWS SSO token expired

Ansible loads AWS credentials once at startup. If your playbook runs longer than the SSO role session (typically 1 hour), boto3 has no chance to refresh them — subsequent AWS API calls fail with expired credential errors.

The fix: a `credential_process` profile that boto3 calls each time it needs credentials.

Add to `~/.aws/config`:

```ini
[profile prod]
sso_session = my-sso
sso_account_id = 123456789012
sso_role_name = AdminRole
region = ap-southeast-2

[profile prod_sdk]
region = ap-southeast-2
credential_process = aws configure export-credentials --profile prod --format process
```

Login once, then run Ansible with the wrapper profile:

```bash
aws sso login --profile prod
AWS_PROFILE=prod_sdk ansible-playbook site.yml
```

Each time boto3 needs credentials, it calls `aws configure export-credentials`, which reads the cached SSO token from `prod` and returns fresh role credentials — no browser, no interaction.

Don't put `aws sso login` in `credential_process`. It opens a browser and will silently hang in the background. `export-credentials` is the non-interactive equivalent.
