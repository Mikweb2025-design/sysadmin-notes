# nginx: [alert] 768 worker_connections are not enough

> One nginx worker has used up all connections it may hold at once. New requests wait or fail. With `proxy_pass` every request needs two connections (client and upstream), so the limit is reached sooner than the visitor count suggests.

Example:

```
2026/10/29 14:02:11 [alert] 1234#1234: *98765 768 worker_connections are not enough while connecting to upstream, client: 203.0.113.5, server: example.com, request: "GET /api/ HTTP/2.0", upstream: "http://127.0.0.1:8080/api/"
```

## Common causes

- `worker_connections` in the `events` block is still the low value from the packaged `nginx.conf`, and traffic has outgrown it.
- A slow upstream keeps connections open for a long time, so they pile up (see also the 504 and 502 pages).
- Long-lived connections such as WebSocket, SSE or long polling occupy a slot for their whole lifetime.
- The limit is higher in nginx than the OS allows: the open-files limit of the nginx process is too small, so the value cannot be used.
- A flood of requests from bots or a misbehaving client.

## How to check

Current worker settings:

```bash
nginx -T 2>/dev/null | grep -nE 'worker_(processes|connections|rlimit_nofile)'
```

How often does the alert appear?:

```bash
grep -c 'worker_connections are not enough' /var/log/nginx/error.log
```

Open-files limit of the running master process:

```bash
grep 'open files' /proc/$(cat /run/nginx.pid)/limits
```

Socket summary (established vs. waiting):

```bash
ss -s
```

## Fix

1. Raise the limit in `/etc/nginx/nginx.conf`: `events { worker_connections 4096; }` and keep `worker_processes auto;`.
2. Allow enough file descriptors: `worker_rlimit_nofile 8192;` in the main context, at least twice `worker_connections` when proxying.
3. If systemd limits the service, add a drop-in with `systemctl edit nginx` and set `LimitNOFILE=` to a matching value.
4. Reduce the load itself: fix the slow upstream, enable `keepalive` to the upstream, and limit abusive clients with `limit_req`.
5. Apply it: `nginx -t && systemctl restart nginx` (a reload does not change the process limit set by systemd).

## FAQ

**What is the maximum number of clients nginx can serve?**

Roughly `worker_processes × worker_connections`, divided by two when nginx proxies every request, because each proxied request uses a client and an upstream connection.

**Is a higher value always better?**

No. Each connection needs memory and a file descriptor. Raise it moderately, make sure the file descriptor limit matches, and fix the cause if connections pile up because of a slow backend.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nginx-worker-connections-are-not-enough/](https://daniele-michelin.com/en/errors/nginx-worker-connections-are-not-enough/)**
