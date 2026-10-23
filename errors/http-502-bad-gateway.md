# HTTP 502 Bad Gateway with nginx and PHP-FPM

> nginx could not get a valid response from the upstream (PHP-FPM, Node.js, Gunicorn, a Docker container) – the backend is down, unreachable or crashed.

Example:

```
2026/10/23 09:14:02 [error] 1234#1234: *5678 connect() to unix:/run/php/php8.3-fpm.sock failed (2: No such file or directory) while connecting to upstream, client: 198.51.100.7, server: cloud.example.com, request: "GET / HTTP/2.0", upstream: "fastcgi://unix:/run/php/php8.3-fpm.sock:"
```

## Common causes

- PHP-FPM is stopped or crashed (e.g. after an update or an OOM kill).
- The socket path no longer matches after a PHP upgrade, e.g. `php8.2-fpm.sock` instead of `php8.3-fpm.sock`.
- No permission on the socket: `listen.owner`/`listen.group` do not match the nginx user.
- The backend listens on a different port or only on IPv6/IPv4.
- The PHP worker dies mid-request (segfault): `upstream prematurely closed connection`.

## How to check

Read the latest errors in the nginx log:

```bash
tail -n 50 /var/log/nginx/error.log
```

Check PHP-FPM status:

```bash
systemctl status php8.3-fpm
```

Does the socket exist and who owns it?:

```bash
ls -l /run/php/
```

PHP-FPM logs, including crashes:

```bash
journalctl -u php8.3-fpm -n 50 --no-pager
```

Is a TCP backend listening on the expected port?:

```bash
ss -ltnp | grep -E ':(9000|3000|8000) '
```

## Fix

1. Restart PHP-FPM or the backend with `systemctl restart php8.3-fpm`, then look for the root cause in the journal.
2. Point `fastcgi_pass` to the correct socket, then `nginx -t && systemctl reload nginx`.
3. Set `listen.owner = www-data` and `listen.group = www-data` in the FPM pool.
4. On OOM kills, size `pm.max_children` to the available RAM.
5. Protect the backend with systemd `Restart=on-failure`.

## FAQ

**What is the difference between 502 and 504?**

502: the backend does not answer at all or answers garbage. 504: the backend is reachable but takes longer than the nginx timeout.

**Why does 502 only happen occasionally?**

Typical for crashing workers or a full listen backlog. Look for `SIGSEGV` or OOM messages in the journal and `Resource temporarily unavailable` in the nginx log.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/http-502-bad-gateway/](https://daniele-michelin.com/en/errors/http-502-bad-gateway/)**
