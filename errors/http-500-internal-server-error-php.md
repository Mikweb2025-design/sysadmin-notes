# HTTP 500 Internal Server Error: find the cause in the logs

> The application or web server failed while handling the request. The status code says nothing about why – the reason is in the nginx error.log, the PHP-FPM log or the application log.

Example:

```
PHP Fatal error:  Allowed memory size of 134217728 bytes exhausted (tried to allocate 20480 bytes) in /var/www/site/vendor/autoload_real.php on line 41
```

## Common causes

- PHP fatal error, e.g. “Allowed memory size … exhausted” or an undefined function after an update.
- Missing PHP extension after switching PHP versions.
- The application cannot write to cache, log or upload directories.
- Rewrite loop in nginx (“rewrite or internal redirection cycle”).
- Database unreachable or credentials changed.

## How to check

Show recent errors in the nginx error.log:

```bash
tail -n 50 /var/log/nginx/error.log
```

Show PHP-FPM messages from the last minutes:

```bash
journalctl -u 'php*-fpm' --since '15 min ago' --no-pager | tail -n 30
```

Search the PHP-FPM log for fatal errors:

```bash
grep -i 'fatal' /var/log/php*-fpm.log | tail -n 10
```

Fetch the status code reproducibly:

```bash
curl -sS -o /dev/null -w '%{http_code}\n' https://example.com/
```

## Fix

1. Read the first error at the time of the request – it names file and line.
2. For memory errors raise `memory_limit` in the pool or php.ini deliberately, not blanket -1.
3. Install missing modules (e.g. `php8.3-intl`) and restart PHP-FPM.
4. Give the PHP-FPM user write access to cache and log directories.
5. After a failed update roll back to the last working version and analyse calmly.

## FAQ

**Should I turn on display_errors?**

Not in production. Errors in the browser leak paths and credentials. Keep `log_errors = On` and read the log.

**Why is there nothing in the nginx log?**

Then PHP handled the error itself and only returned 500. Check the application log or the PHP-FPM log with `catch_workers_output = yes`.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/http-500-internal-server-error-php/](https://daniele-michelin.com/en/errors/http-500-internal-server-error-php/)**
