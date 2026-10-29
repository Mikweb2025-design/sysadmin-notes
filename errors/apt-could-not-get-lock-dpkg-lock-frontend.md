# apt: Could not get lock /var/lib/dpkg/lock-frontend

> Another process is using the package system right now, most often the automatic updater. apt and dpkg allow only one at a time, so yours has to wait.

Example:

```
E: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 2314 (unattended-upgr)
N: Be aware that removing the lock file is not a solution and may break your system.
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?
```

## Common causes

- `unattended-upgrades` installs security updates in the background, often right after boot.
- Another `apt` or `dpkg` command runs in a second terminal or a script.
- A graphical updater (PackageKit, Software Updater) is active.
- A previous run was killed and left a stale lock and an interrupted dpkg state.

## How to check

Which process holds the lock?:

```bash
sudo lsof /var/lib/dpkg/lock-frontend
```

Package tools that are running:

```bash
ps aux | grep -E '[a]pt|[d]pkg|[u]nattended'
```

Is the automatic updater active?:

```bash
systemctl status unattended-upgrades --no-pager
```

## Fix

1. Wait a few minutes. If a process is shown, it is working. Run your command again afterwards.
2. If you must continue now, stop the updater cleanly: `sudo systemctl stop unattended-upgrades`, once it is idle.
3. Only if no process holds the lock, repair the state: `sudo dpkg --configure -a`, then `sudo apt update`.
4. Remove the lock files only as a last resort and only when you are sure nothing uses them: `sudo rm /var/lib/dpkg/lock-frontend /var/lib/dpkg/lock`.
5. In scripts, use `apt-get -o DPkg::Lock::Timeout=120 …` so the command waits instead of failing.

## FAQ

**Is it safe to delete the lock file?**

Not while a process uses it. Deleting a lock under a running dpkg can corrupt the package database. Wait, or kill nothing before you know what runs.

**Why does it happen right after boot?**

The system checks for security updates shortly after start. Give it a few minutes, or wait for `unattended-upgrades` to finish.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/apt-could-not-get-lock-dpkg-lock-frontend/](https://daniele-michelin.com/en/errors/apt-could-not-get-lock-dpkg-lock-frontend/)**
