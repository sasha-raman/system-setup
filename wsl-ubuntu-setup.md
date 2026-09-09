# WSL 2 + Ubuntu Setup on Windows 11

Step-by-step guide for installing and configuring Ubuntu on WSL 2 for development.

## 1. Install WSL 2 and Ubuntu

Open PowerShell as Administrator and run:

```powershell
wsl --install
```

This enables WSL 2, installs the WSL kernel, and installs Ubuntu as the default distribution.

Restart the computer when prompted.

After reboot, the Ubuntu terminal opens automatically. Create a username and password. This user will have `sudo` privileges.

## 2. Initial Package Update

Inside the Ubuntu terminal:

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt install -y build-essential curl wget git unzip
```

## 3. Enable systemd

Create or edit `/etc/wsl.conf` inside Ubuntu:

```ini
[boot]
systemd=true

[user]
default=<your-username>
```

Then restart WSL from PowerShell:

```powershell
wsl --shutdown
```

Reopen Ubuntu.

## 4. Optimize WSL 2 Resources

Create or edit `%USERPROFILE%\.wslconfig` on Windows (e.g., `C:\Users\<you>\.wslconfig`):

```ini
[wsl2]
swap=2GB
networkingMode=mirrored
firewall=true
dnsTunneling=true
autoProxy=true
```

These settings enable mirrored networking and swap. `localhostForwarding` is omitted because it has no effect in mirrored networking mode and will trigger a WSL warning. `memory` and `processors` can be added later if resource tuning is needed, for example `memory=8GB` and `processors=4`. Restart WSL to apply:

```powershell
wsl --shutdown
```

`networkingMode=mirrored` makes the Windows and Ubuntu network stacks share the same IP space, so Ubuntu can access Windows services on `localhost` with fewer firewall or port issues.

## 5. Fix Directory Permissions

### Windows drives (`/mnt/c`, etc.)

Edit `/etc/wsl.conf` inside Ubuntu and add the `[automount]` section:

```ini
[automount]
enabled = true
mountFsTab = true
options = "metadata,umask=022"

[boot]
systemd=true

[user]
default=<your-username>
```

`metadata` allows `chmod` and `chown` to work on Windows files. `umask=022` gives files `644` and directories `755` by default, instead of `777`.

Restart WSL:

```powershell
wsl --shutdown
```

### Ubuntu home directory

Ensure the home directory is not world-accessible:

```bash
chmod 700 ~
```

Project directories can use standard Unix permissions, for example:

```bash
mkdir -p ~/projects
chmod 755 ~/projects
```

## 6. Store Projects on the Linux Filesystem

Keep source code and project files inside the Ubuntu filesystem, for example under `~/projects`. Avoid placing active projects under `/mnt/c` because cross-OS filesystem access is slow and file permissions are synthetic.

## 7. VS Code WSL Integration

On Windows, install these VS Code extensions:

- Remote - WSL
- Remote Development (optional pack)

Open a WSL project by running in Ubuntu:

```bash
code ~/projects
```

Or use the VS Code command palette: `Remote-WSL: New Window Using Distro...`

## 8. Docker Desktop Setup

Install Docker Desktop on Windows and configure it:

1. Settings → General → Use the WSL 2 based engine: **enabled**
2. Settings → Resources → WSL Integration → **Enable integration with my default WSL distro** (or select Ubuntu)

This makes the `docker` and `docker compose` CLIs available inside Ubuntu without installing a Docker daemon inside WSL.

Do not install `docker.io` or `docker-ce` inside Ubuntu when using Docker Desktop; the two daemons will conflict.

## 9. Root Access

Do not set a root password. Use `sudo` instead:

```bash
sudo -i
sudo su -
```

From Windows, open a root shell directly if needed:

```powershell
wsl -u root
```

## 10. Verify the Setup

Inside Ubuntu:

```bash
# WSL kernel info
uname -a

# systemd is running
systemctl is-system-running

# Docker integration works
docker ps

# Filesystem permissions on /mnt/c
ls -la /mnt/c
```

If `/mnt/c` still shows `777`, double-check the `[automount]` `options` line in `/etc/wsl.conf` and run `wsl --shutdown`.

From PowerShell, verify the installed distributions and WSL versions:

```powershell
wsl -l -v
```

## Notes for Later Additions

This guide covers the base WSL 2 / Ubuntu / Docker / VS Code environment. Additional software (languages, databases, cloud CLIs, etc.) will be documented in follow-up files or sections.
