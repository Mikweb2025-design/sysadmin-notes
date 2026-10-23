# Nextcloud: internal server error with request ID

> Nextcloud hit an error while handling the request. The request ID shown is the key: with it you find exactly the matching entry in nextcloud.log.

Example:

```
Internal Server Error. The server encountered an internal error and was unable to complete your request. Request ID: aB3xYz9QkLmN0pRs
```

## Common causes

- Database unreachable, password changed or tables locked.
- Data directory or config.php not writable for the web server user.
- Incomplete update: code and database schema do not match.
- Missing or incompatible PHP extension after a PHP update.
- A broken app after an app update.

## How to check

Check basic Nextcloud status:

```bash
sudo -u www-data php /var/www/nextcloud/occ status
```

Find the log entry for the request ID:

```bash
grep 'aB3xYz9QkLmN0pRs' /var/www/nextcloud/data/nextcloud.log | jq -r '.message, .exception.Message? // empty'
```

Show the last entries of nextcloud.log in readable form:

```bash
tail -n 20 /var/www/nextcloud/data/nextcloud.log | jq -r '[.time, .app, .message] | @tsv'
```

Check for errors at web server level:

```bash
tail -n 20 /var/log/nginx/error.log
```

## Fix

1. Read the message for the request ID – it usually names the app, file or SQL error.
2. For an incomplete update run `occ upgrade`, then `occ maintenance:repair`.
3. Disable a broken app: `occ app:disable appname`.
4. Fix ownership: `chown -R www-data:www-data /var/www/nextcloud` (check the data directory separately if it lives elsewhere).
5. Test the database connection directly with the values from config.php.

## FAQ

**Where is nextcloud.log if it is not in the data directory?**

The path is in config.php under `logfile`. With `log_type => syslog` or `systemd` you find the entries with `journalctl`.

**What if the log stays empty?**

Then the request fails before Nextcloud, usually in PHP-FPM or nginx. Check their logs and set `loglevel` to 0 temporarily.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-internal-server-error-request-id/](https://daniele-michelin.com/en/errors/nextcloud-internal-server-error-request-id/)**
