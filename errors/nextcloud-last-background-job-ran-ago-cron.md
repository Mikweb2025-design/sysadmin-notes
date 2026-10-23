# Nextcloud: “last background job execution ran … ago”

> Nextcloud handles cleanup, notifications, file versions and trash via background jobs. The warning means these jobs have not run for a while.

Example:

```
Last background job execution ran 3 days ago. Something seems wrong.
```

## Common causes

- Background jobs set to AJAX or Webcron instead of Cron.
- No crontab entry for the web server user, or the entry is in root's crontab.
- cron.php aborts, e.g. because of APCu in the CLI, the wrong PHP version or memory limit.
- systemd timer disabled or not moved along after a migration.
- Container setup without a separate cron container.

## How to check

Show the configured background job mode:

```bash
sudo -u www-data php /var/www/nextcloud/occ config:app:get core backgroundjobs_mode
```

Show the web server user's crontab:

```bash
crontab -u www-data -l
```

Run cron.php once by hand and see errors:

```bash
sudo -u www-data php -f /var/www/nextcloud/cron.php; echo "exit $?"
```

Check existing systemd timers:

```bash
systemctl list-timers | grep -i nextcloud
```

## Fix

1. Set the mode to Cron: `occ background:cron`.
2. Create the crontab for www-data: `*/5 * * * * php -f /var/www/nextcloud/cron.php`.
3. Fix errors from the manual run, e.g. set `apc.enable_cli=1`.
4. Alternatively use a systemd timer that runs every 5 minutes as www-data.
5. After a few minutes check in the admin settings that the time updated.

## FAQ

**Why not AJAX?**

AJAX only runs jobs when someone uses the web UI, and only one per page load. With little usage jobs pile up.

**Can cron.php run as root?**

No. cron.php must run as the owner of the Nextcloud files, otherwise it creates files with wrong permissions and Nextcloud aborts.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-last-background-job-ran-ago-cron/](https://daniele-michelin.com/en/errors/nextcloud-last-background-job-ran-ago-cron/)**
