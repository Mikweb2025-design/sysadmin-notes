# HTTP 429 Too Many Requests

> The server or an intermediate layer says that this client sent too many requests in a given time. It is a rate limit working as designed. The task is to find which layer applies it and whether the limit or the client is wrong.

Example:

```
HTTP/2 429
retry-after: 30

203.0.113.5 - - [29/Oct/2026:10:22:41 +0000] "GET /api/items HTTP/2.0" 429 169 "-" "python-requests"
```

## Common causes

- A client or script sends requests too fast, without back-off or retries, and exceeds a quota.
- A rate limit in nginx (`limit_req` with `limit_req_status 429`), fail2ban, a WAF or the CDN is stricter than the real traffic.
- Many users share one IP (office, mobile network, proxy) and are counted as one client, or the real visitor IP is not restored behind a proxy.
- Bots, crawlers or a vulnerability scan hammer the site.
- A remote API (the 429 comes from a third party) has used up its quota.

## How to check

Who gets 429 in the access log (status is field 9 in the default format)?:

```bash
awk '$9 == 429 {print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head
```

Which limits are configured?:

```bash
nginx -T 2>/dev/null | grep -nE 'limit_req|limit_conn|limit_req_status'
```

What does the response say about the limit?:

```bash
curl -sI https://example.com/ | grep -iE '^(HTTP|retry-after|x-ratelimit)'
```

Is the client IP the real visitor IP in the log?:

```bash
nginx -T 2>/dev/null | grep -nE 'real_ip|set_real_ip_from'
```

## Fix

1. Identify the layer first: the access log (own server), the response headers (CDN or API) and the fail2ban status tell you who answers.
2. For your own limit, adjust it: for example `limit_req zone=perip burst=20 nodelay;` with a rate that matches normal use.
3. Restore the real client IP behind a proxy (`real_ip_header`, `set_real_ip_from`) so users are not counted together.
4. Let known good clients skip the limit with `geo` and `map` or an allow list, and block abusive bots separately.
5. As a client: honor `Retry-After`, use exponential back-off, cache responses and batch requests.

## FAQ

**Does nginx limit_req return 429 by default?**

No, the default status is 503. Set `limit_req_status 429;` if you want the standard code.

**Should I retry immediately?**

No. Wait at least for the time in `Retry-After` if present, and increase the delay on each retry, otherwise you stay limited.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/http-429-too-many-requests/](https://daniele-michelin.com/en/errors/http-429-too-many-requests/)**
