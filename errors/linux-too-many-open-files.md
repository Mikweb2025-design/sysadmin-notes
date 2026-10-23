# Linux: too many open files (EMFILE, error 24)

> A process hit its file descriptor limit. That includes files but also sockets and pipes – on web servers mostly open connections.

Example:

```
accept4() failed (24: Too many open files)
```

## Common causes

- Per-process limit too low, often 1024 by default.
- Limit raised in `/etc/security/limits.conf` – which does not apply to systemd services.
- File descriptor leak: the application does not close files or connections.
- Many concurrent connections, e.g. WebSockets or slow upstreams.
- In nginx: `worker_connections` higher than the workers' limit.

## How to check

Show the actual limit of the running process:

```bash
cat /proc/$(pgrep -o nginx)/limits | grep 'open files'
```

Count currently open descriptors:

```bash
ls /proc/$(pgrep -o nginx)/fd | wc -l
```

Show the limit from the systemd unit:

```bash
systemctl show nginx.service -p LimitNOFILE
```

Find processes with the most open files:

```bash
lsof -n 2>/dev/null | awk '{print $1}' | sort | uniq -c | sort -rn | head
```

## Fix

1. Set the limit for the service: `systemctl edit nginx` with `[Service]` `LimitNOFILE=65535`, then restart.
2. For nginx also set `worker_rlimit_nofile 65535;`.
3. If the number of open descriptors keeps rising, look for a leak instead of only raising the limit.
4. Adjust `limits.conf` for login shells, always systemd for services.
5. Check the system-wide limit: `cat /proc/sys/fs/file-nr`.

## FAQ

**Why does ulimit -n show a high value but the service still fails?**

`ulimit` shows your shell's limit. Services inherit limits from systemd. What counts is /proc/PID/limits of the running process.

**How high can the limit be?**

65535 is generous for most services. Higher is possible but hides leaks. Watch the number of open descriptors over time.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/linux-too-many-open-files/](https://daniele-michelin.com/en/errors/linux-too-many-open-files/)**
