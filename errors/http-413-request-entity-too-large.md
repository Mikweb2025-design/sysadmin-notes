# HTTP 413 Request Entity Too Large in nginx

> The request body (usually an upload) is larger than the limit nginx or a proxy in front of it allows – the request never reaches the application.

Example:

```
2026/10/23 11:20:13 [error] 1234#1234: *3456 client intended to send too large body: 104857600 bytes, client: 198.51.100.7, server: cloud.example.com, request: "PUT /remote.php/dav/files/anna/video.mp4 HTTP/2.0"
```

## Common causes

- `client_max_body_size` is not set; the nginx default is 1 MB.
- The directive is in the wrong context or in a file that is not included.
- A second proxy in front (load balancer, Docker proxy, Cloudflare with 100 MB on the free plan) has its own limit.
- PHP limits `upload_max_filesize` and `post_max_size` are too small (usually an application error rather than 413).

## How to check

Find the effective value in the loaded config:

```bash
nginx -T 2>/dev/null | grep -n client_max_body_size
```

Find 413 cases in the log:

```bash
grep 'too large body' /var/log/nginx/error.log | tail -n 10
```

Check the PHP-FPM limits (not the CLI ones):

```bash
grep -rE 'upload_max_filesize|post_max_size' /etc/php/8.3/fpm/
```

Test an upload directly:

```bash
curl -s -o /dev/null -w '%{http_code}\n' -X PUT --data-binary @test-200M.bin https://cloud.example.com/upload-test
```

## Fix

1. Set `client_max_body_size 10G;` in the right `server` or `location` block (`0` disables the check).
2. Test and reload: `nginx -t && systemctl reload nginx`.
3. Raise the same limit on every proxy in front.
4. Adjust `upload_max_filesize` and `post_max_size` in PHP-FPM, then `systemctl reload php8.3-fpm`.
5. With Nextcloud, rely on chunked uploads so single requests stay small.

## FAQ

**Why doesn't changing php.ini help?**

Because the 413 comes from nginx before PHP ever sees the request. Also, PHP-FPM uses a different php.ini than the CLI.

**Is client_max_body_size 0 a good idea?**

Only behind authentication. On public endpoints it allows arbitrarily large requests and makes simple resource attacks easy.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/http-413-request-entity-too-large/](https://daniele-michelin.com/en/errors/http-413-request-entity-too-large/)**
