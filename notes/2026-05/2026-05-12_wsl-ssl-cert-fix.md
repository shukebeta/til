# WSL SSL Certificate Errors on Corporate Networks

If `curl` throws `SSL certificate problem: unable to get local issuer certificate` every time in WSL, it's usually a stale CA bundle — especially common on corporate networks running SSL inspection (Zscaler, etc.).

First, refresh the bundle:

```bash
sudo apt-get update && sudo apt-get install -y ca-certificates
sudo update-ca-certificates
```

That fixes most cases. If not, try a full reinstall:

```bash
sudo apt-get install --reinstall ca-certificates
sudo update-ca-certificates --fresh
```

On corporate networks, you likely need your company's root CA certs. You probably already have them somewhere in your Windows filesystem (`.crt` or `.cer` files). Copy them all into the trusted store:

```bash
sudo cp /c/Certificates/*.crt /usr/local/share/ca-certificates/
sudo cp /c/Certificates/*.cer /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

Skip `.pfx` files — those contain private keys and are a different format.

If you don't have the certs handy, export them from Windows:

```powershell
# PowerShell — list all root certs, look for your company name
Get-ChildItem Cert:\LocalMachine\Root | Select-Object Subject, Issuer | Sort-Object Subject
```

Or use `certmgr.msc` (Win+R → certmgr.msc) → Trusted Root Certification Authorities → find your company's cert → right-click → Export → Base-64 encoded X.509.

Test with `curl https://google.com` after each step.
