# 400 Bad Request: request header or cookie too large

> A single request header or the cookie line is larger than the buffer nginx reserves for it. The default is 4 buffers of 8 KB – large SSO cookies or JWTs exceed that quickly.

Example:

```
400 Bad Request – Request Header Or Cookie Too Large
```

## Common causes

- Many cookies on the same domain, e.g. analytics, A/B tests and several apps on subdomains.
- SSO/OIDC session cookies or bearer tokens with many group claims.
- A redirect loop that sets another cookie on every pass.
- `large_client_header_buffers` too small for the application.
- The backend (Node, Tomcat, Gunicorn) also has its own header limits.

## How to check

Show current buffer sizes in the config:

```bash
nginx -T 2>/dev/null | grep -nE 'large_client_header_buffers|client_header_buffer_size'
```

Test with a 12 KB cookie whether nginx returns 400:

```bash
curl -sS -o /dev/null -w '%{http_code}\n' -H "Cookie: test=$(head -c 12000 /dev/zero | tr '\0' a)" https://example.com/
```

Search error.log for matching entries:

```bash
grep -i 'too large' /var/log/nginx/error.log | tail -n 5
```

## Fix

1. For the user: clear cookies for the domain – that confirms the cause right away.
2. In the server block set `large_client_header_buffers 4 32k;` and run `nginx -t && systemctl reload nginx`.
3. With SSO limit the number of groups in the token or fetch groups from the userinfo endpoint.
4. Scope cookies to the needed domain and path instead of `.example.com`.
5. Raise header limits on the backend too, or you get a 431 or 400 there.

## FAQ

**How large can I set the buffers?**

32k per buffer is almost always enough. Much larger values cost memory per connection and make abuse easier. Better shrink the cause – the large cookie.

**Why does it only affect some users?**

Because only their browser collected that many cookies, e.g. through many apps or an old loop. A private window confirms it quickly.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/http-400-request-header-or-cookie-too-large/](https://daniele-michelin.com/en/errors/http-400-request-header-or-cookie-too-large/)**
