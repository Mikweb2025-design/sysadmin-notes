# dpkg was interrupted, you must manually run 'dpkg --configure -a' to correct the problem

> A previous install or upgrade did not finish. dpkg left packages half unpacked or half configured and refuses new work until they are completed. Usually one command fixes it.

Example:

```
$ sudo apt upgrade
E: dpkg was interrupted, you must manually run 'sudo dpkg --configure -a' to correct the problem.
```

## Common causes

- The previous apt or dpkg run was cut off: SSH session lost, reboot, power loss or a killed process.
- A package script (postinst) failed, so the package stayed in a half-configured state.
- The disk or `/var` ran full during the installation.
- Unattended upgrades were interrupted by a shutdown.

## How to check

Is apt or dpkg still running? If yes, wait for it.:

```bash
ps -eo pid,etime,cmd | grep -E '[a]pt|[d]pkg'
```

Which packages are in a broken state?:

```bash
sudo dpkg --audit
```

Is there free space?:

```bash
df -h / /var /boot
```

What happened in the last run?:

```bash
tail -n 30 /var/log/dpkg.log
```

## Fix

1. Make sure no apt or dpkg process is running. Never delete the lock files while one is.
2. Finish the interrupted work: `sudo dpkg --configure -a`.
3. Then repair dependencies: `sudo apt --fix-broken install`.
4. If one package keeps failing, read its error. Free disk space first if the disk was full, then repeat the two commands.
5. Last resort for one package that cannot be configured: remove it with `sudo dpkg --remove --force-remove-reinstreq PACKAGE` and install it again. Be careful with system packages.

## FAQ

**Can I just delete the lock file?**

No. The message is not about a lock. Run `dpkg --configure -a`. Deleting locks while a process works can corrupt the package database.

**Is it safe to run dpkg --configure -a on a production server?**

Yes, it only finishes what was started. Check free space first, and run it in a screen or tmux session so a lost connection does not interrupt it again.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/dpkg-was-interrupted-configure-a/](https://daniele-michelin.com/en/errors/dpkg-was-interrupted-configure-a/)**
