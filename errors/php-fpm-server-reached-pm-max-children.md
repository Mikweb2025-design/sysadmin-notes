# PHP-FPM: [pool www] server reached pm.max_children setting, consider raising it

> All PHP workers of the pool are busy. New requests queue until one is free, so pages slow down and nginx may return 502 or 504. The cause is either too few workers for the load, or workers that are stuck on something slow.

Example:

```
[29-Oct-2026 10:14:52] WARNING: [pool www] server reached pm.max_children setting (5), consider raising it
```

## Common causes

- Slow scripts keep workers busy for long: a slow database query, a slow external API, a lock.
- A burst of traffic or bots hits uncached PHP pages.
- `pm.max_children` is simply too low for the normal load of this server.
- The value was raised too far in the past and PHP now exhausts RAM, so workers get killed instead (check the OOM page).
- Long-running requests such as exports or imports block workers.

## How to check

Active pool settings (without comments):

```bash
grep -hE '^(pm|pm\.[a-z_]+) *=' /etc/php/*/fpm/pool.d/*.conf
```

How often did the pool hit the limit?:

```bash
journalctl -u 'php*-fpm' --no-pager | grep -c 'max_children'
```

Average memory per PHP-FPM worker:

```bash
ps -eo rss,cmd | awk '/[p]hp-fpm: pool/ {s+=$1; n++} END {if (n) print n " workers, " s/n/1024 " MB average"}'
```

Free memory on the server:

```bash
free -m
```

## Fix

1. Size the pool from memory: `pm.max_children` = RAM you can give to PHP divided by the average worker size from the check above.
2. Set it in the pool file under `/etc/php//fpm/pool.d/www.conf`, together with matching `pm.start_servers`, `pm.min_spare_servers` and `pm.max_spare_servers` for `pm = dynamic`.
3. Find the slow requests: set `request_slowlog_timeout = 5s` and `slowlog = /var/log/php-fpm-slow.log` in the pool, then read the stack traces.
4. Reduce the load: cache pages, block abusive bots, move long jobs out of web requests.
5. Restart the PHP-FPM service of your version, then watch the log for the warning.

## FAQ

**Can I just set a very high pm.max_children?**

No. Each worker uses memory. If the total exceeds RAM, the kernel kills processes or the server swaps, which is worse than queued requests.

**Which pm mode should I use?**

`dynamic` is a good default. `ondemand` saves RAM on idle servers but is slower on first requests, `static` is for busy servers with predictable memory.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/php-fpm-server-reached-pm-max-children/](https://daniele-michelin.com/en/errors/php-fpm-server-reached-pm-max-children/)**
