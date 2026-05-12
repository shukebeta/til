# WSL2 Won't Run on EC2 Windows Server? Use WSL1

Most EC2 Windows Server instances don't expose nested virtualization to the guest OS. WSL2 requires Hyper-V extensions, so it simply won't start. WSL1 works fine — it's a translation layer, not a VM.

Enable the WSL feature and reboot in an Admin PowerShell:

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
Restart-Computer
```

After reboot, pin the default version to 1:

```powershell
wsl --set-default-version 1
```

On Windows Server, `Add-AppxPackage` is often broken or unavailable, so the standard `wsl --install` route fails. Use `wsl --import` with a rootfs tarball instead.

Switch to a non-admin PowerShell — storing the distro under `$HOME` avoids needing admin for every `wsl` operation:

```powershell
mkdir $HOME\WSL\Ubuntu
curl.exe -L -o ubuntu.tar.gz https://cloud-images.ubuntu.com/wsl/jammy/current/ubuntu-jammy-wsl-amd64-wsl.rootfs.tar.gz
wsl --import Ubuntu $HOME\WSL\Ubuntu .\ubuntu.tar.gz --version 1
```

Launch and verify:

```powershell
wsl -d Ubuntu
wsl -l -v
# NAME      STATE    VERSION
# Ubuntu    Running  1
```

What you lose with WSL1: no Docker, no real Linux kernel, no systemd, weaker filesystem compatibility. Fine for CLI tooling, scripting, and build environments. If you need a real kernel, spin up a native Linux EC2 instance instead.
