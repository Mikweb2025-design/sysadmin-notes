# PHP Fatal error: Maximum execution time of 30 seconds exceeded

> A PHP script ran longer than `max_execution_time` (30 seconds by default for web requests) and PHP stopped it.

Example:

```
PHP Fatal error:  Maximum execution time of 30 seconds exceeded in /var/www/html/wp-admin/includes/class-wp-upgrader.php on line 214
```

## Common causes

- A slow database query, a slow external API or a large import or export.
- The task is simply bigger than the default limit: updates, backups, image batches.
- A loop that never ends or waits for something that does not happen.
- The limit was raised for PHP, but nginx or a proxy still cuts the request earlier (then you see 504 instead).
- The command line has no limit (default 0), so the problem only shows in the browser.

## How to check

What limit does PHP-FPM use?:

```bash
grep -rn 'max_execution_time' /etc/php/*/fpm/
```

Check the active value from the web server's point of view:

```bash
php-fpm8.3 -i | grep -i max_execution_time
```

Is the web server timeout shorter?:

```bash
nginx -T 2>/dev/null | grep -E 'fastcgi_read_timeout|proxy_read_timeout'
```

## Fix

1. Find the slow part (query, API call) and speed it up. That is the real fix.
2. Raise the limit where needed: `max_execution_time = 120` in the FPM `php.ini`, or `php_admin_value[max_execution_time] = 120` in the pool, then reload PHP-FPM.
3. Raise `fastcgi_read_timeout` in nginx to match, otherwise users get a 504 first.
4. Run long tasks as a cron job, queue worker or CLI command instead of in a web request.
5. In your own code, call `set_time_limit()` only for the one script that needs it.

## FAQ

**Why do I still get a 504 after raising max_execution_time?**

nginx has its own limit, 60 seconds by default. See [upstream timed out (110)](https://daniele-michelin.com/en/errors/nginx-upstream-timed-out-110/).

**Is a very high limit safe?**

A long limit lets one stuck request block a worker for a long time. Keep it as low as the real task allows.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/php-maximum-execution-time-exceeded/](https://daniele-michelin.com/en/errors/php-maximum-execution-time-exceeded/)**
