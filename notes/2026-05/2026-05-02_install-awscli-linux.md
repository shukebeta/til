# Installing AWS CLI on Debian/MX Linux: use the official binary, not pip

AWS no longer recommends `pip` for global installs. Use the official v2 binary — it bundles its own Python and doesn't conflict with your system.

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

If `unzip` isn't available: `sudo apt update && sudo apt install unzip`.

Verify with `aws --version` — should show `aws-cli/2.x.x`.
