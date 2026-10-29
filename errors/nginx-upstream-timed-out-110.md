# nginx: upstream timed out (110: Connection timed out)

> The backend (PHP-FPM, Node, Gunicorn) did not answer within nginx's time limit, so nginx gives up and returns 504 Gateway Timeout.

Example:

```
2026/10/29 09:14:02 [error] 1234#1234: *5678 upstream timed out (110: Connection timed out) while reading response header from upstream, client: 203.0.113.5, server: example.com, request: "POST /api/export HTTP/1.1", upstream: "fastcgi://unix:/run/php/php8.3-fpm.sock"
```

## Common causes

- The request is simply slow: a long database query, an export, a report or a call to a slow external API.
- All PHP-FPM workers are busy (`pm.max_children` reached) and new requests wait in the queue.
- The default `fastcgi_read_timeout` or `proxy_read_timeout` of 60 seconds is shorter than the task needs.
- The backend hangs, for example in a deadlock or while waiting for a locked file.
- For “while connecting to upstream”: a firewall or network problem between nginx and the backend host.

## How to check

Which part timed out? Read the end of the log line:

```bash
grep -i 'timed out' /var/log/nginx/error.log | tail -n 20
```

Are all PHP-FPM workers busy?:

```bash
grep -i 'max_children' /var/log/php8.3-fpm.log | tail -n 5
```

How long does the backend need for the slow URL?:

```bash
curl -s -o /dev/null -w '%{time_total}s\n' https://example.com/api/export
```

Is the backend reachable at all?:

```bash
systemctl status php8.3-fpm
```

## Fix

1. Find out why the request is slow (database, external API) and fix that first. A longer timeout only hides the problem.
2. If the task really needs more time, raise the limit for that location only: `fastcgi_read_timeout 300;` or `proxy_read_timeout 300;`.
3. Raise `pm.max_children` in the PHP-FPM pool if the workers are the bottleneck and the RAM allows it.
4. Move long jobs to the background (cron, queue) and let the page return at once.
5. For “connecting to upstream”: check the firewall and the address of the backend.

## FAQ

**I raised the timeout in nginx but users still see an error. Why?**

Other layers have their own limits: PHP's `max_execution_time`, a load balancer, or a CDN. Cloudflare, for instance, closes the connection after about 100 seconds.

**Is 504 the same as 502?**

No. 502 means the backend gave an invalid answer or was not reachable. 504 means it did not answer in time.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nginx-upstream-timed-out-110/](https://daniele-michelin.com/en/errors/nginx-upstream-timed-out-110/)**
