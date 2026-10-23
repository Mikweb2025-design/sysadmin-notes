# nginx: connect() failed (111: connection refused) to upstream

> nginx reaches the backend host, but nothing listens on the target port. The OS refuses the connection immediately and the user sees 502 Bad Gateway.

Example:

```
connect() failed (111: Connection refused) while connecting to upstream, client: 198.51.100.7, server: app.example.com, upstream: "http://127.0.0.1:3000/"
```

## Common causes

- The backend service crashed, was stopped or is restarting.
- The backend listens on a different port or a different address only.
- `proxy_pass` uses `localhost`, which resolves to ::1, but the app only listens on 127.0.0.1.
- Docker container stopped or port not published.
- After a deploy the config still points to an old port.

## How to check

Check whether anything listens on the port:

```bash
ss -ltnp | grep -E ':3000\b'
```

Call the backend directly without nginx:

```bash
curl -sv http://127.0.0.1:3000/ -o /dev/null
```

Show upstream targets in the nginx config:

```bash
nginx -T 2>/dev/null | grep -nE 'proxy_pass|fastcgi_pass|server .*:[0-9]+'
```

Show status and recent log lines of the backend service:

```bash
systemctl status app.service --no-pager -l
```

## Fix

1. Start the backend and check the journal for why it stopped.
2. In `proxy_pass` use the IP the app really listens on, e.g. `http://127.0.0.1:3000` instead of `localhost`.
3. Align the port in the app config and nginx, then `nginx -t && systemctl reload nginx`.
4. With Docker publish the port with `-p 127.0.0.1:3000:3000` or put nginx on the same network.
5. Protect the service with `Restart=on-failure` so a crash does not cause 502s for hours.

## FAQ

**How is this different from “upstream timed out (110)”?**

111 means nothing is listening and the connection is refused at once. 110 means the backend listens but does not answer in time – that gives a 504.

**Why does it only happen right after a deploy?**

The service needs a few seconds to start, and connections are refused meanwhile. Fix: a health check before switching, or several instances.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nginx-connect-failed-111-connection-refused-upstream/](https://daniele-michelin.com/en/errors/nginx-connect-failed-111-connection-refused-upstream/)**
