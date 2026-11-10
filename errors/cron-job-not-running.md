# Cron job not running – even though the entry exists

> Cron starts with a minimal environment. Scripts that work in your shell fail there because of `PATH`, relative paths or missing variables.

Example:

```
CRON[2418]: (root) CMD (/usr/local/bin/job.sh)
/usr/local/bin/job.sh: 3: mysqldump: not found
```

## Common causes

- Cron has a different `PATH` than your shell and does not read `.bashrc`.
- The script uses relative paths, so it runs in the wrong directory.
- The execute bit is missing or the shebang points to a missing interpreter.
- The entry is in the wrong user's crontab, or in `/etc/cron.d` without the user column.
- Cron itself is not running, or a syntax error prevents the file from loading.

## How to check

Cron service and recent runs:

```bash
systemctl status cron --no-pager | head -n 5; journalctl -u cron --since '1 hour ago' --no-pager | tail -n 20
```

Entries of the current user:

```bash
crontab -l
```

Permissions and shebang:

```bash
ls -l /usr/local/bin/job.sh; head -n 1 /usr/local/bin/job.sh
```

Test with an empty environment like cron:

```bash
env -i /bin/sh -c '/usr/local/bin/job.sh'
```

## Fix

1. Use absolute paths in the script and set `PATH` at the top: `PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`.
2. Redirect output to a log file: `>> /var/log/job.log 2>&1`. Otherwise you never see the errors.
3. Check the shebang (`#!/bin/bash`) and set the execute bit with `chmod +x`.
4. In `/etc/cron.d` do not forget the user column, otherwise the line is ignored.
5. Test with `env -i` so you get the same environment as cron.

## FAQ

**Why does it work manually but not from cron?**

Because cron starts without your shell environment: different PATH, no home directory, no aliases.

**Where does the output go?**

Cron mails it to the local user. Without a mail server it disappears, so always log to a file.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/cron-job-not-running/](https://daniele-michelin.com/en/errors/cron-job-not-running/)**
