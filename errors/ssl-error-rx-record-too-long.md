# SSL_ERROR_RX_RECORD_TOO_LONG in Firefox

> The browser expects a TLS handshake on port 443 but receives plain text, usually an HTTP response. Firefox reads those bytes as an oversized TLS record and gives up.

Example:

```
Secure Connection Failed – Error code: SSL_ERROR_RX_RECORD_TOO_LONG
```

## Common causes

- nginx has `listen 443;` without `ssl`.
- Apache virtual host on *:443 without `SSLEngine on`.
- Port forwarding on the router or load balancer maps 443 to HTTP port 80.
- Another service (e.g. an app without TLS) occupies port 443.
- TLS termination disabled in the proxy, backend speaks HTTP only.

## How to check

Test with HTTP on port 443 – if HTML comes back, the server speaks no TLS:

```bash
curl -s --max-time 5 http://example.com:443/ | head -c 300
```

Test the TLS handshake directly (“wrong version number” confirms it):

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | head -n 3
```

Show listen lines for 443 in the active config:

```bash
nginx -T 2>/dev/null | grep -nE '^\s*listen .*443'
```

Find the process on port 443:

```bash
ss -ltnp | grep ':443 '
```

## Fix

1. In nginx set `listen 443 ssl;` and `listen [::]:443 ssl;`, plus `ssl_certificate` and `ssl_certificate_key`.
2. In Apache enable `SSLEngine on` in the 443 virtual host and load `mod_ssl`.
3. Check port forwards: external 443 must map to an internal TLS port.
4. Then run `nginx -t && systemctl reload nginx` or `apachectl configtest && systemctl reload apache2`.

## FAQ

**What is the error called in Chrome?**

Chrome usually shows ERR_SSL_PROTOCOL_ERROR, curl says “wrong version number”. Same cause: plain text on a TLS port.

**Is http2 in the listen line enough?**

No. Without `ssl` nginx listens unencrypted. Since nginx 1.25.1 you enable HTTP/2 separately with `http2 on;`.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/ssl-error-rx-record-too-long/](https://daniele-michelin.com/en/errors/ssl-error-rx-record-too-long/)**
