# nginx: upstream prematurely closed connection while reading response header from upstream

> nginx sent the request to the backend, but the backend closed the connection before it sent a response. The user sees **502 Bad Gateway**. The backend crashed, was killed, restarted, or does not speak the protocol nginx uses.

Example:

```
2026/10/29 11:20:45 [error] 1234#1234: *4521 upstream prematurely closed connection while reading response header from upstream, client: 203.0.113.5, server: example.com, request: "POST /api/export HTTP/2.0", upstream: "http://127.0.0.1:8080/api/export"
```

## Common causes

- The backend process crashed or was killed during the request (segfault, out-of-memory kill, a worker recycled after a request limit).
- The backend was restarted or reloaded at that moment, for example by a deployment or a service manager.
- A protocol mismatch: `proxy_pass http://` to a port that expects HTTPS, or `fastcgi_pass` to a plain HTTP port.
- The backend closes idle keepalive connections sooner than nginx reuses them.
- The backend rejects the request, for example a header or body that is too large, and drops the connection.

## How to check

How often and for which URLs:

```bash
grep 'prematurely closed' /var/log/nginx/error.log | tail -n 10
```

Did the backend crash or restart? (PHP-FPM example):

```bash
journalctl -u 'php*-fpm' --since '1 hour ago' --no-pager | tail -n 30
```

Was a process killed by the kernel?:

```bash
sudo dmesg -T | grep -iE 'out of memory|killed process|segfault' | tail -n 10
```

Does the backend answer directly? (adjust the port):

```bash
curl -sv -o /dev/null http://127.0.0.1:8080/
```

## Fix

1. Find out why the backend dies: read its own log and the kernel log, and fix crashes or memory exhaustion first.
2. Check the scheme: use `proxy_pass https://...` for a TLS backend, `fastcgi_pass` only for FastCGI sockets or ports.
3. When keepalive is used towards the backend, set the backend idle timeout higher than nginx's `keepalive_timeout` in the `upstream` block.
4. Avoid restarting the backend during traffic: use a graceful reload (`systemctl reload ...`) in deployments.
5. Test and apply: `nginx -t && systemctl reload nginx`.

## FAQ

**How does this differ from 111 Connection refused?**

With 111 nothing listens on the port. Here the connection was accepted and then closed without a response, so the process is running but failing or disconnecting.

**Why does it only happen on some requests?**

Usually these requests trigger the crash or exceed a limit, for example a large export or upload. Compare the failing URLs in the log.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nginx-upstream-prematurely-closed-connection/](https://daniele-michelin.com/en/errors/nginx-upstream-prematurely-closed-connection/)**
