# HTTP 504 Gateway Timeout with nginx

> nginx reached the backend but did not get a response within the configured timeout (60 seconds by default).

Example:

```
2026/10/23 10:02:41 [error] 1234#1234: *9012 upstream timed out (110: Connection timed out) while reading response header from upstream, client: 198.51.100.7, server: cloud.example.com, request: "POST /index.php/apps/files/ajax/upload.php HTTP/2.0"
```

## Common causes

- Long-running requests: large uploads, exports, reports, image processing.
- The PHP-FPM pool is full (`pm.max_children` reached) and new requests wait in line.
- Slow or locked database queries.
- `fastcgi_read_timeout` or `proxy_read_timeout` too low for the use case.
- Backend on another host unreachable through network or firewall (connect runs into the timeout).

## How to check

Find timeouts in the nginx log:

```bash
grep 'upstream timed out' /var/log/nginx/error.log | tail -n 20
```

Check whether PHP-FPM hits its limit:

```bash
grep -i 'max_children' /var/log/php8.3-fpm.log
```

Measure response time precisely:

```bash
curl -s -o /dev/null -w 'connect=%{time_connect} ttfb=%{time_starttransfer} total=%{time_total}\n' https://cloud.example.com/status.php
```

Show running slow database queries:

```bash
mysqladmin -u root -p processlist
```

## Fix

1. Fix the cause first: optimise slow queries, add indexes, move long jobs to cron or a queue.
2. Raise `pm.max_children` according to RAM (RAM for PHP divided by memory per worker).
3. Raise timeouts together: `fastcgi_read_timeout 300s;` in nginx plus `max_execution_time` and `request_terminate_timeout` in PHP-FPM.
4. With reverse proxy chains (load balancer, Docker, CDN), check the timeout on every hop.
5. After changes run `nginx -t && systemctl reload nginx` and `systemctl reload php8.3-fpm`.

## FAQ

**Should I just set the timeout to 3600 seconds?**

No. It hides the problem and ties up workers. Raise timeouts only for specific locations such as upload or export paths.

**Why do I get a 504 while PHP keeps running?**

nginx only drops the connection to the client. The PHP script continues until `max_execution_time` or `request_terminate_timeout` kicks in.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/http-504-gateway-timeout/](https://daniele-michelin.com/en/errors/http-504-gateway-timeout/)**
