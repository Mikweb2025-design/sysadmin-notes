# systemd: job for X.service failed (control process exited)

> systemd tried to start or reload the service and the start command (ExecStart or ExecStartPre) exited with an error. The actual cause is in the service's journal.

Example:

```
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
```

## Common causes

- Syntax error in the service configuration, e.g. after editing nginx or Postfix.
- Port already in use (“Address already in use”).
- Missing file: certificate, key, socket directory or include.
- Missing permissions, or SELinux/AppArmor blocks access.
- Broken unit or override, e.g. a wrong path in `ExecStart`.

## How to check

Show status with the last log lines:

```bash
systemctl status nginx.service --no-pager -l
```

Show detailed journal entries for the service:

```bash
journalctl -xeu nginx.service --no-pager | tail -n 40
```

Test the config (nginx example; Apache `apachectl configtest`, Postfix `postfix check`):

```bash
nginx -t
```

Show the unit including all overrides:

```bash
systemctl cat nginx.service
```

## Fix

1. Read the first real error in the journal, not just the systemd summary.
2. Fix config errors and always run the service's config test before restarting.
3. If the port is busy, find the process with `ss -ltnp` and stop or reconfigure it.
4. After editing units run `systemctl daemon-reload`.
5. Reset the failed state and start again: `systemctl reset-failed nginx && systemctl start nginx`.

## FAQ

**What does “control process” mean?**

It is the process systemd runs to start, stop or reload the service – ExecStart, ExecStartPre or ExecReload. Its exit code was not 0.

**Why does systemctl status not show the cause?**

Status only shows the last lines. `journalctl -xeu name.service` shows the full history, and the error is often a few lines further up.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/systemd-job-failed-control-process-exited-with-error-code/](https://daniele-michelin.com/en/errors/systemd-job-failed-control-process-exited-with-error-code/)**
