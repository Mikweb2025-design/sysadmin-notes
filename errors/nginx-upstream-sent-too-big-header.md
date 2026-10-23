# nginx: upstream sent too big header while reading response header

> The backend's response headers (usually cookies) are larger than the buffer nginx reserves for them; nginx gives up and returns 502 Bad Gateway.

Example:

```
2026/10/23 08:47:55 [error] 1234#1234: *7890 upstream sent too big header while reading response header from upstream, client: 198.51.100.7, server: cloud.example.com, request: "GET /apps/user_saml/saml/acs HTTP/2.0"
```

## Common causes

- Large or many `Set-Cookie` headers, typical after SSO logins (SAML, OIDC) with long tokens.
- Long `Content-Security-Policy` or `Link` headers.
- Default 4 KB / 8 KB buffers (`fastcgi_buffer_size`, `proxy_buffer_size`) are too small.
- Application debug output ends up in headers.

## How to check

Find the error in the nginx log:

```bash
grep 'too big header' /var/log/nginx/error.log | tail -n 5
```

Measure the response header size:

```bash
curl -s -o /dev/null -w '%{size_header}\n' https://cloud.example.com/
```

Look at the full headers:

```bash
curl -sI https://cloud.example.com/login
```

Show current buffer settings:

```bash
nginx -T 2>/dev/null | grep -E 'fastcgi_buffer|proxy_buffer'
```

## Fix

1. For PHP-FPM: `fastcgi_buffer_size 32k;` and `fastcgi_buffers 16 16k;` in the matching `location` block.
2. For a reverse proxy: `proxy_buffer_size 16k;`, `proxy_buffers 8 16k;`, `proxy_busy_buffers_size 32k;`.
3. Apply with `nginx -t && systemctl reload nginx`.
4. Reduce unnecessary or duplicate cookies in the application.

## FAQ

**Why does it only happen after the SSO login?**

On the redirect back from the identity provider the application sets several session and SAML cookies at once. That one response overflows the buffer.

**Can I make the buffers as large as I want?**

The values apply per connection. 32–64 KB is harmless; much more points to a problem in the application.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nginx-upstream-sent-too-big-header/](https://daniele-michelin.com/en/errors/nginx-upstream-sent-too-big-header/)**
