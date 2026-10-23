# HTTP 503 Service Unavailable: who is sending it?

> The service is temporarily unavailable. First find out who generates the 503: nginx itself (rate limit, return 503), the application (maintenance mode) or a load balancer in front.

Example:

```
limiting requests, excess: 20.640 by zone "perip", client: 198.51.100.7, server: example.com, request: "GET /api/ HTTP/1.1"
```

## Common causes

- `limit_req` or `limit_conn` kicks in – nginx answers with 503 by default.
- Application in maintenance mode, e.g. Nextcloud with `maintenance => true`.
- Load balancer (HAProxy, cloud LB) has no healthy backend server.
- PHP-FPM or app workers saturated, queue overflowing.
- Service not ready yet after an update or restart.

## How to check

Check headers: Server and Retry-After show who answers:

```bash
curl -sSI https://example.com/ | head -n 5
```

Search error.log for rate limit hits:

```bash
grep -E 'limiting (requests|connections)' /var/log/nginx/error.log | tail -n 5
```

Find limits and fixed 503 responses in the config:

```bash
nginx -T 2>/dev/null | grep -nE 'limit_req|limit_conn|return 503'
```

Show failed services:

```bash
systemctl --failed
```

## Fix

1. For rate limits set `burst` and `nodelay` sensibly and return the right code with `limit_req_status 429;`.
2. End the application's maintenance mode, e.g. `occ maintenance:mode --off`.
3. Check the load balancer health checks: path, port and expected status code.
4. Match worker count to RAM and load, e.g. `pm.max_children` for PHP-FPM.
5. Restart the service and check the journal for why it was not ready.

## FAQ

**What is the difference between 502 and 503?**

502 means the backend answered invalidly or not at all. 503 means the service is temporarily unavailable on purpose or due to overload.

**Does a 503 hurt Google rankings?**

Not short term – 503 is meant for exactly this. If it lasts for days, Google drops pages from the index. Send a Retry-After header during maintenance.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/http-503-service-unavailable-nginx/](https://daniele-michelin.com/en/errors/http-503-service-unavailable-nginx/)**
